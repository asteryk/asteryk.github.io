---
layout: post
title: 像Google那样通过缩短选择器类名和使用作用域隔离来压缩CSS文件束(bundle)70%的体积
categories:
- web
tags:
- css
---

> 原文地址：[Reducing CSS bundle size 70% by cutting the class names and using scope isolation—Just like Google does it](https://medium.freecodecamp.org/reducing-css-bundle-size-70-by-cutting-the-class-names-and-using-scope-isolation-625440de600b)

今年年初我离开了资讯公司并开始搭建GO2CINEMA站——一个英国的猫眼电影。我做了一系列优秀的工作来让它更快更简单更安全。而我的习惯是，追求极致的渲染优化。
我用[ūsus](https://github.com/gajus/usus) 解决了HTML的预渲染，ūsus用单页面HTML和内联CSS来渲染页面，而我不喜欢内联70kb 的CSS在每个HTML文档里，尤其是里面大部分是CSS的选择器类名。
就像谷歌做的那样

你是否曾经瞄过一眼谷歌的页面代码，你第一件注意到的事就是CSS的选择器名不会超过一对单词的长度。

![google-code](https://pic4.zhimg.com/v2-1fa20e768c5fcde5751bb048fa944d3f_b.png)

但是如何处理呢？

### CSS压缩工具的缺点

有一点是压缩工具没法做到的 改变选择器的名字，这使得CSS的压缩器没法控制HTML的输出。与此同时，CSS的名字会很长，如果你使用CSS模块化开发，你的CSS模块很有可能包括了 样式表文件的名字、本地用于识别的独立名称和一串随机哈希。css加载器的配置会修饰类名模板的名称，举个例子[name]___[local]___[hash:base64:5]。
因此，一个典型的类名会看起来像这样：
` .MovieView___movie-title___yvKVV `;如果你很喜欢详细描述类名，它有可能会变得更长，举个例子 ` .MovieView___movie-description-with-summary-paragraph___yvKVV `.
在打包的时候对CSS的类名重命名


而好消息是，如果你用webpack和babel-plugin-react-css-modules，你可以在打包的时候用[css-loader getLocalIdent ](https://github.com/webpack-contrib/css-loader)或者[babel-plugin-react-css-modules generateScopedName](https://github.com/gajus/babel-plugin-react-css-modules#configuration)重命名类名。


````````````````````````````
//webpack.configuration.js
/**
 * @file Webpack configuration.
 */
const path = require('path');

const generateScopedName = (localName, resourcePath) => {
  const componentName = resourcePath.split('/').slice(-2, -1);

  return componentName + '_' + localName;
};

module.exports = {
  module: {
    rules: [
      {
        include: path.resolve(__dirname, '../app'),
        loader: 'babel-loader',
        options: {
          babelrc: false,
          extends: path.resolve(__dirname, '../app/webpack.production.babelrc'),
          plugins: [
            [
              'react-css-modules',
              {
                context: common.context,
                filetypes: {
                  '.scss': {
                    syntax: 'postcss-scss'
                  }
                },
                generateScopedName,
                webpackHotModuleReloading: false
              }
            ]
          ]
        },
        test: /\.js$/
      },
      {
        test: /\.scss$/,
        use: [
          {
            loader: 'css-loader',
            options: {
              camelCase: true,
              getLocalIdent: (context, localIdentName, localName) => {
                return generateScopedName(localName, context.resourcePath);
              },
              importLoaders: 1,
              minimize: true,
              modules: true
            }
          },
          'resolve-url-loader'
        ]
      }
    ]
  },
  output: {
    filename: '[name].[chunkhash].js',
    path: path.join(__dirname, './.dist'),
    publicPath: '/static/'
  },
  stats: 'minimal'
};

``````````````````````````````````

### 缩短命名

通过使用babel-plugin-react-css-modules 和 css-loader的相同逻辑来命名，我们可以按我们自己的想法随意改别类名，甚至是随机哈希。而更进一步，我想要更短的选择器类名来直接代替随机哈希。为了改变类名，我创建了一个类名入口然后用[incstr](https://github.com/grabantot/incstr)来改写每个通过这个入口的不断增加的ID。

这样就能改造出既短又唯一的类名。现在` .a_a `, ` .b_a `诸如此类的类名会代替
` .MovieView___movie-title___yvKVV `和` .MovieView___movie-description-with-summary-paragraph___yvKVV`

<b>这使得GO2CINEMA的css文件束从140kb压缩到53kb</b>


``````````````````````````````````

//createUniqueIdGenerator.js

const incstr = require('incstr');

const createUniqueIdGenerator = () => {
  const index = {};

  const generateNextId = incstr.idGenerator({
    // Removed "d" letter to avoid accidental "ad" construct.
    // @see https://medium.com/@mbrevda/just-make-sure-ad-isnt-being-used-as-a-class-name-prefix-or-you-might-suffer-the-wrath-of-the-558d65502793
    alphabet: 'abcefghijklmnopqrstuvwxyz0123456789'
  });

  return (name) => {
    if (index[name]) {
      return index[name];
    }

    let nextId;

    do {
      // Class name cannot start with a number.
      nextId = generateNextId();
    } while (/^[0-9]/.test(nextId));

    index[name] = generateNextId();

    return index[name];
  };
};

const uniqueIdGenerator = createUniqueIdGenerator();

const generateScopedName = (localName, resourcePath) => {
  const componentName = resourcePath.split('/').slice(-2, -1);

  return uniqueIdGenerator(componentName) + '_' + uniqueIdGenerator(localName);
};

``````````````````````````````````

### 使用作用域隔离进一步压缩文件束体积。

我会加下划线_在CSS类名里去分割组件名和识别名称——这种区分方法有助于压缩。
csso（CSS压缩工具）有作用域设置。作用域里定义了一个类名列表用于做一些专门的标记，即不同作用域的选择器不会选中污染同一个元素，这使得优化方案能更规范地修改规则。利用这个，使用[csso-webpack-plugin](https://github.com/zoobestik/csso-webpack-plugin)来后期处理CSS的文件束。


``````````````````````````````````

//getScopes.js 

const getScopes = (ast) => {
  const scopes = {};

  const getModuleID = (className) => {
    const tokens = className.split('_')[0];
  
    if (tokens.length !== 2) {
      return 'default';
    }

    return tokens[0];
  };

  csso.syntax.walk(ast, node => {
    if (node.type === 'ClassSelector') {
      const moduleId = getModuleID(node.name);

      if (moduleId) {
        if (!scopes[moduleId]) {
          scopes[moduleId] = [];
        }

        if (!scopes[moduleId].includes(node.name)) {
          scopes[moduleId].push(node.name);
        }
      }
    }
  });

  return Object.values(scopes);
};

``````````````````````````````````````

### 是否值得


第一个争议是这种压缩本身压缩算法就可以帮你做到。GO2CINEMA的CSS文件束压缩使用的是Brotli算法，比起原来的长类名的文件束只节省了1kb的体积。另一方面，设置这种压缩只是一次性投资并且这样减少的文档体积需要被解析。它有另一种好处，可以有效阻止那些通过CSS类名来扫描广告的反广告屏蔽插件。

此外，给大家提供一下这种压缩方法后的GO2CINEMA页面

- https://go2cinema.com/movies/wonder-woman-2017-1305237
- https://go2cinema.com/venues/odeon-oxford-magdalen-st-1001053