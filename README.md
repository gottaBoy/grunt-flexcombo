# grunt-flexcombo

![](http://gtms01.alicdn.com/tps/i1/T1TgNqFB0bXXbX25fJ-346-77.png)

[flex-combo](https://npmjs.org/package/flex-combo)的grunt插件版本，[Grunt-Flexcombo 原理](https://speakerdeck.com/lijing00333/grunt-flexcombo)。

## Getting Started

依赖 Grunt 版本`~0.4.1`

安装grunt-flexcombo

```shell
npm install grunt-flexcombo --save-dev
```

安装后，在Gruntfile.js中载入任务

```js
grunt.loadNpmTasks('grunt-flexcombo');
```

## "flexcombo" 任务配置

在`grunt.initConfig()`中添加flexcombo的配置：

```js
grunt.initConfig({
	flexcombo:{
		debug:{
			options: {
				target:'src',
				urls:'/group/project',// 匹配[ag].tbcdn.cn时的访问路径
				port:'80',// flexcombo服务端口
				proxyport:'8080',// 反向代理服务端口
				servlet:'?',//开启combo的字符串
				separator:',',//多文件分隔符
				charset:'utf8',//默认输出文件的编码
				proxyHosts:['a.com','b.com'],//本地服务的域名
				// 过滤参数
				filter:{ 
					'-min\\.js':'.js',
					// 当访问a.com/go/act/demo.php时，实际请求
					// a.com/demo.html
					'/go/act/\(.+\\.\)php':'/$1html'
				}
			}
		}
	},
	// flexcombo 需要配合 watch 模块一起使用
	watch: {
		'all': {
			files: ['src/**/*'],
			tasks: [ 'your-build-target' ]
		}
	}
});

// 注册 debug 任务
grunt.registerTask('debug', '开启debug模式', function() {
	task.run(['flexcombo:debug','watch:all']);
});
```

<strong>任务说明：</strong>

`flexcombo`任务无法单独运行，需要配合`watch`启动。一份完整的[Gruntfile.js](https://github.com/jayli/slide/blob/master/Gruntfile.js)样例。

<strong>启动服务：</strong>

	sudo grunt debug

配置参数说明：

#### options.target

- 类型：String
- 默认值：''

基于此目录启用本地服务，类似Apache中的`DocumentRoot`，需写相对目录，相对于`Gruntfile.js`所在的目录。

#### options.urls

- 类型：String
- 默认值：''

启用本地服务后，访问到`options.target`目录对应的url里的路径，比如`options`的配置：

	options:{
		target:'src'
		urls:'/a/b/c'
	}

访问URL`http://localhost/a/b/c/demo.html`，将返回`./src/demo.html`。此配置主要用作模拟cdn的真实路径，比如直接访问`http://g.tbcdn.cn/a/b/c/index.js`，将访问`./src/index.js`。

#### options.port

- 类型：String
- 默认值：'80'

flexcombo 服务的端口，默认启动在80端口，启动在80端口可以让你直接绑定Host到本机，若启动在其他端口，则只能用代理工具来辅助访问。

#### options.proxyport

- 类型：String
- 默认值：'8080'

反向代理服务器的端口。

#### options.servlet

- 类型：String
- 默认：'?'

开启combo的前缀。不同的开发环境有不同的combo需求。通过`servlet`,`seperator`两个参数决定。

#### options.separator

- 类型：String
- 默认值：','

Combo的url中多个文件的分隔符。

#### options.charset

- 类型：String
- 默认：'utf8'

默认输出文件的编码，主要用于约束输出assets文件的编码。

#### options.proxyHosts

- 类型：String 或者 Array
- 默认：'' 或者 []

本地服务自定义虚机的域名。除了`a.tbcdn.cn`和`g.tbcdn.cn`这两个默认域名，可以新增自定义域名配置，多个域名用数组表示。

> 自定义域名无须配Host访问，只需配端口为`proxyport`的代理服务即可。

注意：来自`proxyHosts`中配置的域名的url请求路径不会匹配`options.urls`，比如`proxyHosts:'a.com'`，那么访问

	http://a.com/demo.html

将访问本地文件`./src/demo.html`

#### options.filter

- 类型：Object
- 默认值：`{}`

`filter`可以用来过滤传入url。`filter`对象，其中`key`是匹配的正则表达式，`value`是替换的字符串，支持正则表达式变量。替换的顺序与定义无关。`filter`类似于Apache中的rewrite规则

#### options.urlBasedCharset

- 类型：Object
- 默认值：`{}`

`urlBasedCharset`可针对某一个url设置响应字符集。例如：

    "charset" : "utf-8",
	"urlBasedCharset" : {"/apps/aaa.js":"gbk"}

允许在大多数情况下返回字符集为utf-8字符集的资源。但在访问/apps/aaa.js的情况下，以gbk的方式编码。 这个特性多被用来引入编码几不同的第三方脚本。

options的更多配置请参照[flex-combo](https://npmjs.org/package/flex-combo)。

### 服务启动

flexcombo将在本地启动两个服务

![](http://gtms01.alicdn.com/tps/i1/T1.ey8FnleXXcxFyEb-523-342.png)

启动服务后，可以直接通过`localhost`来预览本地文件

	http://localhost/group/project

若要模拟真实环境调试，两种方法（二者取其一），推荐第二种：

1. 将cdn配向开发机`127.0.0.1 g.tbcdn.cn a.tbcdn.cn`
1. 将浏览器或者设备HTTP代理配置到本机的反向代理服务的端口

比如在手机终端设置代理方法：

![](http://gtms01.alicdn.com/tps/i1/T1bePRFlVXXXXhb4nD-502-341.png)

### SSI

该服务添加了[jayli-server](https://npmjs.org/package/jayli-server)，支持标准格式的 SSI include

	<!--#include path="../src/file-path.html" -->

## 使用案例1

[KISSY 组件如何调试线上代码](http://blog.kissyui.com/2013/11/29/%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E5%9C%A8%E7%BA%BF%E8%B0%83%E5%BC%8F%E4%BD%A0%E7%9A%84gallery%E7%BB%84%E4%BB%B6/)

## 使用案例2

KISSY项目构建工具和本地环境：[generator-clam](https://npmjs.org/package/generator-clam)一同使用，构建工具生成本地配置文件，形如：

	flexcombo:{
		debug:{
			options:{
				proxyHosts:'trip.taobao.com',
				target:'build/',
				proxyport:8080,
				urls:'/trip/proj/0.0.2',
				port:'80',
				servlet:'?',
				separator:',',
				charset:'utf8',
				filter:{
					'-min\\.js':'.js',
					'/go/act/\(.+\\.\)php':'/$1html',
				}
			}
		}
	}

