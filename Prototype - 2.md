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
Object.getPrototypeOf(foo) === Foo.prototype; // true
```

當 foo 物件透過 `new Foo()` 被創造出來時，foo 會得到一內部的 `[[Prototype]]` 連結指向 `Foo.prototype` 所指的物件。

所以其實在 JavaScript 中，我們並不會真正的去建立一個物件的實體，而是產生一個新物件 foo，foo 內部會有 `[[Prototype]]` 連結到 `Foo.prototype`。

![](/images/prototype-2.2.png)

> 在 JavaScript 中，不會從一個物件「類別」製作拷貝到另一個物件「實體」，而是在物件之間建立連結 ( links )，而這種機制被稱為**原型式繼承** ( prototypal inheritance )。

不過我們在使用 `new Foo()` 函式來建立 `foo` 物件這件事情，看起來跟建立 `[[Prototype]]` 連結這件事情幾乎沒有任何直接的關係 ( 感覺比較像是意外的副作用啦 )。使用 `new` 關鍵字是一種比較迂迴的方式，達到讓一個新物件連結至另一個物件。

稍後會提到一個更直接的方式來幫助我們在物件之間建立連結，那就是 `Object.create()`。
<br/>

###### 補充 
`Object.getPrototypeOf(obj)` 可以回傳指定物件的原型，也就是取得該物件的 `[[Prototype]]` 屬性的值。

如果 obj 參數在 ES5 並不是物件時會拋出 TypeError 例外，同樣狀況在 ES6 時該參數將會被強制轉換成 Object。
<br/>

## 原型式繼承 Prototypal Inheritance
**繼承**暗示著一種 **拷貝( copy )** 作業，JavaScript 在原生預設的情況下不會拷貝物件特性，而是會在兩個物件之間建立連結，讓其中一個物件能夠將特性與函式的存取 **委派 ( delegate )** 給另一個物件。

> **委派 ( delegate )**
委派 ( delegate ) 是描述 JavaScript 的物件連結機制比較精確的用詞。

## 建構器
在進入正題之前，先看一段程式碼：
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

當我們今天去使用 `new Foo()` 建立新物件 `foo`，看起來就像是我們用類別建構器來實例化一個類別。但我們也提到了，在 JavaScript 中，並不是拷貝物件，而是在建立物件的同時給他一個指向指定物件的連結 `[[Prototype]]`。

在 Java 中可以使用 `instanceof` 來判斷某物件是不是特定類別的實例，在 JavaScript 中一樣也可以使用相同的語法來判斷，問題是在 JavaScript 中是怎麼做的呢?

首先來看下面這段程式碼：
```js
function Foo(){
   // ...
}

Foo.prototype.constructor === Foo; // true

var foo = new Foo();
foo.constructor === Foo; // true 
```

這段程式碼看起來就像是 `foo` 物件上面有 `.constructor` 屬性，且這個屬性指向了「創建它的那個函式」。

但其實，`.constructor` 屬性式存在於 `Foo.prototype` 這個物件上的，並非是 `foo` 物件本身的屬性。在 `Foo` 這個函式宣告建立出來的物件上預設就會有 `.constructor` 特性。
![ ](/images/prototype-2.3.png)

所以當我們寫下 `foo.constructor === Foo;` 這個判斷式時，`[[Get]]` 作業會沿著原型鏈往上找，在 `Foo.prototype` 物件上找到 `.constructor` 特性。
![ ](/images/prototype-2.4.png)

不過這個關係很容易被破壞，如果你縣立了一個新物件並取代函式預設的 `.prototype` 物件參考，那個新物件就不會有 `.constructor` 特性。
```js
function Foo (){
    // ...
}

Foo.prototype = {
    // 建立一個新的原型物件
}

var foo = new Foo();
foo.constructor === Foo; // false
foo.constructor === Object; // true
/*
因為現在有新物件取代掉原本 prototype 物件，
所以最終會在 Object.prototype 物件上找到 constructor 特性。
*/
```

就結果而言，雖然預設的 `.constructor` 特性是不可被列舉的 ( `enumarable: false` )，但其值是可寫入的 ( 可被更改 )。由於它可以輕易的被破壞，所以某些時候這個特性是無法被信任的。