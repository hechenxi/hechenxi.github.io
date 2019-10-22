---
layout:     post
title:      手写promise和双向数据绑定
subtitle:   简单版
date:       2019-10-22
author:     HCX
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - JavaScript
---

## 手写Promise
> 先修知识：原生Promise的基本使用
### 1. 基本框架:
#### 需求1: 传入一个函数executor，函数里面有两个（函数）参数resolve，reject，并且executor立即执行，
#### 需求2：初始状态为 pending，当调用resolve后，状态变成resolved，调用reject后，状态变为rejected
```js
function MyPromise(executor){
	this.status = 'pending'
	let self = this
	function resolve(value){self.status = 'resolved'}
	function reject(reason){self.status = 'rejected'}				
	executor(resolve,reject)
}
```
#### 需求3: MyPromise有一个原型方法then，传递两个（函数）参数，分别是成功、失败后的回调函数
```js
MyPromise.prototype.then = function(fillfuledCallback,rejectedCallback){}
```
### 2. 同步情况
#### 原生Promise的效果
```js
const p = new Promise((resolve,reject)=>{
	console.log(1)
	resolve(2)
})
p.then(value=>{console.log(value)},reason=>{console.log(reason)})
//输出 1 2
```
#### 我们的实现
MyPromise函数执行完后**status肯定变成resolved或者rejected**

```js
function MyPromise(executor){
	this.status = 'pending'
	this.value = undefined
	this.reason = undefined
	let self = this
	function resolve(value){
		self.status = 'resolved'
		self.value = value //存一下，好在then里面使用
	}
	function reject(reason){
		self.status = 'rejected'
		self.reason = reason //存一下，好在then里面使用
	}				
	executor(resolve,reject)
}

MyPromise.prototype.then = function(fillfuledCallback,rejectedCallback){
	if (this.status === 'resolved') fillfuledCallback(this.value);
	else if (this.status === 'rejected') rejectedCallback(this.reason)
}
```

### 3. 异步情况
### 4. reslve和reject都是异步最后执行的
### 5. 完整代码

```js
function MyPromise(executor){
	this.status = 'pending' //
	/*下面两个参数都是为同步状况做准备的*/
	this.value = undefined; //成功传递的参数
	this.reason = undefined;//失败的原因
	
	this.fillfuledCallback = function(){};
	this.rejectedCallback = function(){};
	
	let self = this
	function resolve(value){
		setTimeout(function(){ //这里加setTimeout,是为了让resolve永远异步最后执行
			self.status = 'resolved'
			self.fillfuledCallback(value) //如果是同步情况,这里执行的是一个空函数;如果是异步情况,这个就在then里面被赋值了
			self.value = value //为同步情况做准备
		},0)
		// self.status = 'resolved'
		// self.fillfuledCallback(value) //如果是同步情况,这里执行的是一个空函数;如果是异步情况,这个就在then里面被赋值了
		// self.value = value //为同步情况做准备
	}
	function reject(reason){
		setTimeout(function(){
			self.status = 'rejected'
			self.rejectedCallback(reason) //如果是同步情况,这里执行的是一个空函数
			self.reason = reason //为同步情况做准备
		},0)
	}
	
	executor(resolve,reject)
}
MyPromise.prototype.then = function(fillfuledCallback,rejectedCallback){
		//同步情况
		if (this.status === 'resolved') fillfuledCallback(this.value);
		else if (this.status === 'rejected') rejectedCallback(this.reason)
		//异步情况
		else if (this.status === 'pending'){
			this.fillfuledCallback = fillfuledCallback || function(){};//防止有人不传函数
			this.rejectedCallback = rejectedCallback || function(){};
		}
}
```

## 手写Vue双向数据绑定
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>双向数据绑定</title>
	</head>
	<body>
		<div id="app">
			<input type="text" id="input" v-model="text" /><br />
			<!-- <input type="text" id="input"  /><br /> -->
			{{ text    }}
		</div>
		<script>
			//2.0版本
			
			//第一步初始化绑定
			//1.编译器:v-model去掉,data.text赋值给{{text}}
			function compile(node,vm){//vm是MyVue实例对象
				let reg = /\{\{.*\}\}/ //.是单个字符,*是量词0个或多个
				if (node.nodeType === 1){ //ELEMENT元素
					let attr = node.attributes
					for (let i=0;i<attr.length;i++){//每一个attr都是ATTRIBUTE_NODE对象
						var attrName = attr[i].nodeName
						var attrValue = attr[i].nodeValue
						if (attrName === 'v-model'){
							node.addEventListener('input',function(e){
								//console.log(node)
								vm[attrValue] = e.target.value
							})
							// node.value = vm[attrValue] //这句话多余了
							node.removeAttribute('v-model')
							new Watcher(vm,node,attrValue,'input')  //这里的attrValue就是text!
						}
					}
				} else if (node.nodeType === 3) { //TEXT元素
					if (reg.test(node.nodeValue)){
						let name = reg.exec(node.nodeValue)[0]
						name = name.slice(2,name.length-2).trim() //去掉{{}}和空格
						// node.nodeValue = node.nodeValue.replace(reg,vm[name]) 由watcher就可以赋值了,这句多余了
						new Watcher(vm,node,name,'text')
					}
				}
			}
			
			//先修知识,创建dom的方式 createElement,innerHTML,createDocumentFragment(p275)
			function node2Fragment(node,vm){
				let fragment = document.createDocumentFragment();
				let child;
				while(child = node.firstChild){
					compile(child,vm)
					fragment.appendChild(child) //原node里的元素添加进fragment后,原node里就没了!
				}
				return fragment
			}
			
			//响应式数据绑定,把key变成vm的访问器属性
			function defineReactive(vm,key,val){
				let dep = new Dep()
				Object.defineProperty(vm,key,{
					get: function(){
						if (Dep.target) dep.addSub(Dep.target)
						return val
					},
					set: function(newVal){
						if (newVal === val) return
						val = newVal
						console.log(val)
						console.log(dep)
						dep.notify()
					}
				})
			}
			//把vm.data的每个属性都变成vm的访问器属性!!!
			function observe(vmData,vm){
				for (let key in vmData){
					defineReactive(vm,key,vmData[key])
				}
			}
			
			//第三步,（主题的）发布订阅模式
			//主题
			function Dep(){
				this.subs = []//主题的订阅者
			}
			Dep.prototype = {
				addSub: function(sub){
					this.subs.push(sub)
				},
				notify:function(){ //主题发布
					this.subs.forEach(sub=>{ //主题的订阅者都更新一下
						sub.update() //
					})
				}
			}
			//订阅者
			function Watcher(vm,node,name,nodeType){
				Dep.target = this
				this.name = name
				this.node = node
				this.vm = vm
				this.nodeType = nodeType
				this.update()
				Dep.target = null
			}
			Watcher.prototype = {
				update: function(){ //订阅者更新method
					this.get()
					if (this.nodeType == 'text'){
						this.node.nodeValue = this.value
					} else if (this.nodeType == 'input'){
						this.node.value = this.value
					}
				},
				get: function(){
					this.value = this.vm[this.name]
				}
			}
			
			function MyVue(options){
				this.data = options.data
				observe(this.data,this)
				let id = options.el;
				let vdom = node2Fragment(document.getElementById(id),this)
				document.getElementById(id).appendChild(vdom)
			}
			
			const vm = new MyVue({
				el: 'app',
				data: {
					text: 'hello--world!'
				}
			})
			
		</script>
	</body>
</html>
```