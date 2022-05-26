---
title: vue3 插件系统 
---

## vue3 插件系统

一段代码给 vue 应用实例添加全局功能

它的格式是 一个 **object** ，暴露出一个 **install**() 方法，或者是一个 **function**

它没有严格的限制，一般有以下几种功能

- 添加全局方法或属性
- 添加全局资源： 指令，过滤器等
- 通过全局呼入一些来添加一些组件选项
- 通过config.globalProperties 来添加app 实例方法

## **编写一个 vue3 插件**

- 首先这个插件拥有属于自己的模板也就是我们的 .vue 文件

```html
<template>
	<div class="HelloWrold"> HelloWrold </div>
</template>

<script>
	import { defineComponent } from 'vue'
	export default defineComponent({
		name: "HelloWrold"
		props: {
			...
		}
	})
</script>
```

- 然后我们需要暴露一个 install 方法出去，在 install 方法里面处理所有的逻辑，这样 vue 就可以 use 这个插件了

```jsx
import HelloWrold from 'HelloWrold.vue'

/*
有了模板之后，
- 我们就需要创建应用 app
- 创建一个元素 div 追加于 body 后面
- 然后使用应用 app 来挂载这个 div 元素
*/ 
const createComp = function(comp, prop) {
	// 创建组件
	const app = createApp(comp, {
		...prop
	})
	
	// 创建一个元素
	const divEle = document.createElement('div')
	document.body.appendChild(divEle)
	// 将组件挂载到创建的 div 中
	app.mount(divEle)
	// 返回创建的元素
	return divEle
}

export default {
	install(app, options) {

		// 创建全局指令
		app.directive('focus', {
      mounted (el, binding, vnode, oldVnode) {
        	// 动态挂载组件
					createComp(HelloWrold, {...})
					// 聚焦元素
					el.focus()
      }
      ...
    })

		// 添加全局实例方法，通过把它们添加到 config.globalProperties 上实现。
		app.config.globalProperties.$echo = (str) => {
			conosle.log(str)
		}

		// 全局注册组件 (不需要调用组件内注册 ，直接使用 )
		app.component('hello-wrold', HelloWrold)

		// 使用 inject 为插件的用户提供功能或 attribute
		// 插件用户现在可以将 inject[test] 注入到他们的组件并访问该对象
		app.provide('test', { msg: 'from plugin'}) 
		
		//使用 mixin
		app.mixin({
      created() {
        console.log('hello world') // 每个组件都会执行
      }
      ...
    })

	}
	
}
```

## vue3 的插件的使用

```jsx
import { createApp } from 'vue'
import App from 'App.vue'
import myPlugin from 'myPlugin.js'

const app = createApp(App)
app.use(myPlugin)
app.mount('#app')

```

```html

<template>
	<!--调用全局组件-->
	<hello-wrold></hello-wrold>
	<!--调用全局指令-->
	<input v-focus />
</template>

<script>
import { defineComponent, onMounted, inject, getCurrentInstance } 'vue'
export default defineComponent({
		name: "App"
		setup() {
			onMounted(() => {
				// 调用全局 provide 的数据
				console.log(inject('test'))
				// 调用全局方法
				getCurrentInstance()?.appContext.config.globalProperties.$echo('hellow')
			})
		}
})
</script>
```

## 组件库入口文件的设计

### 单个组件导入并且作为插件使用

```jsx
import { LText } from 'myComponets'
app.use(LText)
// 或者
app.component(LText.name, LText)

// 实现思路
/*
	每个组件新建一个文件夹，并且创建一个单独的 index.js 文件
	每个组价设计成一个插件（一个 object 拥有一个 install 方法）	
	在全局入口导入
*/

// src/components/LText/index.js
import { App } from 'vue'
import LText from './LText.vue'
LText.install = (app) => {
	app.component(LText.name, LText)
}
export default LText

```

### 所有组件一次性全部导入并且作为插件使用

```jsx
// 实现思路
/*
	建立一个入口文件 index.js
	将所有组件导入，作为一个数组， 常见一个 install 函数，循环调用 app.component
	默认导出一个 插件 （ 这个install 函数 ）
*/

// src/index.js
import { App } from 'vue'
import LText from './components/LText.vue'
...

const components = [ 
	LText
	...
]
const install = (app) => {
	components.forEach(componet => {
		app.component(componet.name, componet)
	})
	
}
export {
	LText,
	...
	install
}
export default { 
	install
}

// 组件库调用
import myComponents from 'myComponets'
app.use(myComponents)

```

## 组件单元测试

```html
<template>
	<h1>{{ msg }}</h1>
	<button @click="setCount">{{ count }}</button>
</template>
<script>
	import { defineComponent, ref } from 'vue'
	export default defineComponent({
		name: 'HelloWorld',
		props: {
			msg: String
		},
		emits: ['send']
		setup(props, context) {
			const count = ref(1)
			const setCount = () => {
				count.value++
			}
			return {
				count,
				setCount
			}
		}
	})
</script>
```

```jsx
// 添加单元测试依赖 (在项目根目录中，添加@vue/test-utils and jest)
	vue add unit-jest

// jest.config.js
/*
	比较重要的配置，也是我们比较多用来解决问题的配置：
	
  transform : 转化方式，匹配的文件要经过转译才能被识别，否则会报错。
	transformIgnorePatterns : 转化忽略配置
	moduleNameMapper : 模块别名，如果有用到都要填写进去
	moduleFileExtensions ： 测试的文件类型，这里默认的配置基本涵盖了文件类型，所以这里一般不需要改
*/

module.exports = {
  preset: '@vue/cli-plugin-unit-jest/presets/typescript-and-babel',
	transform: { // 转化方式
	    // process *.vue files with vue-jest
	    '^.+\\.vue$': require.resolve('vue-jest'),
	}
}

// 常用API [文档](https://test-utils.vuejs.org/guide/)
/*
	mount : 一股脑全都渲染
	shallowMount : 只渲染组件本身，外来的子组件都不渲染 （更快，适合单元测试）
	get : 找不到报错， 程序会中断
	getAll 
	find : 找不到返回 null ，可用来判断元素是否存在
	findAll
	findComponent: 查找组件 没有找到 返回null
	getComponent: 查找组件 找不到报错，程序会中断
	trigger: 触发事件 
	setValue : 更新表单
	wrapper.emitted(): 验证事件是否发送
	
	钩子函数：
		一次性完成的测试准备
			beforeAll
			afterAll
		每个测试测试准备
			beforeEach
			afterEach
		
	小tips：
		it.only // 这运行这一个测试用例
		it.skip // 跳过该测试用例
*/

// 测试代码
import { shallowMount } from '@vue/test-utils'
import LText from '../../src/components/LText'
describe('LText.vue', () => {
	it('default LText render', () => {
		const msg = 'test'
		const props = { text: msg }
		const wrapper = shallowMount(LText, { props })

	// should have the text
	expect(wrapper.text()).toBe(msg)
	// should be default div tag
	expect(wrapper.element.tagName).toBe('DIV')
	// should have certian css attr
	const style = wrapper.attributes().style
	expect(style.includes('font-size')).toBeTruthy()
	})
	// 异步测试
	it('shouldupdate the count when clicking the button', async() => {
		const msg = 'new msg'
		const wrapper = shalloMount(HellowWrold, {
			props: { msg }
		})		
		await wrapper.get('button').trigger('click')
		expect(wrapper.get('button').text()).toBe('2')
	})
})
```

## 组件库打包

### package.json

```jsx
// package.json
{ 
	"name": "my-components",
	"version": "1.0.0", //  <主版本号>.<次版本号>.<修订号>
	"main": 'dist/my-components/umd.js', // 入口文件
	"module": "dist/my-components.esm.js", // node 入口文件
  // "types": "dist/index.d.ts", // ts 入口文件
	"scripts": {
		"lint": "vue-cli-service lint",
		"test": "vue-cli-service test:unit",
		"build": "npm run clean && npm run build:esm",
		"build:esm": "rollup --config build/rollup.config.js",
		"clean": "rimraf ./dist"
		"prepublishOnly": "npm run lint && npm run test && npm run build" 
	},
	"dependencies": {},
	"peerDependencies": {
		"vue": "^3.0.0" // 告知该组件库依赖 vue3
	},
	"devDependencies": {
		"rimraf": "3.0.2",
		"vue": "^3.0.0", // 开发环境使用
		"rollup": "^2.38.5",
		"rollup-plugin-css-only": "^3.1.0",
		"rollup-plugin-vue": "^6.0.0"
	}
}

// npm 依赖的分类
/*
	dependencies
		运行项目业务逻辑需要依赖的第三方库
		npm install '模块名称'的时候都会被机械，下载
	
	devDependencies
		开发模式工作流下依赖的第三方库
		单元测试，语法转换，lint工具，程序构建，本地开发等等
	
	peerDependencies
		需要核心依赖库，不能脱离依赖库单独使用
*/

```

### Rollup打包

```jsx

// build/rollup.config.js
import { terser } from 'rollup-blugin-terser' // 压缩混淆代码
import { nodeReslove } '@rollup/plugin-node-resove' // 帮助 Rollup 查找外部模块
import vue from 'roolup-plugin-vue'
import css from 'roolup-plugin-css-only'
import { name } from '../package.json'
const file = type => `dist/${name}.${type}.js`
export default {
	input: 'src/App.vue',
	output: {
		name, 
		file: file('esm')
		format: 'es'
		globals: { // 全局变量的名字
			'vue': 'Vue'
		}
	},
	plugins: [
		terser(),
		nodeReslove(),
		vue(),
		css({output: 'bundle.css'})
	],
	external: ['vue']// 哪些是外部引用，打包时将其省略掉
	// external: (id) => { return /^vue/.test(id) }
}

/* @rollup/plugin-node-resolve 的作用
 比较：
	 不配置 @rollup/plugin-node-resolve 插件引入方式
			export foo from ‘./foo/index.js‘
			import bar from ‘./bar/index.js‘
	 配置了 @rollup/plugin-node-resolve 插件引入方式
			export foo from ‘./foo‘
			import bar from ‘./bar‘
	 可以看出不配置的话引入路径必须是完整的。
*/

```

## 组件库通过 npm 包发布

```yaml
# 检测是否登录 
	npm whoami

# npm 登录
	npm login

# 查看配置信息是否使用了镜像代理 （registry = "http://registry.npm.taobao.org"）
# 有代理镜像 无法登录
	npm config ls

# 发布前填写 package.json 信息
	# <主版本号>.<次版本号>.<修订号>
	# 主版本号： 当你做了不兼容的 API 修改
	# 此版本号： 当你做了向下加绒的功能性新增
	# 修订号： 当前做了向下兼容的问题修正
		"version": "1.0.0", 
	# 相关信息
	  "keyworks": [ "Component", "Vue3"]
	# 默认忽略掉 gitingore 中的内容
	# 指示 npm publish 的时候需要上传的内容
	# package.json/ README.md/ CHANGLOG.md/ LiCENSE 都包含其中
		"file": [ "dist" ]
	# scripts 钩子
		"scripts": {
			"precompress": "", # compress 之前执行
			"compress": "",
			"postcompress": "" # compress 之后执行
			"prepublishOnly": "" # 在发布之前运行
		}
# npm 发布
	npm publish
```
 