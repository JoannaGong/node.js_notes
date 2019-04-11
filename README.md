## 模块

### 使用 node.js 执行 js 代码

* 用 node 执行单独的 js 代码
在终端中，输入 node 命令，进入 node 代码执行与编辑模式，会显示一个向右箭头和输入光标；此时只能一行行的执行 js 代码，而不能执行一个 js 文件。
![node run](./images/node-run1.png);

* 用 node 执行整个 js 文件
直接在终端中输入：node js文件名

### export 命令
模块功能主要由两个命令构成：export 和 import。export 命令用于规定模块的对外接口，import 命令用于输入其它模块提供的功能。

* 输出变量
```
// 第一种写法
export var firstName = "Jackson";

// 第二种写法，优先使用这种方法
var firstName = "Jackson";
var age = 18;
export { firstName, age };
```

* 输出函数
```
export function demo(x, y){
    return x*y;
};
```

* 通常情况下，export 输出的变量就是本来的名字，但是可以使用 as 关键字重命名；
```
function v1() { ... }
function v2() { ... }

export {
    v1 as streamV1,
    v2 as streamV2,
    v2 as streamLatestVersion
};
// 
export { v1, v2 };
```

* export 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。




## npm 命令
* npm install <包的名称>            安装指定的包工具
安装好之后，包就放在了工程目录下的 node_modules 目录中，因此在代码中只需要通过 require('express')的方式就好，无需指定第三方包路径
var express = require('express');
* npm i <包的名称>                  结果同上，缩写形式
* npm i <包的名称>@版本号           安装指定版本的包
* npm i <包的名称> -g              全局安装，默认安装位置 C:\Users\ht\AppData\Roaming\npm\node_modules
* npm i <包的名称> --save          将安装包写入 package.json 依赖列表
* npm i <包的名称> --save-dev            将安装包写入 package.json 开发时依赖列表
* npm search <包的名称>            搜索包
* npm view <包的名称>              查看包
* npm uninstall <包的名称>            移除包
* npm update <包的名称>            更新包
* npm view <包的名称> version      查看可用版本

## cnpm
安装 cnpm:  npm install -g cnpm --registry=https://registry.npm.taobao.org

## nodejs 控制台
console.log();           普通输出语句
console.error();         警告输出
console.dir();
console.time(标识);      计时开始
console.timeEnd(标识);   计时结束
console.assert(表达式, 输出文字);    当表达式为false时，输出文字（抛出错误）

## nodejs 作用域
nodejs 中一个文件就是一个模块，模块中使用 var 定义的变量为局部作用域，只能在该模块中使用，因为模块在使用时，会把 nodejs 编译为一个函数，那么使用 var 定义的变量，理所当然的只能在这个模块（函数）中使用。
```
var a = 10; 
// nodejs 会在执行之前，编译这个模块为：
function (exports, requires, module, __filename, __dirname){
    var a = 10;     // a 为局部变量
}
```
* 将数据共享给其它模块使用的方法
1）暴露对象 module.exports
2）全局对象 global
```
// index.js
global.username = "刘备";

// demo.js
var obj = require('./index');
console.log(global.username);  // 此处可以省略global
```

# 异步、缓存区、文件系统
## 回调函数
* 概念
将 a 函数作为参数、传入 b 函数中，b 函数在执行过程中、根据时机或条件绝对是否调用 a 函数，a 函数就是回调函数。

## 最小执行时间
* setTimeout 的最短时间间隔是 4ms；
* setInterval 的最短时间间隔是 10ms，即小于 10ms 的时间间隔会被调整到 10ms；
* 在苹果机上的最小时间间隔是 10ms；
* 在 Windows 系统上的最小时间间隔约为 10ms。

## 异步的事件轮询机制
异步的执行始终是同步之后
```
console.log('111')
console.log('222')

setTimeout(function(){
    console.log('333')
}, 0);

console.time('t1')
for(var i=0; i<1000000000; i++){

}
console.timeEnd('t1')

console.log('444')
console.log('555')
```

## 异步的 3 种实现方式
* 回调函数
  回调函数不一定是异步，但是异步一定有回调函数
* promise 承诺对象
  
* 事件
  事件源.on('事件名称', 回调函数);
```
var http = require('http');
var server = http.createServer();
server.on('request', function(req, res){
    res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'})
    res.write('<h1>你正在访问 node.js 服务器</h1>');
    res.end();
})
server.listen(80, function(){
    console.log('服务器已运行……');
})
```

## promise
promise 是 ES6 中新增的承诺对象，用于对异步的操作进行消息的传递

### promise 状态
* Pending   等待中
* Resolved  成功
* Rejected  失败
只有两种方式： Pending => Resolved;  Pending => Rejected;

### 作用
用于传递异步消息
```
var fs = require('fs');     // 多次执行后，下面两个函数执行顺序会随机显示

fs.readFile('./file1.txt', function(err, data){
    console.log(data.toString());
})

fs.readFile('./file2.txt', function(err, data){
    console.log(data.toString());
})
```

```
var fs = require('fs');

// 将异步的数据交给 P1 进行处理
var p1 = new Promise(function(resolve, reject){
    fs.readFile('./file1.txt', function(err, data){
        if(err){
            reject('数据找不到了');
        }else{
            resolve(data.toString());
        }
    })
})

// 调用 p1 对象，统一展示异步的数据
p1.then(function(data){            // Promise.all([p1, p2]).then...
    console.log("promise 对象输出的数据：", data);
}, function(err){
    console.log("promise 对象输出的数据：", err);
})

```

## 删除文件
```
fs.unlink('./file1.txt', function(err){
    if(err){
        throw err;
    }else{
        console.log('删除了');
    }
})
```

## 递归删除非空目录

1) 删除空mull:  fs.rmdir();
2) 读取目录中的文件及文件夹列表:  fs.readdir();
3) 读取每一个文件的详细信息   fs.stat();
4) 判断如果是文件    fs.unlink();
5) 判断如果是目录:  递归调用自己
6) 删除空目录:  fs.rmdir()

## 读取流和写入流

### 概念
所有互联网传输的数据都是以流的方式，流是一组有起点有终点的数据传输方式

### 流的操作

1）流式读取文件
一节一节的读取数据，一节64kb ==> 65536字节
```
var fs = require('fs');
// 创建一个可以读取的流
var stream = fs.createReadStream('./file2.txt');

// 绑定data事件，当读取到内容时执行
stream.on('data', function(a){
    console.log('------------');
    console.log(a);
})

// 读取流的事件 end 完成事件
stream.on('end', function(){
    console.log('ended')
})

// 读取流的事件 error 完成事件
stream.on('error', function(err){
    console.log('error')
})
```

2）以流的方式写文件
```
var fs = require('fs');

// 创建一个可以写入的流
var stream = fs.createWriteStream('./file2.txt');

// 写入数据
stream.write("曹操1");
stream.write("曹操2");
stream.write("曹操3");
stream.end();     // 必须显示的声明结束

// 写入流的事件 finish 完成事件
stream.on('finish', function(){
    console.log('finished')
})

// 写入流的事件 error 错误事件
stream.on('error', function(){
    console.log('error')
})
```

## 管道

### 管道概念
管道是 node.js 中流的实现机制，直接定义一个输出流，导出输入流

### 管道语法
```
var fs = require('fs');
var s1 = fs.createReadStream('./file1.txt');
var s2 = fs.createWriteStream('./file2.txt');

// 使用管道方式实现大文件复制
s1.pipe(s2)

// 以流的方式实现大文件复制，读取一节存一节
s1.on('data', function(a){
    s2.write(a);
})
s1.on('end', function(){
    s2.end();
    console.log('文件复制完成')
})
```

## 链式流
将多个管道连接起来，实现链式处理。
```
var fs = require('fs');
var zlib = require('zlib');

var s1 = fs.createReadStream('./file1.txt');
var s2 = fs.createWriteStream('./file1.txt.zip')

s1.pipe(zlib.createGzip()).pipe(s2);
```

## url 模块
url => 全球统一资源定位符

