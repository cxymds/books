# js异步操作

## 一、异步的概念

同步：A->B->C->D->E->F

异步：最省时算法

A-->B--->D----->E----->F

​	->C--->D      

典型的异步操作

- 网页资源下载

- js获取服务器数据

## 二、多线程

js是单线程的，二事件回调，定时器，以及ajax，promise都是依赖于宿主环境的多线程实现的。即浏览器的多线程辅助js线程的运行。

### 2.1 浏览器的线程

+ GUI渲染线程
+ js引擎线程
+ 定时器触发线程
+ 浏览器事件线程
+ http异步线程
+ EventLoop事件轮训线程
+ ....

> 需注意： GUI渲染线程与js引擎线程是互斥的，也就是说GUI引擎在渲染的时候会阻塞js引擎的计算。

###2.2 进程的概念

一般来说一个运行的应用程序就是一个进程，一个进程可以是很多线程的组合。

QQ进程：接受消息的线程 发送消息的线程 传输文件的线程 检测安全的线程 

所以一个网页在运行过程中也需要很多线程的配合。上面已经提到过浏览器的线程。

###2.3 线程的概念

进程可以拆分为线程，线程是计算机任务调度的最小单位。

## 三、异步演示

### 3.1 加载图片

```js
function loadImage(src){
	let image = new Image();
	image.src=src;
}
loadImage('./image/01.png');
```

添加监听处理

```js
function loadImage(src,resolve,reject){
	let image = new Image();
	image.src=src;
  image.onload = resolve;
  image.onerr = reject;
}
loadImage(
  './image/01.png',
	()=>{
    console.log('图片加载成功')
  },
	()=>{
    console.log('图片加载失败')
  }
);
```

添加结果处理

```js
function loadImage(src,resolve,reject){
	let image = new Image();
	image.src=src;
  image.onload = ()=>{
  	resolve(image)
  };
  image.onerr = reject;
}
loadImage(
  './image/01.png',
	(image)=>{
		//此处可以处图片
    //document.appendChild(image)
    console.log('图片加载成功')
  },
	()=>{
    console.log('图片加载失败')
  }
);
```

### 3.2 定时器

封装定时器

```js
function interval(callback,delay){
	let id = setInterval(()=>{
		callback(id);
	},delay)
}
```

简化一下：

```js
function interval(callback,delay){
	let id = setInterval(()=>callback(id),delay)
}
interval(timeId=>{
  const div = document.querySelector('div');
  let left = parseInt(window.getComputedStyle(div).left);
  div.style.left = left + 10 + 'px'
  if(left>=200){
    clearInterval(timeId);
  }
})
```

###3.3 jsonp加载文件

```js
function load(src){
	let script = document.createElement('script');
	script.src = sec;
	document.body.appendChild(script);
}
```

添加处理

```js
function load(src,resolve){
	let script = document.createElement('script');
	script.src = sec;
	script.onload = resolve;
	document.body.appendChild(script);
}

load('js/data.js',()=>{
  func();
})

console.log(‘主进程优先’)
```

任务嵌套

```js
load('js/a.js'()=>{
	a();
	load('js/b.js',()=>{
		b();
	})
})
```

###3.4 ajax异步请求数据

```js
function ajax(url,callback){
	let xhr = new XMLHttpRequest();
	xhr.open("GET",url);
	xhr.send();
	xhr.onload = function(){
		if(this.status==200){
			callback(JSON.parse(this.response))
		}else{
		throw new Error('加载失败')
		}
	}
}
```

获取用户id之后在获取用户成绩

### 3.5 promise

```js
new Promise(resolve,reject){
	if(成功){
		resolve();
	}
	if(失败){
		reject();
	}
}
```

promise的结果处理

```js
new Promise((resolve,reject)=>{
	if(成功){
		resolve();
	}
	if(失败){
		reject();
	}
}).then(value=>{
  console.log('业务处理成功')
},reson=>{
  console.log('拒绝的业务处理')
})
```

## 四、主任务 宏任务 微任务

promise ,ajax会产生微任务队列。

主任务与宏任务

```js
setTimeout(()=>{
	console.log('宏任务')
}，0)；
console.log('主任务')
```

主任务

```js
new Promise(resolve=>{
	//同步代码
	console.log('promise') //1
})

console.log('主任务')		//2
```

微任务优先级小于主任务

```js
new Promise(resolve=>{
	resolve();
  //先执行
	console.log('主任务同步代码')
}).then(value=>{
	console.log('微任务代码')
})

console.log('主任务代码')
```

微任务优先级大于宏任务

```js
setTimeout(()=>{
  console.log('宏任务代码最后执行')//4
},0)；

new Promise(resolve=>{
	resolve();
  //先执行
	console.log('主任务同步代码')//1
}).then(value=>{
	console.log('微任务代码')//3
})

console.log('主任务代码')//2
```

宏任务中创建微任务

```js
new Promise(resolve=>{
	setTimeout(()=>{
    console.log('宏任务')//3
    resolve();
  },0)
  //先执行
	console.log('主任务同步代码') //1
}).then(value=>{
	console.log('微任务代码')//4
})

console.log('主任务代码')//2
```

promise状态是单向的不可逆的。reject或者resolve之后状态不可以在发生变化。

## 五、promise


