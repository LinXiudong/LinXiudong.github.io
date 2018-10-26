---
layout: mypost
title: 对象/数组的深度监听(deep watch)和首次执行(immediate)
categories: [vue]
---

若用普通监听写法，当对象内部的值改变时(比如a.b=1)，watch无法捕获到对象改变。需要下面写法：
```javascript
new Vue({
	el: '#app',
	data: {
		a: {b:1}
	},
	methods: {
	    //点击时运行
		aa: function(){
			if(this.a.b == 1){
				this.a.b = 2;
			}else{
				this.a.b = 1
			}
		}
	},
	watch: {
	    //普通写法，永远不会执行
		/*a: function(newValue, oldValue){
			console.log(newValue)
			console.log(oldValue)
		}*/
		//深度监听
		a:{
			handler: function(newValue, oldValue) {
				console.log(newValue)
				console.log(oldValue)
			},
			//一般初始化页面时，不会执行watch，只有a改变后才会运行
			//设置immediate为true后，初始化时上面的代码就会运行一次
			//即加载时就会运行上面的函数
			immediate: true,
			deep: true  //开启深度监听
		}
		//上面的方法有个缺点，资源开销大，a里每个对象改变都会运行
		//如果只是监听a中的某个属性，可以直接监听
		/* 'a.b': function(newValue, oldValue){
			console.log(newValue)
			console.log(oldValue)
		}*/
	}
})
```
*数组、数组对象、对象数组等情况类似。不过数组可以用splice，好像没必要。*
