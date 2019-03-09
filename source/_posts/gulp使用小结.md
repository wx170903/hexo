---
title: gulp使用小结j
date: 2018-12-24 11:43:14
tags: JS
categories: js工具
---
### glup安装
````
$ npm install --global gulp-cli
$ npm install --save-dev gulp
````

### 安装各种插件
````
npm install --save gulp            //本地使用
npm install --save gulp-imagemin   //压缩图片
npm install --save gulp-minify-css //压缩css
npm install --save gulp-ruby-sass  //sass
npm install --save gulp-jshint     //js代码检测
npm install --save gulp-uglify     //js压缩
npm install --save gulp-concat     //文件合并
npm install --save gulp-rename     //文件重命名
npm install --save png-sprite      //png合并
npm install --save gulp-htmlmin    //压缩html
npm install --save gulp-clean      //清空文件夹
npm install --save browser-sync    //文件修改浏览器自动刷新
npm install --save gulp-shell      //执行shell命令
npm install --save gulp-ssh        //操作远程机器
npm install --save run-sequence    //task顺序执行
````

### 创建glupfile.js
````
(示例1)
var gulp = require('gulp');
var pug = require('gulp-pug');
var less = require('gulp-less');
var minifyCSS = require('gulp-csso');
var concat = require('gulp-concat');
var sourcemaps = require('gulp-sourcemaps');

gulp.task('html', function(){
  return gulp.src('client/templates/*.pug')
    .pipe(pug())
    .pipe(gulp.dest('build/html'))
});

gulp.task('css', function(){
  return gulp.src('client/templates/*.less')
    .pipe(less())
    .pipe(minifyCSS())
    .pipe(gulp.dest('build/css'))
});

gulp.task('js', function(){
  return gulp.src('client/javascript/*.js')
    .pipe(sourcemaps.init())
    .pipe(concat('app.min.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('build/js'))
});

gulp.task('default', [ 'html', 'css', 'js' ]);
````

### glup 语法
- glup.src() 转化的文件
````
gulp.src(globs[, options])
globs : String or Array

glup.src('client/template/*.jade')
    .pipe(jade())
    .pipe(minify())
    .pipe(glup.dest('build/minifiled_templates'))

// 示例
client/
  a.js
  bob.js
  bad.js
glup.src(['client/*.js', '!client/b*.js', 'client/bad.js']) 
// 要所有.js结尾的文件, 不要以b开头的 , 要bad.js
````

- glup.dest() 数据流变成文件
````
gulp.dest(path[, options])
path: String or Function

gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))  // 路径
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));  // 放置路径
````

- glup.task() 声明一个任务
````
gulp.task(name [, deps, fn])
name: String 不应该有空格
deps: Array
fn: Function

gulp.task('somename', function() {
  // Do ...
});
gulp.task('mytask', ['array', 'of', 'task', 'names'], function() {
  // Do stuff
});
````

- glup.watch() 监控文件的变动
````
gulp.watch(glob[, opts], tasks)
glob: String or Array
tasks: Array

var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});
````

### glup使用案例
- demo1目录结构如下。把demo1中的 index.html压缩，把src里面的less编译、合并、压缩、重命名、存储到dist。src里面的图片压缩、合并存储到dist。src里面的js做代码检查，压缩，合并，存储到dist。

````
+ demo1
        + dist
            + css
                - merge.min.css
            + js
                - merge.min.js
            + imgs
                - 1.png
                - 2.png
            - index.html
        + src
            + css
                - a.css
                - b.css
            + js
                - a.js
                - b.js
            + imgs
                - 1.png
                - 2.png
            - index.html
````
````
(glupfile.js)
    var gulp = require('gulp');
    // 引入组件
    var minifycss = require('gulp-minify-css'), // CSS压缩
        uglify = require('gulp-uglify'), // js压缩
        concat = require('gulp-concat'), // 合并文件
        rename = require('gulp-rename'), // 重命名
        clean = require('gulp-clean'), //清空文件夹
        minhtml = require('gulp-htmlmin'), //html压缩
        jshint = require('gulp-jshint'), //js代码规范性检查
        imagemin = require('gulp-imagemin'); //图片压缩


    gulp.task('html', function() {
      return gulp.src('src/*.html')
        .pipe(minhtml({collapseWhitespace: true}))
        .pipe(gulp.dest('dist'));
    });

    gulp.task('css', function(argument) {
        gulp.src('src/css/*.css')
            .pipe(concat('merge.css'))
            .pipe(rename({
                suffix: '.min'
            }))
            .pipe(minifycss())
            .pipe(gulp.dest('dist/css/'));
    });
    gulp.task('js', function(argument) {
        gulp.src('src/js/*.js')
            .pipe(jshint())
            .pipe(jshint.reporter('default'))
            .pipe(concat('merge.js'))
            .pipe(rename({
                suffix: '.min'
            }))
            .pipe(uglify())
            .pipe(gulp.dest('dist/js/'));
    });

    gulp.task('img', function(argument){
        gulp.src('src/imgs/*')
            .pipe(imagemin())
            .pipe(gulp.dest('dist/imgs'));
    });

    gulp.task('clear', function(){
        gulp.src('dist/*',{read: false})
            .pipe(clean());
    });

    gulp.task('build', ['html', 'css', 'js', 'img']);

$ glup build
````