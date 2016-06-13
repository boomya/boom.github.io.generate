title: react入门实例
date: 2015-08-28 17:05:11
categories: react  
tags: ['react','javascript','前端']
description:
---
简单的react实例, 实现一个投票功能  
<!--more-->
###搭建开发/编译环境

```sh
mkdir react_qa && cd react_qa
npm init
npm install --save react
```
>save和save-dev可以省掉你手动修改package.json文件的步骤。自动把模块和版本号分别添加到dependencies部分和devDependencies部分.

```sh
npm install -g gulp
npm install --save-dev gulp gulp-browserify gulp-concat gulp-react gulp-connect lodash reactify  
```
- *gulp* 前端自动化构建工具
> 语法检查（gulp-jshint）  
> 合并文件（gulp-concat）  
> 压缩代码（gulp-uglify）

**例子**
```javascript
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');
// 语法检查
gulp.task('jshint', function () {
    return gulp.src('src/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
});
// 合并文件之后压缩代码
gulp.task('minify', function (){
     return gulp.src('src/*.js')
        .pipe(concat('all.js'))
        .pipe(gulp.dest('dist'))
        .pipe(uglify())
        .pipe(rename('all.min.js'))
        .pipe(gulp.dest('dist'));
});
// 监视文件的变化
gulp.task('watch', function () {
    gulp.watch('src/*.js', ['jshint', 'minify']);
});
// 注册缺省任务
gulp.task('default', ['jshint', 'minify', 'watch']);
```
- *gulp-browserify* 可以为浏览器编译node风格的遵循CommonJS的模块
- *gulp-concat* 连接合并文件
- *gulp-react* 预编译*JSX*文件
- *gulp-connect* 可以在本地运行web服务器, 结合*livereload*实现热部署
- *lodash* 可以把他看成是对javascript底层的扩充,像是*function、array、object*這些原生的物件，*lodash*都增加了许多方法.
- *reactify* 转换*JSX*.  
<br/>
```sh
bower init
bower install bootstrap --save
```
###运行sample  
```sh
gulp serve
```  
###源码
[react-sample](https://github.com/boomya/react)
