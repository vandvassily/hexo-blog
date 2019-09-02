---
title: gulp配置
p: automation/gulp-config
date: 2019-08-29 20:32:33
categories: 
    - ['自动化工程', '构建工具']
tags: 
    - [自动化工程]
    - [构建]
    - [gulp]
---

# gulp配置

gulp前端工程构建工具，功能强大，配置简单灵活

我暂时只用到以下几方面

>   1. 添加公共头部和尾部
>   2. css和js添加时间戳和版本号
>   3. css兼容前缀、压缩
>   4. js压缩混淆

项目地址: [gulp-config](https://github.com/vandvassily/gulp-config)

## 使用

```node
# 安装所需的npm包
npm install

# 开发环境
npm run dev

# 生产环境，文件打包至dist文件夹
npm run build

# 如需压缩js
npm run build:uglify

# 清空dist文件夹
npm run clean
```

## 项目结构

```tree
gulp-config                                    // 项目名称
├─ .browserslistrc                             // css兼容配置
├─ .gitignore                                  //
├─ node_modules                                // npm包
├─ dist                                        // 生产环境文件
├─ gulpfile.js                                 // gulp配置文件
├─ LICENSE                                     //
├─ package-lock.json                           //
├─ package.json                                // npm包版本管理
├─ README.md                                   // 说明文档
└─ src                                         //
   ├─ assets                                   // 静态文件
   │  ├─ css                                   // css
   │  │  └─ common.css                         // 公共css
   │  ├─ img                                   // 图片
   │  ├─ js                                    // 公共js
   │  └─ lib                                   // 第三方js库
   ├─ include                                  // 公共头部，底部，和内容
   │  ├─ com_body.html                         //
   │  ├─ com_bottom.html                       //
   │  └─ com_head.html                         //
   └─ views                                    // 页面文件夹
      └─ hello-world                           // 某个页面
         ├─ css                                //
         │  └─ index.css                       //
         ├─ index.html                         //
         └─ js                                 //
            └─ index.js                        //
```

## 添加公共头部和尾部

使用插件：`gulp-file-include`
<fancybox>![avatar](gulp_include1.png)</fancybox>

模板文件放在`src/include`中，在需要套入模板的地方使用`@@include('@@webRoot/include/XXX.html')`即可

## css处理

css只做了浏览器兼容处理，并没有使用压缩功能，如要使用，可以使用`gulp-csso`进行处理

```javascript
/**
 * css添加浏览器兼容
 */
function css() {
    let plugins = [
        autoprefixer()
    ];
    return src('src/**/*.css')
        .pipe(postcss(plugins))
        .pipe(dest('dist/'));
}
```

## js处理

开发环境和生产环境使用的不同方法，根据具体情况决定是否使用js混淆压缩

```javascript
/**
 * 未做混淆压缩，开发使用
 */
function js() {
    return src('src/**/*.js')
        .pipe(dest('dist/'));
}

/**
 * 混淆压缩JS代码，如不需要请不要使用
 * 如果只需要压缩，不需要sourcemaps，去除掉sourcemaps选项即可
 */
function uglifyJs() {
    return src('src/**/*.js', { sourcemaps: true })
        .pipe(uglify())
        .pipe(dest('dist/', { sourcemaps: '.' }));
}
```

## 图片处理

现在未对图片做压缩处理

```javascript
function images() {
    return src('src/**/*.{png,jpg,gif}')
        .pipe(dest('dist/'));
}
```

如需要使用图片压缩，自己使用`gulp-imagemin`实现即可

```bash
# 文档地址: https://github.com/sindresorhus/gulp-imagemin

# 安装插件
npm install --save-dev gulp-imagemin
```

```javascript
# gulpfile.js
...
const imagemin = require('gulp-imagemin');
···

function images() {
    return src('src/**/*.{png,jpg,gif}')
        .pipe(imagemin())
        .pipe(dest('dist/'))
}
```

## html处理

```javascript
function getTimestamp() {
    var timestamp = new Date();
    return isDevelopment()
        ? timestamp.getTime()
        : dayjs().format('YYYY-MM-DD-HH-mm-ss')
}

/**
 * html页面添加公共头部和尾部，以及添加时间戳或版本戳
 * 开发环境为时间戳，生产环境为日期戳
 */
function html() {
    return src(['src/**/*.html'])
        .pipe(fileinclude({
            prefix: '@@',
            basepath: '@file'
        }))
        .pipe(replace('@version', 'v=' + getTimestamp()))
        .pipe(dest('dist/'));
}
```