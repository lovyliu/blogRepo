# Gulp! 一小时从入门到 <del>放...</del> 精通 #

### 安装 ###
```
npm install -g gulp //全局安装

npm install --save-dev gulp //本地安装
```
### gulpfile.js ###
```javascript
var gulp = require('gulp');

gulp.task('mini',function(){
  // ...
});
```
命令行运行 `gulp mini`
如果 task 名为`default`，可直接执行gulp

### pipe() 管道 ###

### src ###
 用于产生数据流，参数为符合所提供的匹配模式（glob）或者匹配模式的数组的文件。
glob模式: 匹配路径与文件的模式，类似正则表达式。

* `*` 0-多个任意字符
* `？` 任意1个字符
* `[...]` 该路径段中在指定范围内字符
* `!(pattern|pattern|pattern)` 匹配除所给出的模型以外的情况
* `?(pattern|pattern|pattern)` 匹配所给出的模型中的 0 个或任意 1 个
* `+(pattern|pattern|pattern)` 匹配所给出的模型中的 1 个或者多个
* `*(pattern|pattern|pattern)` 匹配所给出的模型中的 0 个或多个或任意个的组合
* `@(pattern|pat*|pat?erN)` 匹配所给出的模型中的任意 1 个
* `**` 不仅匹配路径中的某一段,而且可以匹配 `a/b/c` 这样带有 / 的内容，所以，它还可以匹配子文件夹下的文件

### dest ###
将管道的输出写入文件，而且这些输出还可以继续输出（目录不存在，会新建）
```javascript
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```
`dest()` 可以接收对象作为第二个参数
```javascript
{
  cwd: '...', // 写入路径的基准目录
  mode: '...' // 权限
}
```

### task ###
用于定义任务，第一个参数是任务名，第二个参数是任务函数。其中有一个可选参数，参数为任务名，在当前任务执行前执行。
```javascript
gulp.task('watch', ['sass'], function () {
    // ...
});
```

### watch ###
用于指定需要监视符合所提供的匹配模式（glob）或者匹配模式的数组的文件。一旦这些文件发生变动，就运行指定任务。
```javascript
gulp.task('watch', function () {
   gulp.watch('templates/*.tmpl.html', ['build']);
});
```


### gulp 插件 ###
`gulp-rename`: 文件重命名
```javascript
var rename = require("gulp-rename");

// rename via string
gulp.src("./src/main/text/hello.txt")
  .pipe(rename("main/text/ciao/goodbye.md"))
  .pipe(gulp.dest("./dist")); // ./dist/main/text/ciao/goodbye.md

// rename via function
gulp.src("./src/**/hello.txt")
  .pipe(rename(function (path) {
    path.dirname += "/ciao";
    path.basename += "-goodbye";
    path.extname = ".md"
  }))
  .pipe(gulp.dest("./dist")); // ./dist/main/text/ciao/hello-goodbye.md

// rename via hash
gulp.src("./src/main/text/hello.txt", { base: process.cwd() })
  .pipe(rename({
    dirname: "main/text/ciao",
    basename: "aloha",
    prefix: "bonjour-",
    suffix: "-hola",
    extname: ".md"
  }))
  .pipe(gulp.dest("./dist")); // ./dist/main/text/ciao/bonjour-aloha-hola.md

  // process.cwd() 当前工作目录
```

`gulp-uglify`: 文件压缩
```javascript
gulp.task('js-uglify', function () {
    gulp.src(['./www/js/vendor.js', './www/js/app.join.js'])
        .pipe(uglify())
        .pipe(gulp.dest('./www/js'));
});
```

`gulp-sass`: 编译sass
```javascript
gulp.task('sass', function (done) {
    gulp.src('./scss/ionic.app.scss')
        .pipe(sass())
        .on('error', sass.logError)
        .pipe(gulp.dest('./www/css/'))
        .pipe(cleanCss({
            keepSpecialComments: 0
        }))
        .pipe(rename({extname: '.min.css'}))
        .pipe(gulp.dest('./www/css/'))
        .on('end', done);
});

gulp.task('watch', ['sass'], function () {
    gulp.watch(paths.sass, ['sass']);
});
```

`gulp-clean-css`: 压缩 CSS 文件
```javascript
gulp.task('sass', function (done) {
    gulp.src('./scss/ionic.app.scss')
        .pipe(sass())
        .on('error', sass.logError)
        .pipe(gulp.dest('./www/css/'))
        .pipe(cleanCss({
            keepSpecialComments: 0
        }))
        .pipe(rename({extname: '.min.css'}))
        .pipe(gulp.dest('./www/css/'))
        .on('end', done);
});

```


`gulp-imagemin`: 压缩jpg、png、gif等图片
```javascript
gulp.task('default', () =>
    gulp.src('src/images/*')
        .pipe(imagemin())
        .pipe(gulp.dest('dist/images'))
);

...
.pipe(imagemin([
    imagemin.gifsicle({interlaced: true}),
    imagemin.jpegtran({progressive: true}),
    imagemin.optipng({optimizationLevel: 5}),
    imagemin.svgo({
        plugins: [
            {removeViewBox: true},
            {cleanupIDs: false}
        ]
    })
]))

```

* `gifsicle` — Compress GIF images
* `jpegtran` — Compress JPEG images
* `optipng` — Compress PNG images
* `svgo` — Compress SVG images

`gulp-angular-templatecache`:
```javascript
var templateCache = require('gulp-angular-templatecache');

gulp.task('default', function () {
  return gulp.src('templates/**/*.html')
    .pipe(templateCache({stadalone:true})) // 不创建新的angular module
    .pipe(gulp.dest('public'));
});
```
把templates下的所有html文件保存到public/templates.js中

`gulp-inject`
```javascript
// 添加templates.js的引用到index.html
gulp.task('inject-templates', function (done) {
    gulp.src('./www/index.html')
        .pipe(inject(gulp.src('./www/js/templates.js', {read: false}), {relative: true}))
        .pipe(gulp.dest('./www'))
        .on('end', done);
});
```

`gulp-useref`
```javascript
// 合并index.html中引用的需要合并的js和css文件
gulp.task('useref', function(done){
    var assets = useref.assets();
    gulp.src('./www/index.html')
        .pipe(assets)
        .pipe(assets.restore())
        .pipe(useref())
        .pipe(gulp.dest('./www'))
        .on('end', done);
});
```

`gulp-ng-annotate`
```javascript
var ngAnnotate = require('gulp-ng-annotate');

// 严格依赖注入
gulp.task('annotate', function () {
    return gulp.src(['./www/js/vendor.js', './www/js/app.join.js'])
        .pipe(ngAnnotate({single_quotes:true}))
        .pipe(gulp.dest('./www/js'));
});
```

`fs` 和 `path` node.js 模块
```javascript
var fs = require('fs');
var path = require('path');

var ionicDir = path.resolve(__dirname, "./www/lib/ionic");
// 将一系列路径或路径段解析为绝对路径

fs.existsSync() //判断是否存在目录
fs.readdirSync() //读取目录
fs.lstatSync() //文件信息
fs.rmdirSync() //删除空文件夹
fs.unlinkSync() //删除文件
```


### 参考链接 ###
[gulp入门](https://juejin.im/post/593cf6efac502e006b3e2bc0)

[gulp官方文档](https://github.com/gulpjs/gulp)
