### Webpack简介
```
为什么需要webpack：
因为现在的前端技术太多了，什么vue，less，react等等，浏览器是无法直接识别的，就需要webpack来打包处理

可以压缩资源文件，识别less，编译vue等等作用，还能处理兼容性js和css

之前是这种引入方式：
<script src="js/jquery.js"></script>
...一堆js
现在就两个js
<script src="js/vendor.js"></script>
<script src="js/index.js"></script>

.scss, .sass -> .css
typescript, cofferscript -> .js
所有js，css，图片等都连接到main.js
会将一堆资源引入变为几个static assets： .js, .css, .jpg, .png

需要
nvm（管理nodejs版本工具，切换nodejs版本），
nodejs，
npm（nodejs套件管理工具），

类似打包工具的还有grunt以及gulp，不过没有webpack流行

具体的使用感觉和maven之类dependency管理软件或者chef之类的devops很像，有特定语法，每个插件由不同人开发，也会有不同的配置方式，都需要分别研究。针对的是nodejs的前后端分离的前端程序，通过tomcat之类跑的不确定好不好用。

https://juejin.cn/post/6844903609587466253
https://segmentfault.com/a/1190000006178770
```

尚硅谷的youtube课程很好，免费还贴心的提供[课件](https://s8jl-my.sharepoint.com/:f:/g/personal/atguigu_s8jl_onmicrosoft_com/EosoaaVTfItBvvGjXpMhIuIBlaCakNGav2mB8ibDNsfdYQ?e=pX7U21)，很感谢

这里是学习笔记了

### 01 课程介绍
```
Nodejs 10 版本以上
webpack 4.26 版本以上
```

### 02 webpack简介
```
1）CSS代码：
less可以包裹使用，比如这么写
html {
	#title {
	}
}
不过less是需要解析为css才能浏览器识别的

2）JS代码：
在安装完nodejs就可以这么下载包了
$ npm i jquery

//引入js资源
import $ from 'jquery';
//引入样式资源
import './index.less';
也无法识别，一些新的语法或者其他语言可能就无法识别

所以前端提出了构建工具的概念，Webpack就是一种，是一个静态模块打包器。
会以为indexjs为入口，里面的资源形成chunk，将资源js/json/css/img/less/...进行打包，形成bundle。
```

### 03 webpack五个核心概念
```
1）Entry
指示webpack以那个文件为入口为起点开始打包，上面例子就是index.js
2）Output
指示webpack打包后资源bundles输出到哪里，如何命名
3）Loader
让webpack能处理那些非javascript文件（翻译官，webpack自身只理解javascript）
4）Plugins
执行范围更广的任务，比如打包优化和压缩等等

(Entry获得）Less -（通过loader）-> xxx -(通过压缩) -> bundle

5)Mode
包括development模式（能调试）和production模式
```

### 04 webpack的初体验
```
安装依赖
$ npm init
$ npm i webpack webpack-cli -g (全局安装）
$ npm i webpack webpack-cli -D (本地安装）

入口文件，新建index.js
webpack ./src/index.js -o ./build/build.js --mode=development
会以./src/index.js为入口文件开始打包，打包后输出到 ./build/built.js，整体打包环境是开发环境

生产环境的区别就是会压缩js代码，让代码不可读了。

json文件打包也没有问题，在index.js引入
import data from './data.json';
console.log(data);

css/img文件直接引入是不行的，比如下面的引入是无法打包的
import './index.css';

核心就是将ES6模块化编译成浏览器能识别的模块化
```

### 05 打包样式资源
```
webpack如何报道css/less呢？需要loader。
首先增加webpack.config.js，下面这个是webpack的配置文件（是commonjs格式）：

const { resolve } = require('path');
module.exports = {
	//入口起点
	entry: './src/index.js',
	//输出
	output: {
		//输出文件名
		filename: 'built.js',
		//输出路径，__dirname nodejs的变量，代表当前文件的目录绝对路径
		path: resolve(__dirname, 'build')
	},
	//loader的配置
	module: {
		//不同文件类型要配置不同的loader
		rules: [
			//详细loader配置
			{
				//匹配那些文件
				test: /\.css$/,
				//使用那些loader进行处理
				use: [
					//user数据中loader执行顺序：从右到左，从下到上依次执行
					//创建style标签，将js中的样式资源插入进行，增加到head中生效
					'style-loader',
					//将css文件变成commonjs模块加载js中，里面内容是样式字符串
					'css-loader'
				]
			},{
				test: /\.less$/,
				use:[
					'style-loader',
					'css-loader',
					//需要下载less-loader和less
					'less-loader'
				]
			}
		]
	},
	//plugins的配置
	plugins: [
		//详细plugins的配置
	],
	//模式
	mode: 'development',
}

//需要下载style-loader和css-loader
$ npm i webpack webpack-cli -D
$ npm i css-loader style-loader -D
$ npm i less-laoder less -D
$ webpack
```

### 06 打包html资源
```
同样在webpack.config.js定义，
html是需要通过plugins来处理的，

const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
		]
	},
	plugins: [
		//plugins的配置
		//html-webpack-plugin
		//功能：默认会创建一个空的HTML，自动引入打包输出的所有资源(JS/CSS)
		//需求：需要有结构的HTML文件
		new HtmlWebpackPlugin({
			//复制'./src/index.html' 文件，并自动引入打包输出的所有资源(JS/CSS)
			template: '.src/index.html'
		})
	],
	mode: 'development',
}

//下载依赖
$ npm i html-webpack-plugin -D
```

### 07 打包图片资源
```
index.js中
import './index.less';

webpack.config.js中
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			{
				test: /\.less$/,
				use: [
					'style-loader',
					'css-loader',
					'less-loader'
				]
			},
			{
				//处理图片资源
				test: /\.(jpg|png|gif)$/,
				//使用一个loader，需要下载url-loader file-loader
				loader: 'url-loader',
				options: {
					//图片大小小于8kb，就会被base64处理(字符串体积更大）
					limit: 8 * 1024,
					//问题：因为url-loader默认使用es6模块化解析，而html-loader引入图片是commonjs
					//解析时候会出现问题[object Module]
					//解决：关闭url-loader的es6模块化，使用commonjs解析，下面这行
					esModule: false,
					//给图片进行重命名
					//[hash:10]取图片的hash的前10位
					name: '[hash:10].[ext]'
				}
				//问题：默认处理不了html中的img图片，需要下面loader
			},
			{
				test: /\.html$/,
				//处理html文件的img图片（负责引入img，从而能被url-loader进行处理）
				//需要下载html-loader
				loader: 'html-loader'
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	mode: 'development',
}

//下载依赖
$ npm i url-loader file-loader html-loader -D
```

### 08 打包其他资源
```
比如字体资源的打包

html文件：
<body>
	<span class="iconfont icon-icon-test"></span>
</body>

index.js文件：
import 'iconfont.css';

webpack.config.js中：
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [
					'style-loader',
					'css-loader'
				]
			},
			//打包其他资源（除了html/js/css资源以外的资源）
			//原封不动的输出资源
			{
				//排除css/js/html资源
				exclude: /\.(css|js|html|less)$/,
				loader: 'file-loader'
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	mode: 'development',
}

就会在build里面生成一堆字体文件了
```

### 09 devServer
```
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	mode: 'development',
	//开发服务器devServer：用来自动化（自动编辑，自动打开浏览器，自动刷新浏览器）
	//特点：智慧在内存中编译打包，不会有输出
	//启动devServer指令为： npx webpack-dev-server
	devServer : {
		//项目构建后路径
		contentBase: resolve(__dirname, 'build'),
		//启动gzip压缩
		compress： true,
		//端口号
		port: 3000,
		//自动打开浏览器
		open: true
	}
}

//下载依赖
$ npm i webpack-dev-server -D
//这么运行webpack
$ npx webpack-dev-server
这样每次一改变webpack里面的代码，就自动实时编译了，很方便
```

### 10 开发环境基本配置
```
大综合，将之前各个资源的配置整合起来的demo

所有的资源都在index.js定义，如果import的css里面包含了其他资源也会一起打包了，不过资源的路径需要是相对路径才行。
如果更改了目录也一样需要手动修改所有资源内部的路径。

两种执行指令：
1） $webpack 有console输出，用于调试
2) $npx webpack-dev-server 在内存，适合写业务逻辑的时候

const{resolve}=require('path');constHtmlWebpackPlugin=require('html-webpack-plugin');
module.exports={entry:'./src/js/index.js',output:{filename:'js/built.js',path:resolve(__dirname,'build')},module:{rules:[//loader的配置{//处理less资源test:/\.less$/,use:['style-loader','css-loader','less-loader']},{//处理css资源test:/\.css$/,use:['style-loader','css-loader']},{//处理图片资源test:/\.(jpg|png|gif)$/,loader:'url-loader',options:{limit:8*1024,name:'[hash:10].[ext]',//关闭es6模块化esModule:false,outputPath:'imgs'}},{//处理html中img资源test:/\.html$/,loader:'html-loader'},{//处理其他资源exclude:/\.(html|js|css|less|jpg|png|gif)/,loader:'file-loader',options:{name:'[hash:10].[ext]',outputPath:'media'}}]},plugins:[//plugins的配置newHtmlWebpackPlugin({template:'./src/index.html'})],mode:'development',devServer:{contentBase:resolve(__dirname,'build'),compress:true,port:3000,open:true}};
```
[看课件]
(https://s8jl-my.sharepoint.com/personal/atguigu_s8jl_onmicrosoft_com/_layouts/15/onedrive.aspx?originalPath=aHR0cHM6Ly9zOGpsLW15LnNoYXJlcG9pbnQuY29tLzpmOi9nL3BlcnNvbmFsL2F0Z3VpZ3VfczhqbF9vbm1pY3Jvc29mdF9jb20vRW9zb2FhVlRmSXRCdnZHalhwTWhJdUlCbGFDYWtOR2F2Mm1COGliRE5zZmRZUT9ydGltZT1ybXh6REQtbTJFZw&id=%2Fpersonal%2Fatguigu%5Fs8jl%5Fonmicrosoft%5Fcom%2FDocuments%2Fwebpack%E8%B5%84%E6%96%99%2F%E8%AF%BE%E4%BB%B6%2F%E5%B0%9A%E7%A1%85%E8%B0%B7%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF%E4%B9%8Bwebpack%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A%28%E4%B8%8A%29%2Epdf&parent=%2Fpersonal%2Fatguigu%5Fs8jl%5Fonmicrosoft%5Fcom%2FDocuments%2Fwebpack%E8%B5%84%E6%96%99%2F%E8%AF%BE%E4%BB%B6)

### 11 构建环境介绍
```
生产环境区别
1）css -> js，因为js太大，加载慢可能闪屏，所以开发环境要分开
2）文件要压缩
3）兼容性很重要，一些高级版本需要增加前缀才能被浏览器识别
```

### 12 提取css成单独文件
```
单独的css需要插件plugin的帮助

const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [
					//创建style标签，将样式放入
					//'style-loader',
					//将这个loader取代style-loader。作用：提取js中的css成单独文件
					MiniCssExtractPlugin.loader,
					//将css文件整合到js文件中
					'css-loader'
				]
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		}),
		new MiniCssExtractPlugin({
			//对除数文件进行重命名
			filename: 'css/built.css'
		})
	],
	mode: 'development',
}

下载依赖
$ npm i minis-css-extract-plugin -D
```

### 13 css兼容性处理
```
这个特性非常好，工具插件自动把兼容性做好了，只要配置好插件就行，不过和maven一看要学习怎么配置

const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [
					MiniCssExtractPlugin.loader,
					'css-loader',
					/*css兼容性处理: postcss --> postcss-loader postcss-preset-env
					  帮postcss找到package.json中browserlist里面的配置，通过配置加载指定的css兼容性样式
					  "browserlist": {
						//开发环境 --> 设置nodejs环境变量process.env.NODE_ENV = "development"
						"development":[
							"last 1 chrome version",
							"last 1 firefox version",
							"last 1 safari version"
						],
						//生产环境: 默认就是看生产环境的配置
						"production": [
							">0.2%",
							"not dead",
							"not op_mini all"
						]
					  }
					
					*/
					//使用loader的默认配置
					//'postcss-loader',
					//修改loader的配置
					{
						loader: 'postcss-loader',
						options: {
							ident: 'postcss',
							plugin: () => [
								// postcss的插件
								require('postcss-preset-env')()
							]
						}
					}
				]
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		}),
		new MiniCssExtractPlugin({
			//对除数文件进行重命名
			filename: 'css/built.css'
		})
	],
	mode: 'development',
}

下载依赖
$ npm i postcss-loader postcss-preset-env -D
```

### 14 压缩css
```
css变小了，能让下载速度更快，让代码运行也更快。
需要使用插件optimize-css-assets-webpack-plugin

const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
module.exports = {
	...
	...
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		}),
		new MiniCssExtractPlugin({
			//对除数文件进行重命名
			filename: 'css/built.css'
		}),
		//压缩css
		new OptimizeCssAssetsWebpackPlugin()
	],
	mode: 'development',
}

下载依赖
$ npm i optimize-css-assets-webpack-plugin -D
```

### 15 js语法检查eslint
```
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			//规范团队的代码风格eslint-loader eslint
			//只检查自己写的源代码，第三方的库是不用检查的
			//设置检查规则：
			//在package.json中的eslintConfig中设置
			//推荐了aribnb的javascript代码规范
			//airbnb --> eslint-config-airbnb-base eslint eslint-plugin-import
			{
				test: /\.js$\,
				exclude:
				loader: 'eslint-loader',
				options: {}
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	mode: 'development',
}

具体的配置搜索npm中的eslint-config-airbnb-base插件使用指南即可
```

### 16 js兼容性处理eslint
```
可以将es6转换问es5以下的代码，
比如下面的es6的javascript代码
const add = (x, y) => {
	return x + y;
};

再比如promise
const promise = new Promise((resolve) => {
	setTimeout = (() => {
		console.log('定时器执行完了');
		resolve();
	}, 1000)
})

默认是不进行js兼容性处理了，还是会有箭头函数，在低版本browser比如IE就根本无法使用，需要配置兼容性处理。
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			/*
				js兼容性处理： 神器babel-loader #babel.core @babel.preset-env
				//2和3只能使用其中一种
				1.基本js兼容性处理 --> @babel/preset-env
					问题：直接转换基本语法，比如promise就不能转换
				2.全部js兼容性处理 --> @babel/polyfill 这个万能方案比较暴力
					问题：会下载所有兼容性代码，提及太大
				3.需要做兼容性处理的就做：按需加载 --> corejs
			*/
			{
				test: /\.js$/,
				loader: 'babel-loader',
				options: {
					//预设： 只是babel做怎么样的兼容性处理
					presets: [
						[
							'@babel/preset-env',
							{
								//按需加载
								useBuiltIns: 'usage',
								//指定core-js版本
								corejs: {
									version: 3
								},
								//指定兼容性做到那个版本浏览器
								targets {
									chrome: '60',
									firefox: '60',
									ie: '9',
									safari: '10',
									edge: '17'
								}
							}
						]
					]
				}
			}
		]
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	mode: 'development',
}

$ npm i @babel/polyfill core-js -D

引入即可polyfill即可，不过会引入一堆js兼容性包，让项目变得臃肿
import '@babel/polyfill';
const promise = new Promise((resolve) => {
	setTimeout = (() => {
		console.log('定时器执行完了');
		resolve();
	}, 1000)
})
```

### 17 压缩html和js
```
js代码的压缩是自动的：
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html'
		})
	],
	//生产环境会自动压缩js代码
	mode: 'production',
}

html的压缩处理：
html没有兼容处理，如果浏览器不识别html标签，那也没办法
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: '.src/index.html',
			minify: {
				//移除空格
				collapseWhitespace: true,
				//移除注释
				removeComments: true
			}
		})
	],
	mode: 'production',
}
```

### 18 生产环境基本配置
```
这个就是基本配置的模板了：

const { resolve } = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

//复用loader
const commonCssLoader = [
	MiniCssExtractPlugin.loader,
	'css-loader',
	{
		//还需要在package.json中定义browserslist
		loader: 'postcss-loader',
		options: {
			ident: 'postcss',
			plugins: () => [
				require('postcss-preset-env')()
			]
		}
	}
]

module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'built.js',
		path: resolve(__dirname, 'build')
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [...commonCssLoader]
			},
			{
				test: /\.less$/,
				use: [...commonCssLoader, 'less-loader']
			},
			/*
				正常来讲，一个文件只能被一个loader处理。
				当一个文件要被多个loader处理，那么一定要制定loader执行的先后顺序：
				先执行eslint，再执行babel
			*/
			{
				//在package.json中的eslintConfig --> airbnb
				test: /\.js$/,
				exclude: /node_modules/,
				//加上enforce表示优先执行，
				enforce: 'pre',
				loader: 'eslint-loader',
				options: {
					fix: true
				}
			},
			{
				test: /\.js$/,
				exclude: /node_modules/,
				loader: 'babel-loader',
				options: {
					presets: [
						[
							'@babel/preset-env',
							{
								useBuiltIns: 'usage',
								corejs: {
									version: 3
								},
								targets {
									chrome: '60',
									firefox: '60',
									ie: '9',
									safari: '10',
									edge: '17'
								}
							}
						]
					]
				}
			},
			{
				test: /\.(jpg|png|gif)$/,
				loader: 'url-loader',
				options: {
					limit: 8 * 1024,
					name: '[hash:10].[ext]',
					outputPath: 'imgs',
					esModule: false
				}
			},
			{
				test: /\.html$/,
				laoder: 'html-loader'
			},
			{
				exclude: /\.(js|css|less|html|jpg|png|gif)/,
				loader: 'file-loader',
				options: {
					outputPath: 'media'
				}
			}
		]
	},
	plugins: [
		 new MiniCssExtractPlugin({
			filename: 'css/built.css'
		 }),
		 new OptimizeCssAssetsWebpackPlugin()
		new HtmlWebpackPlugin({
			template: '.src/index.html',
			minify: {
				collapseWhitespace: true,
				removeComments: true
			}
		}),
	],
	mode: 'production',
}
```

剩下的原理层面的就不用看了，到这里基本对webpack的使用和应用场景有了解了。