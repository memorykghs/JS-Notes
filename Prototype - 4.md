# Prototype - 4

## 物件連結
到目前為止，`[[Prototype]]` 機制是一個內部的連結，他存在於一個物件之上，用來參考其他的物件。當對第一個物件進行特性或方法參考，而特性或方法不存在時，就會往 `[[Prototype]]` 串鏈上繼續尋找。這些在物件之間的系列連結就是所謂的「原型串鏈 ( Prototype Chain )」。

#### Create() 連結
```js
var foo = {
    something: function(){
        console.log('Tell me something good...');
    }
};

var bar = Object.create(foo);

bar.something(); // Tell me something good...
```
`Object.create()` 會建立新的物件 bar，連結到我們指定的物件 foo，賦予了 `[[Prototype]]` 機制所有的委派行為，不需要用到 `new` 函式來當作建構式呼叫。

所以在使用 `Object.create()` 時，我們不需要類別才能建立兩個物件之間的鏈結。不過，`Object.create(null)` 會建立一個具有空 ( 即「`null`」 ) `[[Prototype]]` 鏈結的物件，因此該物件無法委派 ( delegate ) 功能到任何地方。

#### Object.create() polyfill
由於 `Object.create()` 是在 ES5 新增的，在舊友環境中就需要提供 `Object.create()` polyfill 的部分功能。
```js
if(!Object.create){
    Object.create = function(o){
        function F(){}
        F.prototype = o;
        return new F();
    };
}
```
在這裡一樣用了一個用完即丟的函式 F，我們覆寫了其 `.prototype` 特性，指向我們希望連結的物件；再使用 `new F()` 建構式來製作一個新物件。

而 ES5 `Object.create()` 還有一個額外的功能，這是在舊環境無法做成 polyfill 的功能：
```js
var anotherObject = {
    a: 2
};

var myObject = Object.create(anotherObject, {
    b: {
        enumeralbe: false,
        writable: true,
        configurable: false,
        value: 3
    },
    c: {
        enumeralbe: true,
        writable: false,
        configurable: false,
        value: 4
    }
});

myObject.hasOwnProperty('a'); // false
myObject.hasOwnProperty('b'); // true
myObject.hasOwnProperty('c'); // true

myObjecy.a; // 2
myObjecy.b; // 3
myObjecy.c; // 4
```

`Object.create()` 的第二個引數指定了要新增到那個新建物件的特性名稱，方法是宣告各個新特性的**特性描述器 ( property descriptor )**。

因為要把特性描述器做成 ES5 之前環境的 polyfill 是不可能的，所以 `Obejct.create()` 這個額外功能是無法被 polyfill 的。