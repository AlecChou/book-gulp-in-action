# Gulp.js 筆記

## 建置工具可以幫我們什麼？

自動批次執行較常見的前端工作處理，像是：

* 預先編譯 SASS / Coffee
* Lint 語法檢查
* 測試
* 多檔打包與最小化
* 為檔名上版號
* 監控檔案修改
* 即時重整 (LiveReload)

一般來說我們可以稱它們為「任務 (task) 」

## 建置情境

將 SASS 檔編譯成 CSS 檔。

* 來源檔： `assets/styles/*.scss`
* 目的地： `public/css`

### 有哪些選擇？

* Make
* Grunt
* Gulp

### Make

使用 Shell Script ，多數作業系統都可直接使用。

範例：

```bash
# Makefile
SASSDIR=assets/styles
CSSDIR=public/css
CSS := $(addprefix $(CSSDIR)/, main.css login.css)

# Default target
all: dist $(CSS)

# Clean and make directory
dist:
	rm -rf $(CSSDIR)
	mkdir -p $(CSSDIR)

# Compile *.scss
$(CSSDIR)/%.css: $(SASSDIR)/%.scss
	sass -C --style compressed $^ $@
```

* 語法不易學習，較難被 Web 開發人員接受。
* 沒有整理好的 patterns 可供套用，程式會因為一些流程上的小改動而變得複雜。

### Grunt

**The JavaScript Task Runner / Code over Configuration**

範例：

```js
// gruntfile.js
module.exports = function(grunt) {
  grunt.initConfig({
    sass: {
      dist: {
        options: { style: 'expanded' },
        files: [{
          expand: true,
          cwd:  './assets/styles', src: ['*.scss'],
          dest: './public/css',    ext: '.css'
        }]
      }
    }
  });
  grunt.loadNpmTasks('grunt-contrib-sass');
  grunt.registerTask('default',['sass']);
}
```

* 執行於有安裝 node.js 的環境。
* 提供豐富的 plugins 。
* 使用設定選項來定義任務流程。
* 必須要靠暫存目錄來放置處理中的檔案。
* 設定複雜且不易看出任務流程。

### Gulp

**The streaming build system / Code over Configuration**

範例：

```js
// gulpfile.js
var gulp = require('gulp'),
    rubySass = require('gulp-ruby-sass');

gulp.task('default', function() {
    return gulp.src('assets/styles/*.scss')
        .pipe(rubySass({ style: 'compressed' }))
        .pipe(gulp.dest('public/styles'));
});
```

* 執行於有安裝 node.js 的環境。
* 提供豐富的 plugins 。
* 基於 streaming 的檔案處理。
* 任務的動作在記憶體裡完成。
* 任務行為與流程容易理解。

## Gulp 基本工作流程

* 讓專案支援 gulp
* 撰寫 `gulpfile.js`
* 執行 `gulp` 指令

### gulpfile.js

範例：

```js
var gulp = require('gulp');

gulp.task('default', function () {
    return gulp.src('sources')
        .pipe(...)
        .pipe(...)
        .pipe(gulp.dest('destination'));
});
```

![inline](gulp-basic-flow.svg)

* 定義任務 (task)
* 在任務中設定要處理的檔案來源 (source)
* 用 pipe 方式處理檔案來源
* 將處理結果寫入目的地 (destinaion)

### 執行

指令：

```bash
$ cd /path/to/project
$ npm install gulp
$ gulp
```

* 切換到專案根目錄。
* 安裝 gulp 模組。
* 執行 `gulp` 指令。
* `gulp` 指令會找到 `gulpfile.js` 中的 `default` 任務來執行。

### gulp.task

用來定義一個任務。

```js
// 任務名稱： styles
gulp.task('name', ['deps'], function(done) {
    return stream || promise;
    // ...or, call done()
});
```

* 第一個參數是任務名稱。
* 第二個參數可以設定依賴的任務，必須是陣列，如果沒有可省略。
* 第三個參數 (如無依賴的任務，即變成第二個參數) 是任務的 callback 。
* 任務執行後可回傳一個 stream 或 promise ，另外也可以呼叫 done 這個 callback 做為結束。

### gulp.src

取得檔案來源的 stream 。

```js
gulp.src(['styles/**/*.scss', '!styles/partial/*.scss'])
	.pipe(...)
	.pipe(...);
```

* 第一個參數可為字串或字串陣列。
* 字串中支援萬用字元 `*` 。
* `**` 表示所有目錄，也包含沒有。
* 字串前端加入 `!` 表示要排除。
* `pipe` 方法可置入 plugin 實體來處理 stream 。

### gulp.dest

將 stream 中的檔案存到目的路徑裡。

```js
gulp.src('assets/styles/**/*.scss')
	.pipe(gulp.dest('public/css'));
```

* `gulp.dest` 要當做 `pipe` 方法的參數，以接收處理後的 stream 檔案。
* 第一個參數必須是目錄路徑，若不存在會自動建立。

### gulp.watch

* 監看指定的檔案，如果有修改的話就呼叫指定的任務。
* 第一個參數規則同 `gulp.src` 。
* 第二個參數為任務陣列。

```
gulp.watch('assets/styles/**/*.scss', ['styles']);
```

## Node 小知識

### package.json

* 記錄專案會用到的 node 模組。

```json
{
  "name": "my-project",
  "version": "0.0.0",
  "private": true,
  "devDependencies": {
	// ...
  },
  "engines": {
    "node": ">=0.10.0"
  }
}
```

### npm

* 管理 node 套件

 安裝套件，並寫入 package.json 的 `devDependencies` 區段。

 ```bash
$ npm install plugin-name --save-dev
```

* 依照 package.json 上的 `devDependencies` 定義來安裝所有相依套件：

 ```bash
$ npm install
```

## 共用的任務

* 載入啟動時需要的模組
* 編譯 SASS
* 編譯 Browserify 化的 JavaScript
* 編譯 Jade 樣板
* 圖檔最佳化
* 複製字型

### 自動載入 plug-in

gulp-load-plugins 是用來自動載入以 `gulp-` 開頭的 gulp plugins 。

原本沒有自動載入時，要手動 require 每個 module ：

```js
var rubySass     = require('gulp-ruby-sass');
var autoprefixer = require('gulp-autoprefixer');
var jade         = require('gulp-jade');

.pipe(rubySass({ ... }))
.pipe(autoprefixer('....'))
.pipe(jade({ ... })
```

用 gulp-load-plugins 就可以改為：

```js
var $ = require('gulp-load-plugins')();

.pipe($.rubySass({ ... }))
.pipe($.autoprefixer('....'))
.pipe($.jade({ ... })

```

### 編譯 SASS

將 `assets/styles/*.scss` 編譯到 `public/css` ：

```js
// Styles
gulp.task('styles', function() {
    return gulp.src('assets/styles/*.scss')
        .pipe($.plumber())
        .pipe($.rubySass({
            compass: true,
            style: 'compressed'
        }))
        .pipe($.autoprefixer('last 1 version'))
        .pipe(gulp.dest('public/styles'))
        .pipe($.size({ title: 'styles' }));
});
```

* gulp-plumber 可以攔截錯誤訊息，並轉換成較清楚的格式。
* gulp-ruby-sass 將 SASS 編譯成 CSS 。
* gulp-autoprefix 自動加上 vendor prefix
* gulp-size 可顯示處理過的檔案大小總計， `title` 可以明確標示處理的是什麼檔案。

### 編譯 Browserify 化的 JavaScript

將 `assets/scripts/*.js` 編譯到 `public/js` ：

```js
var browserify = require('browserify');
var transform = require('vinyl-transform');

// Scripts
gulp.task('scripts', function() {

    var browserified = transform(function(filename) {
        var b = browserify(filename);
        return b.bundle()
            .on('error', function (err) {
                console.log(err.toString());
                this.emit('end');
            });
    });

    return gulp.src('./assets/scripts/*.js')
        .pipe($.jshint('.jshintrc'))
        .pipe($.jshint.reporter('jshint-stylish'))
        .pipe(browserified)
        .pipe(gulp.dest('public/scripts'))
        .pipe($.size({ title: 'scripts' }));
});
```

* gulp-jshint 會透過 `.jshintrc` 中的設定來檢查 JS 檔案語法，回報格式則用 jshint-stylish 美化。
* browserify 可以將 js 檔裡以 `require` 引用的外部模組合併進來。
* transform 會把 browserify bundle 後的結果包裝成 through object 供 `pipe` 使用。

### 編譯 Jade 樣板

將 `assets/templates/*.jade` 編譯到 `public` ：

```js
// Templates
gulp.task('views:develop', function() {
    return gulp.src('assets/templates/*.jade')
        .pipe($.plumber())
        .pipe($.jade({ pretty: true }))
        .pipe(gulp.dest('public'))
        .pipe($.size({ title: 'views:develop' }));
});
``` 

* gulp-jade 會將 Jade 樣版編譯成 HTML 檔案， `pretty` 會產生較美觀的 HTML 碼排版。

### 圖檔最佳化

將 `assets/images` 下所有圖片做最佳化，然後放到 `public/images` ：

```js
// Images
gulp.task('images', function() {
    return gulp.src('assets/images/**/*')
        .pipe($.plumber())
        .pipe($.cache($.imagemin({
            optimizationLevel: 3,
            progressive: true,
            interlaced: true
        })))
        .pipe(gulp.dest('public/images'))
        .pipe($.size({ title: 'images' }));
});
```

* gulp-cache 會暫存圖檔處理後的結果，避免重複處理。
* gulp-imagemin 會最佳化圖檔，可以給它最佳化參數。
* 發生圖檔無法處理的時候，通常只要用其他軟體再存一次即可。

### 複製字型

將 `public/bower_components` 下的第三方套件字型，全部複製到 `public/fonts` ：

```
// Fonts
gulp.task('fonts', function () {
    return gulp.src('public/bower_components/**/fonts/**/*.{otf,eot,svg,ttf,woff}')
        .pipe($.flatten())
        .pipe(gulp.dest('public/fonts'))
        .pipe($.size({ title: 'fonts' }));
});
```

* gulp-flatten 可以去掉檔案所在路徑，只保留檔案名稱。

## 開發時用的任務

* 清除暫存檔
* 重新編譯
* 啟動開發用 Web Server
* 監看檔案變化

### 清除暫存檔

刪除掉編譯過程中的暫存檔：

```js
var del = require('del');

// Clean
gulp.task('clean:develop', function(cb) {
    del([
        'public/**/*.html',
        'public/styles',
        'public/scripts',
        '.sass-cache'
    ], cb);
});
```

* del 可以一次刪除掉多個資料夾或檔案，最後呼叫 cb 來結束任務。
* cb 只能被執行一次，所以不能重複呼叫。

### 重新編譯

先清掉暫存檔，然後重新編譯所有的 assets ：

```js
var runSequence = require('run-sequence');

// Prepare for development
gulp.task('prepare:develop', function (cb) {
    runSequence(
        'clean:develop',
        [
            'views:develop',
            'styles',
            'scripts',
            'images',
            'fonts'
        ],
    cb);
});
```

* run-sequence 可以依順序執行 gulp 的任務，如遇到任務陣列就會平行執行，最後呼叫 cb 來結束任務。

### 啟動開發用 Web Server

當不需要執行任何 PHP 程式碼時，可以啟用一個靜態 Web Server

```js
// Start Web Server
gulp.task('serve', function() {
    gulp.src('public')
        .pipe($.webserver({
            livereload: true,
            open: true
        }));
});
```

* gulp-webserver 可以對指定的路徑啟動一個支援 LiveReload 的靜態 Web Server 。

### 監看檔案變化

當來源檔案有變化時，要即時做編譯：

```js
// Watch
gulp.task('watch', ['prepare:develop'], function() {
    gulp.start('serve');
    gulp.watch('assets/templates/**/*.jade', ['jade']);
    gulp.watch('assets/styles/**/*.scss', ['styles']);
    gulp.watch('assets/scripts/**/*.js', ['scripts', 'jest']);
    gulp.watch('assets/images/**/*', ['images']);
});
```

* 監看前要預先編譯一次 (`prepare:develop`) ，再啟動 Web Server (`serve`) 。
* 當發現 Jade 樣版、 SCSS 、 JS 及圖檔有更改時，就呼叫對應的任務執行。

## 佈署前的任務

* 打包與加上版本
* 清理暫存檔

### 打包與加上版本

在佈署前要把所有 JS / CSS 都分別打包成一個檔案並最小化；同時為了避免快取，也要加上版號。

```js
// HTML
gulp.task('views:build', ['views:develop', 'styles', 'scripts', 'images', 'fonts'], function() {

    var jsFilter = $.filter('**/*.js');
    var saveLicense = require('uglify-save-license');
    var useref = $.useref;
    var assets = useref.assets({
            searchPath: 'public'
        });

    return gulp.src('public/**/*.html')
        .pipe($.plumber())
        .pipe(assets)
        .pipe(jsFilter)
        .pipe($.uglify({ preserveComments: saveLicense }))
        .pipe(jsFilter.restore())
        .pipe($.rev())
        .pipe(useref.restore())
        .pipe(useref())
        .pipe($.revReplace())
        .pipe(gulp.dest('public'))
        .pipe($.size({ title: 'views' }));
});
```

* gulp-useref 的 `useref.assets` 方法會找出 `<!-- build:xxx filename -->` 與 `<!-- endbuild -->` 之間的 CSS 或 JS 檔，中間可以先對 JS 檔做最小化處理； `useref.restore` 會還原原來的 HTML 檔案 stream ；最後 `useref` 會將 CSS 檔或 JS 檔打包成一個檔案，並以輸出到指定的 `filename` 上。

 ```html
<!-- build:css css/main.css-->
<link rel="stylesheet" href="bower_components/font-awesome/css/font-awesome.css"/>
<link rel="stylesheet" href="bower_components/ionicons/css/ionicons.css"/>
<link rel="stylesheet" href="bower_components/bootstrap-daterangepicker/daterangepicker-bs3.css"/>
<link rel="stylesheet" href="bower_components/nvd3/nv.d3.css"/>
<link rel="stylesheet" href="styles/main.css"/>
<!-- endbuild-->
```

 會輸出為：

 ```html
<link rel="stylesheet" href="css/main.css">
```

* gulp-rev 會找出 `link` 或 `script` 標籤指向的 CSS 或 JS 檔，然後依照修改內容在它們的檔名上加上版本號。

 例如 `css/main.css` 重新命名為 `css/main-3db5111c.css ` 。

* gulp-rev-replace 將 HTML 中的 CSS 或 JS 檔替換已有版本號的檔名：

 ```html
<link rel="stylesheet" href="css/main.css">
```

 替換為：

 ```html
<link rel="stylesheet" href="css/main-3db5111c.css">
```

* gulp-filter 可以先找出特定的檔案來處理，然後再用 `restore` 方法還原成原來要處理的檔案。
* gulp-uglify 可以將 JS 檔做最小化，搭配 uglify-save-license 可以保留套件的版權宣告。 

### 清理快取與暫存檔

清理圖檔快取：

```js
// Clean Cache
gulp.task('clean:cache', function (cb) {
    return $.cache.clearAll(cb);
});
```

建置前清理所有已編輯的檔案：

```js
// Clean
gulp.task('clean:build', ['clean:develop', 'clean:cache'], function(cb) {
    del([
        'public/css',
        'public/js',
        'public/fonts',
        'public/images'
    ], cb);
});
```

清理編輯過程中的暫存檔：

```js
// Clean temporary assets
gulp.task('clean:temporary', function (cb) {
    del([
        'public/styles',
        'public/scripts',
        '.sass-cache'
    ], cb);
});
```

### 預設任務

先清理舊的編譯與暫存檔，然後重新建置：

```js
// Build
gulp.task('default', function(cb) {
    runSequence(
        'clean:build',
        'views:build',
        'clean:temporary',
    cb);
});
```

## 不進 Repository 的檔案

在 .gitignore 中加入：

```
/public/*.html
/public/styles
/public/scripts
/public/css
/public/js
/public/fonts
/public/images
/public/bower_components
/node_modules
.bundle
.sass-cache
.tmp
.DS_Store
Thumbs.db
```

## 流程

開發中：

```bash
$ gulp watch
```

![inline](gulp-basic-flow.svg)

佈署前：

```bash
$ gulp
```

![inline](gulp-basic-flow.svg)

## 結合 Laravel

前面提到的是 Prototype 的流程，如果要與 Laravel 結合，就要做以下調整：

* Jade 樣版改為　Blade 樣版
* 清理編譯的 Blade 樣版
* 使用 Laravel 的 Web Server
* 佈署前的編譯
* 清理暫存檔

### Jade 樣版改為　Blade 樣版

1. 先將原來的　`app/views` 更名為 `app/templates` 。

2. 任務　`views:develop` 改成將　`app/templates/*.blade.php` 複製到 `app/views` ：

```
// Templates
gulp.task('views:develop', function() {
    return gulp.src('app/templates/**/*.blade.php')
        .pipe($.plumber())
        .pipe(gulp.dest('app/views'))
        .pipe($.size({ title: 'views:develop' }));
});
```

### 清理編譯的 Blade 樣版

任務 `clean:develop` 原本清理 `public/**/*.html` 改成清理 `app/views` ：

```js
// Clean
gulp.task('clean:develop', function(cb) {
    del([
        'app/views', // here
        'public/styles',
        'public/scripts',
        '.sass-cache'
    ], cb);
});
```

### 使用 Laravel 的 Web Server

因為 gulp-webserver 不提供執行 PHP 的功能，因此要改用 Laravel 提供的 Web Server 指令：

```js
// Start Web server
gulp.task('serve', function () {
    var spawn = require('child_process').spawn,
        child = spawn('php', [ 'artisan', 'serve' ], { cwd: process.cwd() }),
        log = function (data) { console.log(data.toString()) };
    child.stdout.on('data', log);
    child.stderr.on('data', log);
    process.on('exit', function () {
        child.kill();
    });
    process.on('uncaughtException', function () {
        child.kill();
    });
});
```

而因為 Laravel Web Server 不提供 LiveReload ，因此改用 gulp-livereload ：

```js
// Start Livereload server
gulp.task('livereload', function () {
    var server = $.livereload;
    server.listen();
    gulp.watch([
        'public/**/*',
        '!public/bower_components/**/*',
    ]).on('change', server.changed);
});
```

* gulp-livereload 要排除 `public/bower_components` 目錄，避免監控過多檔案。

### 佈署前的編譯

因為不使用 HTML 檔案，改用 `*.blade.php` ，所以修改如下：

1. `public/**/*.html` 改為 `app/views/**/*.blade.php` 。
2. CSS 與 JS 檔案，處理完後先放回 `public` 。
3. 在 gulp-rev-replace 要特別指定副檔名為 `.php` ，因為預設是用 `.html` 。
4. 最後輸出到 `app/views` ，這時候會連 `css` 和 `js` 都放在 `app/views` 下。

```js
// HTML
gulp.task('views:build', ['views:develop', 'styles', 'scripts', 'images', 'fonts'], function() {

    var jsFilter = $.filter('**/*.js');
    var saveLicense = require('uglify-save-license');
    var useref = $.useref;
    var assets = useref.assets({
            searchPath: 'public'
        });

    return gulp.src('app/views/**/*.blade.php')
        .pipe($.plumber())
        .pipe(assets)
        .pipe(jsFilter)
        .pipe($.uglify({ preserveComments: saveLicense }))
        .pipe(jsFilter.restore())
        .pipe(gulp.dest('public'))
        .pipe($.rev())
        .pipe(useref.restore())
        .pipe(useref())
        .pipe($.revReplace({
            replaceInExtensions: ['.php']
        }))
        .pipe(gulp.dest('app/views'))
        .pipe($.size({ title: 'views' }));
});
```

### 清理暫存檔

在任務 `views:build` 做完後，會留下 `app/views/css` 與 `app/views/js` 兩個無用的資料夾，因此要加到暫存檔的清除任務中：

```js
// Clean temporary assets
gulp.task('clean:temporary', function (cb) {
    del([
        'public/styles',
        'public/scripts',
        'app/views/css', // here
        'app/views/js', // here
        '.sass-cache'
    ], cb);
});
```

## 不佈署到線上的檔案

因為只會在本機開發，所以 Prototype 沒有佈署的問題；而有用到 Laravel 的專案在 rsync 時要排除掉：

```
node_modules
assets
public/bower_components
.gitignore
.sass-cache
app/database
app/storage
```

## 注意事項

* 除非 minify 有版本管理，否則任何透過編譯所產生的的檔案都不要放在 repository 中。
