---
author: Axton
pubDatetime: 2023-11-10T15:22:00Z
modDatetime: 2023-11-10T15:22:00Z
title: JS如何判断文字被ellipsis了？
featured: false
draft: false
tags:
  - ellipsis
description:
   如果我们想要当文本被省略的时候，**也就是当文本超出指定的宽度后，鼠标悬浮在文本上面才展示popper**，应该怎么实现呢？
---

## Table of contents

如果想要文本超出宽度后用省略号省略，只需要加上以下的css就行了。

```css
.ellipsis {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
```

3行css搞定，但是问题来了：如果我们想要当文本被省略的时候，**也就是当文本超出指定的宽度后，鼠标悬浮在文本上面才展示popper**，应该怎么实现呢？

CSS帮我们搞定了省略，但是JS并不知道文本什么时候被省略了，所以我们得通过JS来计算。接下来，我将介绍几种方法来实现JS计算省略。

---

## createRange

我发现Element-plus表格组件已经实现了这个功能，所以就先来学习一下它的源码。

源码地址：[https://github.com/element-plus/element-plus/blob/dev/packages/components/table/src/table-body/events-helper.ts](https://github.com/element-plus/element-plus/blob/dev/packages/components/table/src/table-body/events-helper.ts "标题")

```ts
// 仅仅粘贴相关的
const cellChild = (event.target as HTMLElement).querySelector('.cell') 
const range = document.createRange()
range.setStart(cellChild, 0)
range.setEnd(cellChild, cellChild.childNodes.length)
let rangeWidth = range.getBoundingClientRect().width
let rangeHeight = range.getBoundingClientRect().height
/** detail: https://github.com/element-plus/element-plus/issues/10790
* What went wrong?
* UI > Browser > Zoom, In Blink/WebKit, getBoundingClientRect() sometimes returns inexact values, probably due to lost
precision during internal calculations. In the example above:
* - Expected: 188
* - Actual: 188.00000762939453
*/
const offsetWidth = rangeWidth - Math.floor(rangeWidth)
if (offsetWidth < 0.001) {
  rangeWidth = Math.floor(rangeWidth)
}
const offsetHeight = rangeHeight - Math.floor(rangeHeight)
if (offsetHeight < 0.001) {
  rangeHeight = Math.floor(rangeHeight)
}


const { top, left, right, bottom } = getPadding(cellChild) // 见下方
const horizontalPadding = left + right
const verticalPadding = top + bottom
if (
  rangeWidth + horizontalPadding > cellChild.offsetWidth ||
  rangeHeight + verticalPadding > cellChild.offsetHeight ||
  cellChild.scrollWidth > cellChild.offsetWidth
) {
  createTablePopper(
    parent?.refs.tableWrapper,
    cell,
    cell.innerText || cell.textContent,
    nextZIndex,
    tooltipOptions
  )
}
```
```ts
// 上面代码17行中的getPadding函数
const getPadding = (el: HTMLElement) => {
  const style = window.getComputedStyle(el, null)
  const paddingLeft = Number.parseInt(style.paddingLeft, 10) || 0
  const paddingRight = Number.parseInt(style.paddingRight, 10) || 0
  const paddingTop = Number.parseInt(style.paddingTop, 10) || 0
  const paddingBottom = Number.parseInt(style.paddingBottom, 10) || 0
  return {
    left: paddingLeft,
    right: paddingRight,
    top: paddingTop,
    bottom: paddingBottom,
  }
}
```

`document.createRange()` 是 JavaScript 中的一个方法，用于创建一个 Range 对象，表示文档中的一个范围。Range 对象通常用于选择文档中的一部分内容，然后对其进行操作。

它可以：

- 设置选中文本范围：可以使用 `document.createRange()` 方法创建一个 Range 对象，并使用 `setStart()` 和 `setEnd()` 方法设置选中文本的起始和结束位置。

- 插入新元素：可以使用 `document.createRange()` 方法创建一个 Range 对象，并使用 `insertNode()` 方法将新元素插入到文档中的指定位置。

- 获取特定元素的位置：可以使用 `document.createRange()` 方法创建一个 Range 对象，并使用 `getBoundingClientRect()` 方法获取元素在文档中的位置和大小信息。

这边element就是使用range对象的getBoundingClientRect获取到元素的宽高，同时因为得到的宽高值有很多位的小数，所以element-plus做了一个判断，如果小数值小于0.001就舍弃小数部分。

接下来，就让我们进行一下复刻吧，可以通过调整盒子的宽度，在页面中看到是否有省略号的判断。

```html
<div class="ellipsis box">
  Lorem ipsum dolor sit amet consectetur adipisicing elit. 
</div>

<style>
  .ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .box {
    border: 1px solid gray;
    padding: 10px;
  }

</style>
```

注意这里，我们需要区分clientWidth和offsetWidth，因为我们现在给了box加了1px的边框，所以offsetWidth = 1 * 2 (左右两边的border宽度) + clientWidth，所以我们这边使用clientWidth来代表box的实际宽度。

![图片](@/assets/images/1699607271000.png "标题")

```js
const checkEllipsis = () => {
  const range = document.createRange();
  range.setStart(box, 0)
  range.setEnd(box, box.childNodes.length)
  let rangeWidth = range.getBoundingClientRect().width
  let rangeHeight = range.getBoundingClientRect().height
  const contentWidth = rangeWidth - Math.floor(rangeWidth)
  const { pLeft, pRight } = getPadding(box)
  const horizontalPadding = pLeft + pRight
  if (rangeWidth + horizontalPadding > box.clientWidth) {
    result.textContent = '存在省略号'
  } else {
    result.textContent = '容器宽度足够，没有省略号了'
  }
}
```

这种方法div里面放的元素和样式是不受限制的，比如html这样写还是能够正确计算的。

```html
<div class="ellipsis box">
  Lorem ipsum dolor sit amet consectetur adipisicing elit. 
  <span style="font-size: large;">hello world</span>
  <span style="letter-spacing: 20px;">hello world</span>
</div>
```

## 创建一个div来获取模拟宽度

我们可以还可以通过创建一个几乎相同的div来获取没有overflow:hidden时元素的实际宽度。

```html
<div class="ellipsis box">
  Lorem ipsum dolor sit amet consectetur adipisicing elit. 
</div>

<style>
  .ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .box {
    border: 1px solid gray;
    padding: 10px;
  }

</style>
```
```js
const checkEllipsis = () => {
  const elementWidth = box.clientWidth;
  const tempElement = document.createElement('div');
  const style = window.getComputedStyle(box, null)
  tempElement.style.cssText = `
    position: absolute;
    top: -9999px;
    left: -9999px;
    white-space: nowrap;
    padding-left:${style.paddingLeft};
    padding-right:${style.paddingRight};
    font-size: ${style.fontSize};
    font-family: ${style.fontFamily};
    font-weight: ${style.fontWeight};
    letter-spacing: ${style.letterSpacing};
  `;
  tempElement.textContent = box.textContent;
  document.body.appendChild(tempElement);
  if (tempElement.clientWidth >= elementWidth) {
    result.textContent = '存在省略号'
  } else {
    result.textContent = '容器宽度足够，没有省略号了'
  }
  document.body.removeChild(tempElement);
}
```

当box元素里面存在多个dom元素的时候，还得进行一个递归创建dom，或者也可以试试cloneNode(true)来试试克隆。

## 创建一个block元素来包裹inline元素

这种方法从acro design vue中学到的，应该是最简单的办法。要点就是外层一定是block元素，内层是inline元素

```html
<div class="ellipsis box">
  <span class="content">
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Lorem ipsum dolor sit amet consectetur adipisicing
    elit.
  </span>
</div>

<style>
  .ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .box {
    border: 1px solid gray;
    padding: 10px;
  }

</style>
```

通过上面对css和html做的处理，我们可以实现让box元素里面的文字进行ellipisis，同时由于并没有 对span.content进行任何overflow的处理，所以该 span 的offsetWidth还是保持不变。

```js
const checkEllipsis = () => {
  const { pLeft, pRight } = getPadding(box)
  const horizontalPadding = pLeft + pRight
  if (box.clientWidth <= content.offsetWidth+horizontalPadding ) {
    result.textContent = '存在省略号'
  } else {
    result.textContent = '容器宽度足够，没有省略号了'
  }
}
```

同样，只要满足外层元素是block，内层元素是inline的话，里面的dom元素其实是随便放的

```html
<div class="ellipsis box">
  <span class="content">
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Lorem ipsum dolor sit amet consectetur adipisicing
    elit.
    <span style="font-size: large;">
      hello world
    </span>
    <span style="letter-spacing: 20px;">
      hello world
    </span>
  </span>
</div>
```
##  方法比较

1. 性能（个人主观判断）3>1>2<br/>
2. 省心程度（个人主观判断）：1>3>2<br/>
3. 精确度（个人主观判断）：3种方法精确度几乎相同，如果硬要比较我觉得是3>1>2<br/>
之后我在看看其他组件库有什么好的方法，然后再补充上来，前端总是在做这些很小很小的点，哈哈。

>作者：嘉琪coder<br>
>链接：https://juejin.cn/post/7262280335978741797<br>
>来源：稀土掘金<br>
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。