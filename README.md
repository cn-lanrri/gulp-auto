## gulp前端自动化构建环境的搭建

**为了提高项目编码质量，我们需要将Web前端使用工程化方式构建一个简单的自动化环境；**

### **目前需要的功能：**

```
01. 创建项目文件目录
02. gulp同步执行任务
03. css中引入css文件
04. 支持css嵌套写法
05. css相同合并
06. px转rem（手机页面）
07. 编译SASS
08. css添加前缀
08. pug模板引擎
09. index生成时自动加入头部和底部代码
10. index生成时自动修改title代码
11. 文件压缩后利于查看与调试
12. 自动刷新
```

### 项目目录结构

```

├─ gulpfile.js            // gulp配置文件
├─ package.json           // 依赖模块json文件,npm install会安装项目所有的依赖模块
├─ README.MD              // 项目说明
├─ node_modules           // 依赖模块包目录
├─ layout			     // 用于index自动加入头部和底部代码
│   ├─ head.html	      // head.html 头部html代码
│   └─ foot.html	      // foot.html 底部html代码
└─ project                // 所有项目目录（project下所有的项目目录全部都是自动生成）
    └─ test 		   	 // 单个项目目录 (发布环境 和 预览环境)
        ├─ 1_dev       	  // 开发环境项目目录
	    │   ├─ css        // 开发环境里的样式文件
	    │   ├─ fonts      // 开发环境里的字体图标文件
	    │   ├─ images     // 开发环境里的图片文件
	    │   ├─ js         // 开发环境里的js文件
	    │   ├─ page       // 开发环境里的PUG组件文件
	    │   └─ index.pug  // 开发环境里的默认页面
	    ├─ css            // 预览环境里的样式文件
        ├─ images         // 预览环境里的图片文件
        ├─ js             // 预览环境里的js文件
        └─ index.html     // 预览环境里的默认启动页面
```

### gulpfile配置文件

#### 常用的插件

```
colors:			    设置控制台颜色
gulp-sequence:		同步执行任务
gulp-notify: 		加控制台文字描述用的
gulp-plumber:		监控错误
gulp-replace:		替换脚本
gulp-postcss:		postcss 处理css
autoprefixer:        浏览器前缀
postcss-import:		css引入css文件
postcss-nested:		支持css嵌套
css-mqpacker:		css相同合并
postcss-px2rem:		px转rem
gulp-sass:			sass css
gulp-sourcemaps:	文件压缩后利于查看与调试
gulp-pug:			pug 模板引擎
browser-sync:		自动刷新插件初始化
```

#### gulpfile.js内容

```js
/**
 * gulp 自动化构建工具
 * gulpfile.js 配置文件
 */
'use strict';
const
	fileName       = 'test',								// 文件夹名称
	projectTitle   = 'test title',							// 项目标题名称
	remUnit        = 64,									// 设计图的宽度/10 适用于手机页面开发
	distDir        = './project/'+fileName,					// 文件夹目录
	srcDir         = './project/'+fileName+'/1_dev',		// 开发文件目录

	fs             = require('fs'),							// 文件操作
	mkdirp 		   = require('mkdirp'),						// 创建文件目录
	path           = require('path'),						// 处理目录的对象
	gulp           = require('gulp'),						// 自动化构建工具
	colors 		   = require('colors'),						// 设置控制台颜色
	gulp_sequence  = require('gulp-sequence'),				// 同步执行任务
	notify         = require('gulp-notify'), 				// 加控制台文字描述用的
	plumber 	   = require('gulp-plumber'),				// 监控错误
	rename         = require('gulp-rename'),            	// 文件重命名
	replace        = require('gulp-replace'),				// 替换脚本
	gulp_postcss   = require('gulp-postcss'),				// postcss 处理css
	autoprefixer   = require('autoprefixer'),           	// 浏览器前缀
	cssimport      = require('postcss-import'),				// css引入css文件
	nested         = require('postcss-nested'),				// 支持css嵌套
	mqpacker       = require('css-mqpacker'),				// css相同合并
	px2rem         = require('postcss-px2rem'),				// px转rem
	sass           = require('gulp-sass'),					// sass css
	sourcemaps     = require('gulp-sourcemaps'),			// 文件压缩后利于查看与调试
	pug            = require('gulp-pug'), 					// pug 模板引擎
	// contentInclude = require('gulp-content-includer'), 	// 合并按模块引入html文件
	browserSync    = require('browser-sync').create(),		// 自动刷新插件初始化
	reload         = browserSync.reload;

var dirArray = [
	distDir,											// 项目目录
	srcDir,												// 项目开发目录
	srcDir + '/js/',									// 开发项目中JS源文件目录
	srcDir + '/css/', 									// 开发项目中SCSS源文件目录
	srcDir + '/fonts/', 								// 开发项目中字体图标源文件目录
	srcDir + '/images/', 								// 开发项目中图片源文件目录
	srcDir + '/page/', 									// 开发项目中PUG组件文件源文件目录
];
var indexPug = {
	file: srcDir+'/index.pug',							// index.pug 的文件路径
	head: '<!-- head -->',
	foot: '<!-- foot -->'
}
// 创建目录结构
var mkdirs = (dirpath, dirArray, _callback) => {
	console.log(colors.red("系统开始处理，请等待！"));
  	fs.exists( dirpath ,function(exists){
	    if(!exists){
	      mkdir(0, dirArray, function(){
	        console.log('项目文件创建完毕!准备写入文件!');
	        _callback();
	      });
	    }else{
	      console.log('项目文件已经存在!准备写入文件!');
	      _callback();
	    }
	});
	// 创建文件夹
	var mkdir = (pos, dirArray, _callback) => {
	    var len = dirArray.length || '';
	    if(pos >= len){
		    _callback();
		    return;
		}
		var currentDir;
	    for(var i= 0; i <=pos; i++){
		    currentDir = dirArray[i];
		}
		fs.exists(currentDir,function(exists){
		    if(!exists){
		      fs.mkdir(currentDir,function(err){
		        if(err){
		          console.log(currentDir+'创建文件夹出错！',err);
		        }else{
		          console.log(currentDir+'文件夹-创建成功！');
		          mkdir(pos+1,dirArray,_callback);
		        }
		      });
		    }else{
		      console.log(currentDir+'文件夹-已存在！');
		      mkdir(pos+1,dirArray,_callback);
		    }
		});
	}
}
// 写入index.pug头部和底部文件
var writeFile = (indexHtml) => {
	if(!indexHtml){
		indexHtml = indexPug.head +'\n\n'+ indexPug.foot;
	}
	console.log("准备写入index.pug头部和底部文件");
	fs.writeFile(indexPug.file, indexHtml, (err) => {
	    if (err) return console.error(err);
	    console.log("数据写入成功！");
	});
};
// 创建完项目后回调执行
var callback = () => {
	fs.open(indexPug.file, 'r+', (err, fd) => {
	    console.log("--------我是分割线-------------");
	    if(err){
	    	writeFile();
	    }else{
		    	console.log("index.pug文件已存在！");
			var data = fs.readFileSync(indexPug.file);
			if(data.toString().indexOf(indexPug.head) == -1){
				console.log("index.pug文件中头部和底部文件不全！");
				writeFile(indexPug.head +'\n'+ data.toString());
			}
			if(data.toString().indexOf(indexPug.foot) == -1){
				console.log("index.pug文件中头部和底部文件不全！");
				writeFile(data.toString() +'\n'+ indexPug.foot);
			}
	    }
	    console.log("--------我是分割线-------------");
	    console.log(colors.yellow("1s后开始执行自动化构建。。。"));
	});
};
// 创建项目所需目录
gulp.task('mkdirs', () => {
	mkdirs(distDir,dirArray,callback);

	// gulp创建项目目录方法
	// dirArr.forEach(dir => { mkdirp.sync(dir) });
});
/**
 * 错误输出
 * @param {any} error
 */
var onError = (error) => {
    var title = error.plugin + ' ' + error.name;
    var msg = error.message;
    var errContent = msg.replace(/\n/g, '\\A '); // replace to `\A`, `\n` is not allowed in css content
    // system notification
    notify.onError({
        title: title,
        message: errContent,
        sound: true
    })(error);
    // prevent gulp process exit
    this.emit('end');
};
// postcss 插件库
var plugins = (device) => {
	var plugin;
	if(device == 'mobile'){
		plugin = [
			autoprefixer,
			cssimport,
			nested,
			mqpacker,
			px2rem({ remUnit: remUnit })
		];
	}else{
		plugin = [
			autoprefixer,
			cssimport,
			nested,
			mqpacker,
		];
	}
	return plugin;
}
// pug 模板引擎 转为html
gulp.task('pug', () => {
	return gulp
		.src([srcDir+'/*.pug',srcDir+'/page/*.pug'])
		.pipe(plumber(onError))
	    .pipe(pug({ pretty:true }))
        .pipe(replace(/<!-- head -->/, fs.readFileSync('./layout/header.html', 'utf-8')))
        .pipe(replace(/<!-- foot -->/, fs.readFileSync('./layout/footer.html', 'utf-8')))
        .pipe(replace(/PROJECTTITLE/, projectTitle))
	    .pipe(gulp.dest(distDir+'/'))
    	.pipe(reload({ stream: true }));
});
// postcss 处理css
gulp.task('postcss', () => {
	return gulp
		.src([srcDir+'/css/**','!'+srcDir+'/css/icons.scss'])
	 	.pipe(plumber(onError))
	 	.pipe(sourcemaps.init())
        .pipe(sass().on('error', sass.logError))
        .pipe(sourcemaps.write())
	 	.pipe(gulp_postcss(plugins))
	 	.pipe(gulp.dest(distDir+'/css/'))
	 	.pipe(reload({ stream: true }));
});
// 把开发版本的图片和js移动到线上版中
gulp.task('copyimage', () => {
    return gulp.src(srcDir + '/images/**')
        .pipe(gulp.dest(distDir + '/images/'))
});
gulp.task('copyjs', () => {
    return gulp.src(srcDir + '/js/**')
	 	.pipe(plumber(onError))
        .pipe(gulp.dest(distDir + '/js/'))
        .pipe(reload({ stream: true }));
});
// 先执行创建项目目录，等待0.5秒再异步执行其他任务
gulp.task('sequence', ['mkdirs'], (cb) => {
	setTimeout(() => {
		gulp_sequence(['pug','postcss','copyimage','copyjs'], cb);
	},800);
});
// 启动
gulp.task('default',['sequence'], (cb) => {
    browserSync.init({ server: distDir });
    gulp.watch(srcDir + '/css/**', ['postcss']);
    gulp.watch(srcDir + '/*.pug', ['pug']);
    gulp.watch(srcDir + '/js/*.js', ['copyjs']);
    console.log(colors.green.bold('感谢您的等待，请开始编写项目。。。'));
});
```

#### 启动项目

```cmd
gulp
```

![gulp](http://minisite.hocodo.com/test/lanrri/web/gulp.png)

### [项目地址](https://github.com/cn-lanrri/gulp-auto)

```
git clone git@github.com:cn-lanrri/gulp-auto.git
```

