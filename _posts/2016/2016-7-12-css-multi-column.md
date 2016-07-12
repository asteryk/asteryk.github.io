---
layout: post
title: CSS实现多列布局
categories:
- web
tags:
- css
---

##CSS实现多列布局

> 之前布局一直都是瞎JB乱写，正好抽时间总结一下在这里记录下来

###一边定宽，一边自适应

<img alt="效果图" src="http://i1.piimg.com/567571/c06b6df26d442e62.jpg" width="200px;" />

1、float布局

HTML:

``````````````````````````````````
 \<div class="parent">
        <div class="left">
            <p>left</p>
        </div>
        <div class="right">
            <p style="clear:left;">right</p>
            <p>right</p>
        </div>
    </div>
    <!-- bug：一旦在内部元素设置清除浮动就会换行 -->

``````````````````````````````````

CSS：

``````````````````````````````````
    .left {
        float: left;
        width: 100px;
    }
    
    .right {
        margin-left: 120px;
    }
    /*子元素的浮动会受影响，IE有间隔3px的bug*/

``````````````````````````````````

改进方案

HTML:

``````````````````````````````````
\<div class="parent1">
        <div class="left1">
            <p>left</p>
        </div>
        <div class="right-fix">
            <div class="right1">
                <p>right</p>
                <p>right</p>
            </div>
        </div>
    </div>

``````````````````````````````````

CSS：

``````````````````````````````````
	.left1 {
        float: left;
        width: 100px;
    }
    
    .right-fix {
        float: right;
        width: 100%;
        margin-left: -100px;
        /*使得right部分不会换行，IE6下没有间隔3px问题*/
    }
    
    .right1 {
        margin-left: 120px;
    }

``````````````````````````````````

2、float＋overflow布局

HTML同1，

CSS：

``````````````````````````````````
	.left {
        float: left;
        width: 100px;
    }
    
    .right {
        overflow: hidden;
    }
    /* overflow不为visiable会触发BFC模式，使得right内部不受浮动影响垂直上下排列。不支持IE6 */

``````````````````````````````````

3、table布局

HTML同1

CSS：

``````````````````````````````````
    .parent {
        display: table;
        width: 100%;
        table-layout: fixed;
        /* 上边这句话是因为table是根据内容设置宽度，这句话使得优先布局而不是内容，而且会加速table的渲染速度 */
    }
    
    .left,
    .right {
        display: table-cell;
        /* 小tips：table-cell不能设置margin但是可以设置padding */
    }
    
    .left {
        width: 100px;
        /* table的特点就是所有table－cell加在一起就是table总宽，所以设置了left的宽度余下的right会自己占满 */
    }
``````````````````````````````````

4、flex布局

HTML同1

CSS：

``````````````````````````````````
    .parent {
        display: flex;
    }
    
    .left {
        width: 100px;
        margin-right: 20px;
    }
    .right {
    	flex:1 1 0;
    }
    /* flex布局是根据内容来的，影响性能，不适合整个页面布局，适合局部小范围布局 */
``````````````````````````````````

###左边不定宽，右边自适应

1、float＋overflow布局

HTML同上1

CSS：

``````````````````````````````````
    .left {
        float: left;
        margin-right: 20px;
    }
    
    .right {
        overflow: hidden;
    }
    
    .left p {
        width: 200px;
        /* 靠里面内容撑开 */
    }
``````````````````````````````````

2、table布局

CSS：

``````````````````````````````````
    .parent {
        display: table;
        width: 100%;
        /* table-layout: fixed; */
        /* 上边这句话是注释掉使得table仍然是根据内容设置宽度 */
    }
    
    .left,
    .right {
        display: table-cell;
    }
    
    .left {
        width: 0.1%;
        /* 宽度设置成极小，不能不设或者1px，否则在IE8有问题 */
        /* table的特点就是所有table－cell加在一起就是table总宽，所以设置了left的宽度余下的right会自己占满 */
    }
    
    .left p {
        width: 200px;
        /* 靠里面内容撑开 */
    }
``````````````````````````````````

3、flex布局

CSS：

``````````````````````````````````

    .parent2 {
        display: flex;
    }
    
    .left2 {
        /* width: 100px; */
        background: #eee;
        margin-right: 20px;
    }
    
    .right2 {
        background: #000;
        flex: 1 1 0;
    }
    
    .left2 p {
        width: 200px;
        /* 靠里面内容撑开 */
    }
    /* flex布局只有兼容性问题*/
``````````````````````````````````

> BTW table和flex都是默认拉伸左右等高，table存在因为使用padding背景色会填充间距的问题，可以设置backgroud只填充content来解决

> 而 float做不到左右等高，必须左右设置9999px的padding－bottom和－9999px的margin－bottom来撑开，然后用overflow：hidden截取掉

###多列等宽

<img alt="效果图" src="http://i1.piimg.com/567571/46007da139544d89.jpg" width="200px;" />

1、float

HTML：

``````````````````````````````````

\<div class="parent">
        <div class="column">
            <p>1</p>
        </div>
        <div class="column">
            <p>2</p>
        </div>
        <div class="column">
            <p>3</p>
        </div>
        <div class="column">
            <p>4</p>
        </div>
    </div>
    
``````````````````````````````````

CSS:

``````````````````````````````````
	.parent {
        margin-left: -20px;
        /* 因为设计初衷是四块包括间距在内等分，但是只要三个间距，所以要把父容器延展，留出最左边间距的空间 */
    }
    
    .column {
        float: left;
        width: 25%;
        padding-left: 20px;
        box-sizing: border-box;
        /* border-box将间距包裹在里面 */
    }

``````````````````````````````````

2、table

HTML：

``````````````````````````````````

\<div class="parent-fix">
        <div class="parent1">
            <div class="column1">
                <p>1</p>
            </div>
            <div class="column1">
                <p>2</p>
            </div>
            <div class="column1">
                <p>3</p>
            </div>
            <div class="column1">
                <p>4</p>
            </div>
        </div>
    </div>
    
``````````````````````````````````

CSS:

``````````````````````````````````
	.parent-fix{
    	margin-left:-20px;
    	/* table没法margin */
    }
    .parent1{
    	display:table;
    	width:100%;
    	table-layout:fixed;
    	/* fixed特性:除了之前说的之外，多列不设宽度自动等分 */
    }
    .column1{
    	display:table-cell;
    	padding-left:20px;
    }

``````````````````````````````````

3、flex布局

HTML同1

CSS:

``````````````````````````````````
    .parent{
    	display:flex;
    }
    .column{
    	flex:1;
    	/* 不需要考虑留出多余空间，因为flex：1就是在先扣掉margin之后等分 */
    }
    .column+.column{
    	margin-left:20px;
    }

``````````````````````````````````