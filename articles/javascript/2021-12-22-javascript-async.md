# 异步JavaScript

## JavaScript Timeouts 和 intervals

**`setTimeout()`**  
`setTimeout()`在指定的时间后执行一段特定的代码。  
示例：  
```javascript
function sayHi(who){
    alert('Hello, '+who+' welcome join!');
}

let greeting = setTimeout(sayHi, 5000, 'Qijie');
```

使用`clearTimeout()`来取消上面的超时。
```javascript
clearTimeout(greeting);
```

**`setInterval()`**  
`setInterval()`每隔一段时间执行一段特定的代码。 
示例：  
```javascript
function displayTime(){
    let date = new Date();
    let time = date.toLocaleTimeString();
    document.getElementById('divDate').textContent = time;
}

let setTime = setInterval(displayTime, 2000);
``` 
使用`clearInterval()`来清除intervals。
```javascript
clearInterval(setTime);
```

可以使用递归`setTimeout()`实现每隔100毫米执行一次函数：
```javascript
let i = 1;
setTimeout(function run(){
    console.log(i);
    i++;
    setTimeout(run, 100);
}, 100);
```

那么， `setTimeout()`和`setInterval()`有什么不同吗？ `setTimeout()`保证了每次等待的时间间隔相同。不管执行函数的时间长短，每次间隔都是100毫秒。  
而使用`setInterval()`时，间隔时间包含了执行代码所花费的时间。假设执行代码需要40毫秒， 那么最终间隔只有60毫秒。

**`requestAnimationFrame()`**  
`requestAnimationFrame()` 是一个专门的循环函数，旨在浏览器中高效运行动画。  
我们没有为`requestAnimationFrame()`指定时间间隔；它只是在当前条件下尽可能快速平稳地运行它。如果动画由于某些原因而处于屏幕外浏览器也不会浪费时间运行它。  
示例   
```javascript

//requestAnimationFrame

const spinner = document.querySelector('#spinner');
let rotateCount = 0;
let rAF;
let startTime = null;
let isRunning = true;

function draw(timestamp){
    if(!startTime)
    {
        startTime = timestamp;
    }

    rotateCount = (timestamp - startTime)/3;
    if(rotateCount > 359)
    {
        rotateCount=rotateCount%360;
    }

    spinner.style.transform = `rotate(${rotateCount}deg)`;
    rAF = requestAnimationFrame(draw);
    isRunning = true;
}

draw();


document.querySelector('body').addEventListener('click', function(){
    if(isRunning)
    {
        cancelAnimationFrame(rAF);
        isRunning = false;
    }
    else{
        draw();
    }
})
```

## 使用Async callbacks（回调函数）实现异步请求

**callback示例**  
使用回调函数获取网上图片资源，并显示在页面中。
```javascript
let imgUrl = "https://www.collinsdictionary.com/images/full/apple_158989157.jpg";

function loadAsset(url, type, callback){
    let xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.responseType = type;
    xhr.onload = function(){
        callback(xhr.response);
    };
    xhr.send();
}

function displayImage(blob){
    let objUrl = URL.createObjectURL(blob);
    let image = document.createElement('img');
    image.src = objUrl;
    document.body.appendChild(image);
}

loadAsset(imgUrl, 'blob', displayImage);
```
## Promise
**Promise**是JavaScript中相对较新的功能，允许你推迟进一步的操作，直到上一个操作完成或响应其失败。

本质上，Promise是一个对象，代表操作的中间状态，它保证在未来可能返回某种结果。
1. 创建promise时，它既不是成功也不是失败状态。这个状态叫作**pending**（待定）。
1. 当promise返回时，称为 **resolved**（已解决）。
    1. 一个成功**resolved**的promise称为**fullfilled**（实现）。它返回一个值，可以通过将`.then()`块链接到promise链的末尾来访问该值。 `.then()`块中的执行程序函数将包含promise的返回值。
    1. 一个不成功**resolved**的promise被称为**rejected**（拒绝）了。它返回一个原因（reason），一条错误消息，说明为什么拒绝promise。可以通过将`.catch()`块链接到promise链的末尾来访问此原因。

Promise解决了回调函数的嵌套问题(callback hell)，在用回调函数时，你的代码可能难以阅读， 而且每个嵌套都要调用相应的错误处理。

而使用Promise改良后，我们能够一个接一个地链接多个异步操作，因为每个`.then()`块返回一个新的promise，当所有`.then()`块链接完毕, 只需要一个`.catch()`块来处理所有错误。

**Promise示例**  
使用fetch获取网上图片资源，并显示在页面中。
```javascript
let imgUrl = "https://www.collinsdictionary.com/images/full/apple_158989157.jpg";

fetch(imgUrl).then(function(response){
    if(response.type == 'cors')
    {
        return response.blob();
    }
    else if(response.type == 'text'){
        return response.text();
    }
    else{
        throw new Error(`Type not correct, expect blob. returned ${response.type}`)
    }
    console.log(response.type);
})
.then(function(blob){
    let objectURL = URL.createObjectURL(blob);
    let image = document.createElement('img');
    image.src = objectURL;
    document.body.appendChild(image);
})
.catch(error => {
    console.log(error.message);
})
```

**Promise的另一种写法**
```javascript
let imgUrl = "https://www.collinsdictionary.com/images/full/apple_158989157.jpg";

console.log('Starting...')
let promise = fetch(imgUrl);
console.log('Promise finish')
let promise2 = promise.then(response => {
    console.log('In promise 2')
   
    return response.blob();
})

let promise3 = promise2.then(blob => {
    console.log('In promise 3')
    let objUrl = URL.createObjectURL(blob);
    let image = document.createElement('img');
    image.src = objUrl;
    document.body.appendChild(image);
})
```
> [!TIP]
> 同步的`try...catch`结构不可以和Promise一起使用。

## async 和 await
`async`关键字和`await`关键字时最近添加到JavaScript语言里面的，它使得异步代码更易于编写和阅读。 把`async`关键字放在函数声明之前，使其成为异步函数。异步函数是一个知道怎样使用`await`关键字调用异步代码的函数。

我们使用异步函数和await关键字写个简单示例：
```javascript
let hello = async function(){
    return greeting = await Promise.resolve("hello");
}

hello().then(function(message){
    console.log(message);
});
```

我们将Promise方法从网上资源获取图片，显示在页面上的代码用`async`和`await`关键字重新写一遍:
```javascript
async function fetchImage()
{
    let imgUrl = "https://www.collinsdictionary.com/images/full/apple_158989157.jpg";
    let response = await fetch(imgUrl);
    let blob = await response.blob();
    let imageURL = URL.createObjectURL(blob);
    let image = document.createElement('img');
    image.src = imageURL;
    document.body.appendChild(image);
}

fetchImage().catch(error=> {
    console.log(error.message);
});
```
> [!TIP]
> 注意： 同步的`try...catch`结构可以和`async`/`await`一起使用。

以上代码可以修改为:
```javascript
async function fetchImage()
{
    try{
        let imgUrl = "https://www.collinsdictionary.com/images/full/apple_158989157.jpg";
        let response = await fetch(imgUrl);
        let blob = await response.blob();
        let imageURL = URL.createObjectURL(blob);
        let image = document.createElement('img');
        image.src = imageURL;
        document.body.appendChild(image);
    }
    catch(err)
    {
        console.log(err);
        console.log(err.message);
    }
    finally{
        console.log('finished');
    }
}

fetchImage();
```