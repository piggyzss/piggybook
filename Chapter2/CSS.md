# 第2节：CSS

<!-- toc -->

- [1、盒模型](#1、盒模型)
- [2、BFC](#2、bfc)
- [3、清除浮动](#3、清除浮动)
- [4、定位机制](#4、定位机制)
- [5、水平居中/垂直居中](#5、水平居中垂直居中)
- [6、flex布局](#6、flex布局)
- [7、实现一个两边宽度固定中间自适应的三列布局](#7、实现一个两边宽度固定中间自适应的三列布局)

<!-- tocstop -->



## 1、盒模型

盒模型:content、padding、border、margin

标准模型
`box-sizing:content-box`
IE模型
`box-sizing:border-box`

```css
div{
    height: 100px;
    width: 100px;
    border: 10px solid red;
    padding: 10px;
    box-sizing:content-box;  实际大小为140px*140px;
    box-sizing:border-box;  实际大小为100px*100px;
}
```



## 2、BFC

**BFC(Block Formatting Context , 块级格式化上下文)**

BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

**官方定义：**

- 浮动元素、绝对定位元素、不是块级盒的包含块（比如inline-block、tabel-cell、tabel-captain）、和overflow值不为visible的块级盒子为他们的内容建立了一个新的块级排版上下文
- 在一个BFC中，盒子是从包含块顶部开始放置的，他们会在垂直方向一个接一个的排列
- 盒子垂直方向的距离由margin决定，属于同一个bfc的两个相邻box的margin会发生重叠
- bfc是一个页面上的独立的容器，外面的元素不会影响bfc里的元素，反过来，里面的也不会影响外面的

- 计算bfc高度的时候，浮动元素也会参与计算

**怎么取创建BFC**

- float属性不为none（脱离文档流）
- position为absolute或fixed
- display为inline-block、table-cell、table-caption、flex、inine-flex
- overflow不为visible



## 3、清除浮动

#### 3.1、浮动产生的问题

- 不清除浮动，浮动层后面跟随的非浮动内容就有可能被浮动层所覆盖，造成版面混乱
- 在文档流中，父元素的高度未设置时，默认是被子元素撑开的，也就是说子元素多高，父元素就多高。但当子元素被设置浮动之后，它会完全脱离文档流，此时会导致子元素无法撑起父元素的高度，从而导致父元素的高度塌陷。

#### 3.2、解决浮动的方法

- 增加一个空标签

```css
.div1{border:1px solid red}
.left{float:left; width:20%; height:200px;}
.right{float:right; width:30%; height:80px;}
/*清除浮动代码*/
.clearfloat{clear:both}

<div class="parent"> 
    <div class="left">Left</div> 
    <div class="right">Right</div>
    <div class="clearfloat"></div>
</div>
```

优点：浏览器支持好

缺点：增加了无意义的空标签

- 利用br标签

```html
<br clear="all">
```

优点：比空标签方式语义稍强

缺点：有悖结构与表现分离的原理

- 父元素overflow:auto或overflow:hidden

优点：不存在结构和语义化问题

缺点：内容超出父级元素时，auto会产生滚动条，hidden会隐藏部分内容

- 伪元素

```css
.parent{border:1px solid red}
.parent:after{
    content:""; 
    display:block; 
    clear:both;
}
```

优点：结构和语义化完全正确	



## 4、定位机制

position有四个属性值：static、relative、absolute、fixed

- static

position的默认值，按照正常的文档流进行排列。

当元素未定义position或定义position值为static时，该元素内定义的top, bottom, left, right 和 z-index无效。

- relative

相对定位，按照正常的文档流进行排列，相对于元素在文档流中的位置进行偏移，会依据top，right，bottom，left等属性在正常文档流中偏移位置。

- absolute

绝对定位，脱离正常文档流，不为元素预留空间，通过指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置。

绝对定位之后，标签就不再区分行内元素和块级元素了，可以设置宽高（无需display）。

- fixed

不为元素预留空间，而是通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置。元素的位置在屏幕滚动时不会改变。



## 5、水平居中/垂直居中

#### 5.1、水平居中

- 行内元素：设置text-align: center
- 块级元素：margin: 0 auto

#### 5.2、垂直居中

- 行内元素：设置line-height：height
- 块级元素：利用tabel属性

```css
#parent{
    display: table;
}
#child{
    display: table-cell; 
}
/**center code**/ vertical-align: middle; /**center code**/ 在表单元格中，这个属性会设置单元格框中的单元格内容的对齐方式
```

#### 5.3、块级元素水平垂直均居中

- 利用flex

```css
display: flex;
flex-direction: column;
justify-content: center;
align-items: center;
```

- 利用定位

```css
#parent{
    position: relative;
}

#child{
    width: 100px;
    height: 100px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%); // margin:-50px 0 0 -50px;
} 
```

或

```css
#parent{
    position:relative;
}

#child{
    position:absolute;
    margin:auto;
    top:0;
    bottom:0;
    left:0;
    right:0;
}
```



## 6、flex布局

#### 6.1、flex布局

flex布局即弹性盒布局，用来为盒状模型提供最大的灵活性。

设为 flex 布局以后，子元素的float、clear和vertical-align属性将失效。

#### 6.2、容器元素属性

- `flex-direction：row | row-reverse | column | column-reverse // 子元素的主轴方向`
- `flex-wrap：nowrap | wrap | wrap-reverse // 换行方式`
- `flex-flow： flex-direction || flex-wrap`
- `justify-content：flex-start | flex-end | center | space-between | space-around // 主轴的对齐方式`

- `align-items：flex-start | flex-end | center | baseline | stretch // 交叉轴上如何对齐`
- `align-content：flex-start | flex-end | center | space-between | space-around | stretch //多根轴线的对齐方式`

#### 6.3、子元素属性

- `order：num // 排列顺序`
- `flex-grow：num // 放大比例，默认为0，即如果存在剩余空间，也不放大`
- `flex-shrink：num // 缩小比例，默认为1，即如果空间不足，该项目将缩小，如果为0，其他项目都为1，则空间不足时，前者不缩小`
- `flex-basis：length // 在分配多余空间之前，项目占据的主轴空间（main size），默认值为auto，即项目的本来大小`
- `flex：flex-grow flex-shrink flex-basis // 0 1 auto`

```css
.item {
    flex: 1;
}
```

等同于 

```css
.item {    
  flex-grow: 1;    
  flex-shrink: 1;    
  flex-basis: 0%; 
}
```

- `align-self：auto | flex-start | flex-end | center | baseline | stretch // align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch`



**【Tips】**

**对于flex-basis的说明**

这个属性决定CSS如何给可伸缩项在容器中分配初始大小（为简化讨论，以下一律假定“大小”为宽度，高度与之类似）。以下是这个属性几种常用的值：

- auto（默认值）：auto的意思是首先看当前项有没有明确设置宽度，如果有则使用该宽度；如果没有，则以包含的内容决定宽度。
- content：content是不管当前项是否明确设置了宽度，一律以内容决定宽度
- 长度或百分比值：百分比是相对于容器而言的。知道了每一项的宽度，简单相加就能得到所有项宽度的总和。而知道了所有项宽度的总和，再与容器宽度比较，就能知道容器里是不是还有剩余空间可供再次分配。

Flex Items的应用准则：content –> width –> flex-basis

也就是说：

- 如果没有设置flex-basis属性，那么flex-basis的大小就是项目的width属性的大小
- 如果没有设置width属性，那么flex-basis的大小就是项目内容(content)的大小
- **如果同时设置了flex-basis属性和width属性，那么以flex-basis为准



## 7、实现一个两边宽度固定中间自适应的三列布局

