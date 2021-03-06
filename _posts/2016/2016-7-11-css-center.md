---
layout: post
title: CSS实现水平和垂直居中
categories:
- web
tags:
- css
---

     
	 
##CSS实现水平和垂直居中

> 在布局中，居中是很实用的技巧，很惭愧，会的不多

> 刚好有机会多接触思考一些居中方法，记录一下

###css水平居中：

``````````````````````````````````
\<div class=“parent”>
     \<div class=“child”>DEMO</div>
\</div>
``````````````````````````````````

1、

``````````````````````````````````
.child{
     display:inline-block/*内容多宽就多宽*/
}
.parent{
     text-align:center;/*只对inline有效*/
}
/*兼容性好，但是center会被继承*/
``````````````````````````````````

2、

``````````````````````````````````
.child{
     display:table;/*类似block，但是宽度跟着内容走*/
     margin:0 auto;
}
/*IE8兼容性好，不受parent影响*/
``````````````````````````````````

3、

``````````````````````````````````
.parent{
     position:relative;/*给子元素的absolute提供定位参考*/
}
.child{
     position:absolute;/*参照父元素位置，且宽度也随着元素走*/
     left:50%;/*只有左边居中*/
     transform:translateX(-50%);/*自身宽度50%左移*/
}
/*脱离文档流不受其他元素影响，但是CSS3兼容差*/
``````````````````````````````````

4、

``````````````````````````````````
.parent{
     display:flex;/*宽度auto*/
     justify-content:center;/*居中对齐*/
}
``````````````````````````````````

或者加一个

``````````````````````````````````
.child{
     margin: 0 auto;
}
/*不用动子元素但是兼容差*/
``````````````````````````````````

###垂直居中

1、

``````````````````````````````````
.parent{
     display: table-cell;/*单元格布局，根据元素高度划分*/
     vertical-align:middle;/*中间*/
}
/*兼容IE8，只修改父元素*/
``````````````````````````````````
2、

``````````````````````````````````
.parent{
     position:relative;
}
.child{
     position:absolute;
     top:50%;
     transform:translateY(-50%);
}
/*有缺点同水平3*/
``````````````````````````````````
3、

``````````````````````````````````
.parent{
     display:flex;/*child会拉伸占满父元素高度*/
     align-items:center;
}
/*优缺点同水平4*/
``````````````````````````````````

###水平垂直居中

> 水平＋垂直

- 2+1

- 3+2

- 4+3


----

