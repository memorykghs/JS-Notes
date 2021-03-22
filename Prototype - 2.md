# Prototype - 2
## 類別
在 JavaScript 中，所有函式預設都會得到一個公開的、不可列舉的特性 -- `prototype`。如果沒有特別指定，它可以指向任意的物件。
```js
function Foo(){
    // ...
}

Foo.prototype; // {}
```

這個物件被稱為是Foo的原型 ( prototype )，可以透過 `Foo.prototype` 的特性參考 ( property reference ) 來存取它。

最常看到的就是當我們使用 `new` 關鍵字來建立物件時 ( 這裡以 `new Foo()` 為例子 )，每一個創建的物件都會有 `[[Prototype]]` 連結到 `Foo.prototype` 物件。
```js
function Foo(){
    // ...
}

var foo = new Foo();
Object.getPrototypeOf(a) === Foo.prototype; // true
```

當 foo 物件透過 `new Foo()` 被創造出來時，foo 會得到一內部的 `[[Prototype]]` 連結指向 `Foo.prototype` 所指的物件。

所以其實在 JavaScript 中，我們並不會真正的去建立一個物件的實體，而是產生一個新物件 foo，foo 內部會有 `[[Prototype]]` 連結到 `Foo.prototype`。

<font color="red">補圖</font>

> 在 JavaScript 中，不會從一個物件「類別」製作拷貝到另一個物件「實體」，而是在物件之間建立連結 ( links )，而這種機制被稱為**原型式繼承** ( prototypal inheritance )。
<br/>

## 原型式繼承 Prototypal Inheritance
**繼承**暗示著一種 **拷貝( copy )** 作業，JavaScript 在原生預設的情況下不會拷貝物件特性，而是會在兩個物件之間建立連結，讓其中一個物件能夠將特性與函式的存取 **委派 ( delegate )** 給另一個物件。

```js
function Seal(){
    console.log('I am a seal.');
}

var fatSeal = new Seal();
Object.getPrototypeOf(fatSeal) === Seal.prototype; // true
fatSeal.constructor === Seal; // true
```

> **委派 ( delegate )**

## 建構器
```js
function Seal(){
    console.log('I am a seal.');
}

var fatSeal = new Seal();
fatSeal;
```
![ ](/images/prototype-2.1.png)

在這裡看起來 function `Seal` 就像是一個建構器，但實際上使用 `new` 關鍵字的時候，`Seal` 還是會立即被執行，console 第二行的結果 ( 空物件 ) 是使用 `new` 的副作用。

其實也就是說在 JavaScript 的世界裡，並沒有所謂的建構器，只要函式前面被放了 `new` 被呼叫，就可以被當作式建構器 &rArr; 任何的函示都可以是建構器。

通常我們會將被拿來當作建構器的函式，以大寫開頭來命名，不過無論是大小寫，其實對 JavaScript Engine 來說都沒有差。
