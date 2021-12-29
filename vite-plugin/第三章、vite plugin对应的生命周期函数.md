> 本章介绍声vite插件对应的声明周期函数，这样有助于我们在开发vite插件时候，找到对应的声明周期函数，做出相应的处理方法，生命周期函数对于做过vue、react开发的同学来讲并不是一个陌生的概念；

<a name="Du1xb"></a>
### 1、通用的声明周期钩子函数
开发时候 ，vite dev server启动创建一个插件容器，会按照rollup调用创建钩子的规则请求各个钩子函数，下面钩子函数会在服务器启动时候调用一次

- options 替换或操作 rollup选项（如果不是打包处理，一般用不到）
- buildStart 准备创建的信号

下面的钩子每次有模块请求都会被调用，每次import都会触发这些钩子函数

- resolveId 创建定义确认函数
- load 床架插件函数，用于返回自定义内容
- transform 用于转换已加载的模块

​<br />
<a name="ZPEwa"></a>
### 2、vite特有的钩子

- config 修改vite的配置，例如配置别名，可以覆盖之前的配置
```typescript
config(conf) {
  // return的对象会和 conf合并生成一个新的配置
	return {
  	// 设置一些配置项内容
  }
}
```

- configResolved vite配置确认
- configureServer 用于配置开发服务器（也就是我们通常yarn vite启动的服务器）
```typescript
configureServer(server) {
  // koa2的中间件模式类似的方式来去做，好像不完全用的是koa2这套东西（后续计划做个mock请求拦截工具）
  // 之前在做mock在网上有一个类似的插件库，可以按照拦截的模式做mock数据
	server.app.use(req, res, next) ... 
}
```

- transformIndexHtml 用于转换宿主页面
```typescript
transformIndexHtml(html: string): Promise<IndexHtmlTransformResult> {
	return {
		html: '' as string,
  	tags: [{
			tag: '', // div script ... html元素
  		attrs: {
				src: '' // 元素的的属性、属性值
			},
  		children?: tags[] // 此处为距离，大概意思就是tags类型的数据做为子元素
			injectTo: 'head-prepend' // 'head' | 'body' | 'head-prepend' | 'body-prepend' 
		}]
	}
}
```

- handleHotUpdate 用户hmr热更新调用

​<br />
<a name="q2GbG"></a>
### 3、插件生命周期调用的顺序
在做顺序之前，这里我编写一个一个测试生命周期函数调用的基本顺序测试plugin，大家可以去吧这个方式vite项目中，启动项目实际感受一下顺序，纸上得来终觉浅，亲自实操一番记忆会更深刻
```typescript
function myExamplePlugin(): Plugin {
  return {
    name: 'my-example-plugin',
    // 只执行一次
    options(opt) {
      console.log('options', opt)
      return opt
    },
    buildStart() {
      console.log('start')
    },
    // vite特有的钩子
    config(conf) {
      console.log('config', conf)
    },
    configResolved(id) {
      console.log('configResolved', id)
    },
    configureServer(server) {
      console.log('configureServer', 'server拦截，做mock')
    },
    transformIndexHtml(html) {
      console.log('transformIndexHtml')
      return html
    },
    // 每次插件执行后都会去调用的钩子
    resolveId(source) {
      console.log('resolveId')
      return source
    },
    // 加载模块代码
    load(id) {
      console.log('load', id)
      return id
    },
    transform(code, id) {
      console.log('transform')
      return code
    }
  }
}
```
这块我们需要区分一下运行时环境，和打包过程的环境，两者有一些区别<br />

<a name="Qr5my"></a>
### 4、插件执行的顺序
在vite插件开发的过程中，我们一般都会给插件配置enfore 插件属性，来去确定我们的产检具体在哪个位置执行；下面流程图展示一下具体插件的执行顺序：
