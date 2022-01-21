---
title: PDF展示与合成Vue组件(lz-pdf-merage)
date: 2022/01/21
categories: 
    - vue组件
tags:
    - vue
---
## 前言
主要是介绍使用pdfjs和jsPdf在vue组件中对pdf文件进行展示和对原有pdf进行合成(包括文字，图片合成)。
<br>
## 体验
[demo地址](https://lizehongss.github.io/lz-pdf-merage/dist/index.html)

npm包引入
```
npm install lz-pdf-merage
```
## pdf展示
主要使用pdfjs(**使用的版本是2.1.266**)对pdf文件通过对canvas对其进行预览展示，主要原理如下:
```js
import Pdfjs from 'pdfjs-dist'
// 获取pdf对象
this.pdf = await Pdfjs.getDocument({
  url: pdfPath
})
// 获取一页的pdf数据
this.pdf.getPage(1)
// 获取htmlK中的canvas元素
let canvas = document.getElementById('canvas')
let context = canvas.getContext('2d')
let scale = 3.2
// 放大pdf
let viewport = page.getViewport(scale, this.rotate)
// 设置canvas的宽度和高度，以解决显示模糊的问题
canvas.height = viewport.height;
canvas.width = viewport.width;
canvas.style.width = `${this.width * widthScale}px`
canvas.style.height = `${this.width/viewport.width * viewport.height * widthScale}px`;
//将pdf数据渲染到指定的canvas上
let renderContext = {
  canvasContext: context,
  viewport: viewport
};
await page.render(renderContext);
```
需要注意的点：
- 为了解决pdf在canvas上渲染后模糊的问题，需要将pdf通过getViewport(scale，0)方法进行放大。同时设置canvas的style的width和height使其在页面上正常展示。
- 对于对pdf进行放大缩小的功能，pdf本身已经放大了3.2倍了，所以只需要控制canvas的style样式展示（即通过widthScale控制），就可以在页面上呈现放大缩小的效果。
- 旋转功能的实现，一样是通过getViewport(scale, this.rotate)控制。

## pdf合成
pdf合成主要是创建一个新的canvas元素，其大小与pdf预览的canvas一致，然后将需要添加的图片和文字记录坐标并添加到新的canvas元素上。最后通过drawImage分别获取到两个canvas对应的图片，通过jsPdf将两张图片合成新的pdf页，从而实现pdf合成。
### canvas元素合成
主要代码及解释如下：
```html
<canvas class="draw-canvas" ref="drawCanvas"></canvas>
<Page ref="page" :defaultRotate="pageRotate" :page="currentPage" :width="pdfWidth - 25" @saveRotate="saveRotate" @drawFinish="handleFinishDraw"></Page>
```
```js
// 在pdfjs渲染完成canvas后触发
handleFinishDraw(canvas){
  // 获取当前pdf所在页
  let current = this.current - 1
  // 将渲染的pdf存放到baseCanvasImages中，方便后续合成
  this.baseCanvasImages[current] = {
    url: canvas.toDataURL('image/jpeg'),
    width: canvas.width,
    height: canvas.height
  }
  if (!this.canvasArray[current]) {
    let { width, height } = canvas.getBoundingClientRect()
    // 设置canvas的宽高
    let drawCanvas = this.setDrawCanvas(canvas)
    // 生成一个DrawObj实例，并存入canvasArray中，方便后续切换页时添加,
    // DrawObj是对canvas画图进行操作的类，会在后续中介绍
    // 当添加图片，文字时，都是对该实例进行操作
    this.canvasArray[current] = new DrawObj(drawCanvas, current, canvas.width/width, canvas.height/height)
    }
    // 对drawCanvas进行重绘，因为有可能进行了切换页
    this.canvasArray[current].drawItemList()
  }
  // 设置drawCanvas的宽高
  setDrawCanvas(canvas) {
    let drawCanvas = this.$refs.drawCanvas
    drawCanvas.width = canvas.width
    drawCanvas.height = canvas.height
    drawCanvas.style.width = canvas.style.width
    drawCanvas.style.height = canvas.style.height
    return drawCanvas
  },
```
### pdf合成叠加
主要介绍对pdfjs渲染而成的canvas和drawCanvas通过jsPdf进行合成
```js
async getTheMeragePdf() {
  const pdf = new jsPDF('p', 'pt', 'a4')
  // pdfjs渲染而成的canvas生成base64
  let pdfImages = await this.getThePdfImages()
  for(let i = 0; i< pdfImages.length; i++) {
    let imageObj = pdfImages[i]
    pdf.addImage(imageObj.url, 'JPEG', 0,0, 592.28, 592.28/imageObj.width * imageObj.height)
    //如果当前页对应的drawCanvas存在，叠加图片
    if(this.canvasArray[i]) {
      let drawCanvas = this.canvasArray[i].getDrawCanvas()
      const drawData = drawCanvas.toDataURL('image/png')
      pdf.addImage(drawData, 'PNG', 0,0, 592.28, 592.28/drawCanvas.width * drawCanvas.height)
    }
    // 大于一页时，生成新的一页
    if (i < pdfImages.length - 1) pdf.addPage() 
    }
    // 下载合成后的pdf
    pdf.save('合成的pdf')
  }\
  // 对pdf渲染而成的canvas生成base64
  async getThePdfImages() {
    let pdfImages = Array(this.total).fill(Infinity)
    for(let i = 0; i<pdfImages.length; i++) {
      // use the canvas after the user operation
      // 如果已存在，直接返回
      if(this.baseCanvasImages[i]){
        pdfImages[i] = this.baseCanvasImages[i]
      } else {
        // 不存在时，调用page接口渲染生成
        pdfImages[i] = await this.$refs.page.drawPage(this.pdf.getPage(i+1))
      }
    }
    return pdfImages
    },
```
### DrawObj
[DrawObj](https://github.com/lizehongss/lz-pdf-merage/blob/master/src/components/pdf-view/canvasDraw.js)是一个自定义的类，它主要实现在canvas中进行添加图片，添加文字，对添加的图片，文字进行移动和删除功能。

**对元素的操作都会反映到_drawItemList数组中,每一次操作都会清空画布，然后再重新对_drawItemList里的元素进行重绘，元素操作对_drawItemList数组的修改如下:**

- 添加元素时，会向_drawItemList数组中添加一个元素对象(主要包括元素内容，元素类型，元素位置等信息)。
- 删除元素时， 会删除_drawItemList数组中该元素。
- 移动元素时， 会修改_drawItemList数组中对应元素的X,Y。

需要注意的是由于在pdf展示时是对原有canvas进行缩小展示的，所以在调用canvas绘制元素前要调用**ctx.scale(this._scaleX, this._scaleY)**，this._scaleX和this._scaleY在初始化DrawObj时传入：
``` js
  // canvas.width/width, canvas.height/height, 是pdf展示的canvas缩放比例
  this.canvasArray[current] = new DrawObj(drawCanvas, current, canvas.width/  width, canvas.height/height)
    }
```
## pdf打印
pdf打印的原理主要还是pdfjs对canvas进行渲染生成，然后将每一页对应的canvas通过drawImage方法生成图片，再将图片添加到body上，并通过 **@media print**媒体查询控制打印样式最后调用window.print()打印。具体代码可以看这里
[printPdf](https://github.com/lizehongss/lz-pdf-merage/blob/master/src/components/pdf-view/printPdf.js)。