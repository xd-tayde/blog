## Gulp简要入门笔记

### gulp:

是一款基于node的前端自动化构建工具，它可以工程化完成许多重复性的各种文件的测试、检查、合并、压缩、格式化、浏览器自动刷新、部署文件生成，并监听文件改动后重复这些步骤。因为这些过程其实是有规律可循，因此我们可以通过构建工具来帮助我们快速地自动化重复执行。

#### `流`的思想：

`gulp`使用了node中的流`((stream))`管道思想，使数据像流水一样通过管道进行输出与输入，管道便是 `.pipe( )`，使用内存来过渡，而不是像`grunt`那样直接使用硬盘的直接写入，因此拥有更优秀的性能。


#### 简要工作流程：

> 1、使用`gulp.src( )`找到目录下的 `.js`/`css`/`sass`等各种需要处理的 文件流；

> 2、通过`gulp.pipe( )`将文件流导入到各种插件中进行处理；

> 3、再次通过`gulp.pipe( )`将流传给`gulp.dest( )`,该指令可以将数据流写入到相应的目录文件中；

### 1、gulp的安装；

由于`gulp`依赖于`node`，因此在安装`gulp`之前需要先安装`node`;然安装之前可以先在终端中输入`node -v`用于确定环境是否已经安装node了。安装node可以通过直接在官网下载pkg安装包，`https://nodejs.org`;

安装完`node`后，会自带安装其包管理器`npm`，我们便可以通过它来进行`gulp`的安装了。

直接在终端中输入：`npm install -g gulp` ；

全局安装完`gulp`后，还需要在每个项目中单独安装，终端中进入到项目目录，执行终端命令：

	npm install gulp

如果想创建时直接写入`package.json`，是使用：

	npm install --save-dev gulp


### 2、gulp API学习

`gulp`的API十分的简介清晰，常用的主要有：
`gulp.src()`，`gulp.pipe()`,`gulp.task()`,`gulp.dest()`,`gulp.watch()`,`gulp.run()`;

####  `gulp.src(globs[, options])`:  

`globs`：字符串或数组，是文件匹配模式，可以通过文件路径来定位文件，获取文件流(stream)，后可以使用pipe管道输入到各种插件中；

`[options]`：对象，可选参数；里面包含多个属性；

`options.buffer`：类型： Boolean   默认值： true

如果该项被设置为 false，那么将会以 stream 方式返回 file.contents 而不是文件 buffer 的形式。这在处理一些大文件的时候将会很有用。**注意：**插件可能并不会实现对 stream 的支持。

`options.read`：类型： Boolean  默认值： true

如果该项被设置为 false， 那么 file.contents 会返回空值（null），也就是并不会去读取文件。

`options.base`：类型： String  

设置基准路径，影响导出文件路径，会使用传入`dest()`的路径替换掉 原始路径中`base`部分。如

	gulp.src('client/js/**/*.js') // 匹配 'client/js/somedir/somefile.js' 并且将 `base` 解析为 `client/js/`
	  .pipe(minify())
	  .pipe(gulp.dest('build'));  // 写入 'build/somedir/somefile.js'
	
	gulp.src('client/js/**/*.js', { base: 'client' })
	  .pipe(minify())
	  .pipe(gulp.dest('build'));  // 写入 'build/js/somedir/somefile.js'
		  
		  
#### `gulp.dest(path[, options])`：

该方法是可以将`pipe`导入的数据流写入对应的文件中。可以同时写入多个文件；如：
	
	gulp.src('./client/templates/*.jade')
	  .pipe(jade())
	  .pipe(gulp.dest('./build/templates'))
	  .pipe(minify())
	  .pipe(gulp.dest('./build/minified_templates'));
	  
#### `gulp.task(name[, deps], fn)`:

该方法是用于定义`gulp`任务；
	
	gulp.task('taskname', function() {
	  // 做一些事
	});

`deps`：类型：数组 可选参数，可定义一系列任务列表，将会在任务之前执行；

	gulp.task('mytask', ['array', 'of', 'task', 'names'], function() {
	  // 做一些事
	});

> Tip：列表中的任务将被并行执行；
	
#### `gulp.watch(glob[, opts], tasks)`:

监视文件的变化，如果变化，则执行 `tasks`任务；

	var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
	watcher.on('change', function(event) {
	  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
	});

####  `gulp.watch(glob[, opts, cb])`:

	gulp.watch('js/**/*.js', function(event) {
	  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
	});

#### 总结 gulp API:

> gulp.task： 表示创建gulp任务；

> gulp.src：输入的目录，通过 `*.js`可以匹配到js目录下的所有js文件；

> gulp.pipe：管道指令，可以理解为加入执行队列；

> gulp.dest：处理后的文件输出的目录；

> gulp.watch：实时监控文件改变，一旦改变则马上执行任务；

### 3、自动加载配置中的插件，gulp-load-plugins；

我们可以在项目中为`gulp`保存一份配置文件：`package.json`，然后使用`gulp-load-plugins`这个插件便可以一次性加载配置中保存的所有插件；

	var gulp = require('gulp');
	//加载gulp-load-plugins插件，并马上运行它
	var plugins = require('gulp-load-plugins')();

### 4、使用gulp进行JS的压缩，gulp-uglify；

1、在项目中新建一个gulpfile.js文件，用于写入gulp配置代码；

2、安装`gulp-uglify`模块，终端输入：`npm install --save-dev gulp-uglify`;

3、在gulpfile.js中载入gulp模块

	// 获取 gulp
	var gulp = require('gulp');

4、载入`gulp-uglify`模块：

	// 获取 uglify 模块（用于压缩 JS）
	var uglify = require('gulp-uglify');

5、创建`gulp`任务；

	// 压缩 js 文件
	gulp.task('javascript', function() {
	    // 1. 找到文件
	    gulp.src('js/*.js')
	    // 2. 压缩文件
	        .pipe(uglify())
	    // 3. 另存压缩后的文件
	        .pipe(gulp.dest('dist/js'))
	});

6、终端执行，在终端中使用 `cd`进入到项目目录中，输入命令：`gulp javascript`;如果得到输出结果为：

	gulp script
	[13:34:57] Using gulpfile ~/Documents/code/gulp-book/demo/chapter2/gulpfile.js
	[13:34:57] Starting 'script'...
	[13:34:57] Finished 'script' after 6.13 ms

则任务执行成功；在目录的dist/js下，便可以看到已经压缩后的js文件；

### 5、压缩HTML，gulp-minify-html：

使用`gulp-minify-html`插件用来压缩html文件。任务如下：
	
	var gulp = require('gulp'),
    minifyHtml = require("gulp-minify-html");

	gulp.task('minify-html', function () {
	    gulp.src('src/*.html') // 要压缩的html文件
	    .pipe(minifyHtml())    //压缩
	    .pipe(gulp.dest('dist/html'));
	});

### 6、压缩CSS，gulp-minify-css：

使用`gulp-minify-css`插件用来压缩css文件，任务如下：

	var gulp = require('gulp'),
	    minifyCss = require("gulp-minify-css");
	
	gulp.task('minify-css', function () {
	    gulp.src('src/*.css') // 要压缩的css文件
	    .pipe(minifyCss())    //压缩css
	    .pipe(gulp.dest('dist/css'));
	});

### 7、压缩图片，gulp-imagemin；

	var gulp = require('gulp');
	var imagemin = require('gulp-imagemin');
	var pngquant = require('imagemin-pngquant'); //png图片压缩插件
	gulp.task('default', function () {
	    return gulp.src('src/images/*')
	        .pipe(imagemin({
	            progressive: true,
	            use: [pngquant()] //使用pngquant来压缩png图片
	        }))
	        .pipe(gulp.dest('dist'));
	});
	
### 8、文件的重命名，gulp-rename；

`gulp`会以输入的文件流的名称来命名处理后的文件名，因此，如果需要重命名文件时，可以使用 `gulp-rename`插件；

	var gulp = require('gulp'),
    rename = require('gulp-rename'),
    uglify = require("gulp-uglify");

	gulp.task('rename', function () {
	    gulp.src('src/1.js')
	    .pipe(uglify())           //压缩
	    .pipe(rename('1.min.js')) //会将1.js重命名为1.min.js
	    .pipe(gulp.dest('js'));
	});

### 9、JS代码检查，使用 gulp-jshint；

	var gulp = require('gulp'),
    jshint = require("gulp-jshint");
	
	gulp.task('jsLint', function () {
	    gulp.src('src/*.js')
	    .pipe(jshint())
	    .pipe(jshint.reporter()); // 输出检查结果
	});

### 10、文件的合并，gulp-concat；
	
	var gulp = require('gulp'),
    concat = require("gulp-concat");
	gulp.task('concat', function () {
	    gulp.src('src/*.js')     //要合并的文件
	    .pipe(concat('all.js'))  // 合并匹配到的js文件并命名为 "all.js"
	    .pipe(gulp.dest('dist/js'));
	});

### 11、自动刷新，实时反馈代码的改变，gulp-livereload；
	
	var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload');
	gulp.task('less', function() {
	  gulp.src('less/*.less')
	    .pipe(less())
	    .pipe(gulp.dest('css'))
	    .pipe(livereload());
	});
	gulp.task('watch', function() {
	  livereload.listen(); //要在这里调用listen()方法
	  gulp.watch('less/*.less', ['less']);
	});



该笔记参考于 `Nimo Chu`的github与前端乱炖社区文章《学习前端自动化构建工具Gulp》以及`gulp`官方文档；

