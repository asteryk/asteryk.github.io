---
layout: post
title: CSS实现垂直布局
categories:
- web
tags:
- css
---

##CSS实现垂直布局

> 今天和小伙伴讨论到当年面试被张云龙大神问的一个问题：有一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度。

> 我记得我当时是用的position，可是今天被盼哥一问我给问蒙了，想不起来我咋做的，就口胡说JS实现的。后来小伙伴一提醒我才想起来，实在惭愧，对不起校招进来的层层筛选啊，但是装逼失败归装逼失败，这个东西我想了下很有意思，就拿出来实现了一下，也算对自己有个交代

###一个外部容器，上面边定高，下边自适应

<img alt="效果图" src="http://i1.piimg.com/4851/db083f1b7d848170.jpg" width="200px;" />

1、position布局，大家都很熟悉的西方那一套，兼容性很好，很一颗赛艇，可是今天被盼哥一问我居然忘了，实在是半瓶子醋实力不行啊

HTML:

``````````````````````````````````
 \  <div class="container">
        <div class="msc1">
        </div>
        <div class="msc2">
        </div>
    </div>

``````````````````````````````````

CSS：

``````````````````````````````````
    .container {
        position: relative;
        background: #cccccc;
        width: 400px;
        height: 400px;
    }
    
    .msc1 {
        background: #999999;
        height: 100px;
        width: 100%;
    }
    
    .msc2 {
        background: #000;
        position: absolute;
        top: 100px;
        left: 0px;
        bottom: 0px;
        width: 100%;
    }
    // 因为最外面设一个高度好查看，所以才在最外面设了400px，原题是要外面自适应的，不影响

``````````````````````````````````

2、flex方案，这也是最近在前端微专业学来的，用浩哥的话就是：这种东西你webview玩下就好了，平时谁会用啊。言下之意就是兼容性差，恩。

HTML:同上

CSS：

``````````````````````````````````
    .container {
        background: #cccccc;
        width: 400px;
        height: 400px;
        display: flex;
        flex-direction: column;
    }
    
    .msc1 {
        background: #999999;
        height: 100px;
        width: 100%;
    }
    
    .msc2 {
        background: #000;
        width: 100%;
        flex: 1
    }

``````````````````````````````````

3、CSS3实现，这里用了calc()的方法可以拿百分比减px，不过这里有个坑，那就是：
<br />
计算公式里‘-’两头要带空格，至于为什么呢，我也不知道，因为W3标准就是这么写的

>Note that the grammar requires spaces around binary ‘+’ and ‘-’ operators. The ‘*’ and ‘/’ operators do not require spaces.

`http://www.w3.org/TR/css3-values/#calc § 8.1.1`

<br />
HTML同1，

CSS：

``````````````````````````````````
    .container {
        background: #cccccc;
        width: 400px;
        height: 400px;
    }
    
    .msc1 {
        background: #999999;
        height: 100px;
        width: 100%;
    }

    .msc2{
        //需要兼容不同浏览器
        height:-moz-calc(100% - 100px);
        height:-webkit-calc(100% - 100px);
        height: calc(100% - 100px);
        width: 100%;
        background: #000;
    }

``````````````````````````````````

3、JS实现，本来我想不起CSS的解法，情急之下口胡说JS获取高度一减就OK，其实这里也有个我不常遇到的情况（平时原生写得少），就是写在css里的元素属性，用object.style.height是读不到的，具体要怎么解决呢，看下面

HTML同1

CSS：

``````````````````````````````````
    .container {
        /*position: relative;*/
        background: #cccccc;
        width: 400px;
        height: 400px;
        display: flex;
        flex-direction: column;
    }
    
    .msc1 {
        background: #999999;
        height: 100px;
        width: 100%;
    }
    
    .msc2 {
        width: 100%;
        background: #000;
    }

``````````````````````````````````

JS：

``````````````````````````````````
    function getHeight(className) {
        var dom = document.getElementsByClassName(className)[0];
        // 先获得dom
        var domHeight = window.getComputedStyle(dom).getPropertyValue("height");
        // 再去window下找这个元素的属性，然后读到值，然后这个值是个字符串'400px'
        return Number(domHeight.split('px')[0]);
    }
    var containerHeight = getHeight('container');
    document.getElementsByClassName('msc2')[0].style.height = String(containerHeight - 100) + 'px';
    // 提取出400做加减法，再用style，OK

``````````````````````````````````

>7-23日补充：玉林师兄说原生JS用clientHeight可以得到高度，这个我给我忘了，这样简单不少，sorry


``````````````````````````````````
    function getHeight(className) {
        var dom = document.getElementsByClassName(className)[0];
        var domHeight = dom.clientHeight;
        return domHeight;
    }
    var containerHeight = getHeight('container');
    document.getElementsByClassName('msc2')[0].style.height = String(containerHeight - 100) + 'px';

``````````````````````````````````


###总结:没事实力不行还是要多实践多看书，少装硬逼多吹水（上面有写错的，实现不佳的，新的补充的欢迎告诉我：-））