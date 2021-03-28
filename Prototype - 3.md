# Prototype - 3

## 原型式繼承 Prototypal Inheritance
**繼承**暗示著一種 **拷貝( copy )** 作業，JavaScript 在原生預設的情況下不會拷貝物件特性，而是會在兩個物件之間建立連結，讓其中一個物件能夠將特性與函式的存取 **委派 ( delegate )** 給另一個物件。

> **委派 ( delegate )**
委派 ( delegate ) 是描述 JavaScript 的物件連結機制比較精確的用詞。

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// 這裡我們製作了一個新的 Bar.prototype 連結到 Foo.prototype
Bar.prototype = Object.create( Foo.prototype );

// 注意！現在 Bar.prototype.constructor 已經沒了
// 而且可能需要手動「修理」
// 如果你習慣仰賴這種特定性的話
Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

這裡最重要的部分是 `Bar.prototype = Object.create(Foo.prototype)`。`Object.create()` 這個方法無中生有的創建了一個「新的」物件，並將那個新物件內部的 `[[Prototype]]` 屬性連結到你指定的物件 ( 在此為 Foo.prototype )。

換句話說，`Object.create()` 的功能是：「製作一個連結到 Foo.prototype 的新物件」。此例子的新物件是 Bar.prototype。

在使用上我們很常會有一些誤解，這些 code 很有可能不會如你預期地運作：
* **範例 1**
    ```js
    // 跟你想要的不同
    Bar.prototype = Foo.prototype;
    ```
    此處不會建立一個要讓 Bar.prototype 連結的新物件，它只是直接把 Bar 連結到 Foo 所連結到的銅一個物件：Foo.prototype。
    <br/>

    這意味著，當開始進行指定 ( assigning ) 動作時，像是 `Bar.prototype.myLabel = ...`，你所修改的不是另一個物件，而是共用的 Foo.prototype 物件本身，那會影響到任何連結到 Foo.prototype 的物件。
    <br/>

* **範例 2**
    ```js
    // 有點像是你想要的
    // 但帶有你大概不會想要的副作用
    Bar.prototype = new Foo();
    ```
    這麼做迪會會正確的建立新物件，且適當的連結到 Foo.prototype ( 但這樣我們就不需要 Bar，直接使用 Foo 就好了 )，但是由於是使用 `new Foo()` 建立出來的，當 Foo 這個 function 向其他物件註冊、或是新增資料特性給 `this` 等，這些副作用就會在建立此連結的時候發生。

所以今天我們想要建立一個連結並正確的指向想連結的物件，在 ES6 之前只能使用 `Object.create()` 或是使用 `.__proto__` 特性來設定。

`Object.create()` 的壞處是，在設定時一定要建立新物件，並將就地丟掉，而不是修改提供給我們的現有預設物件。`.__proto__` 則是一種非標準、且並不完全跨瀏覽器的設定方式。

ES6 新增了一個 `Object.setPrototypeOf()` 輔助工具，以一種標準且可預測的方式來進行這種工作。我們來比較一下 pre-ES6 以及經過 ES6 標準化的技巧：
```js
// pre-ES6
// 丟掉現有的預設 Bar.prototype
Bar.prototype = Object.create(Foo.prototype);

// ES6
// 修改現有的 Bar.prototype
Object.serPrototypeOf(Bar.prototype, Foo.prototype);
```

雖然 pre-ES6 的方法需要多一建立一個物件，但實際上它比 ES6 的程式碼來的短且易讀。

## 檢視類別關係
檢視一個實體 ( 在 JS 中就是一個物件 ) 查看它的繼承世系 ( inheritance ancestry，JS 中的委派連結 )，在傳統的類別導向環境中，通常被稱為 intropection ( 內省，或反思，reflection )。

我們會使用幾種方法來檢視：
* **`instanceof`**

    `instanceof` 會檢視左邊物件的 `[[Prototype]]` 的串鏈中，右邊那個物件所任意指向的物件有沒有出現。
    ```js
    foo instanceof Foo;
    ```

    `instanceof` 只適用於物件或函式的 `.prototype` 物件是可以被測試的情況。但是如果有任意兩個物件 a 跟 b，想要知道這兩個物件是否透過某個 `[[Prototype]]` 串鏈關係到彼此，就無法使用 `instanceof`。
    <br/>
    來看看下面的例子：

    ```js
    // 查看 a 是否關聯至 b 的輔助工具
    function isRelatedTo(b, a){
        function F(){}
        F.prototype = a;
        return b instanceof F;
    }
    
    var a = {};
    var b = Object.create(a);

    isRelatedTo(b, a); // true
    ```
    在 `isRelatedTo()` 中，我們用了一個用完即丟的函式 `F`，重新指定其 `.prototype`，指向任意物件 a，然後檢查 b 是否為 F 的一個實體 ( `b instanceof F` )，顯然 b 實際上並沒有繼承或是由建構自 F。
    <br/>

* **`isPrototypeOf()`**
    `isPrototypeOf()` 是較為乾淨的作法，不需要去建立另一個物件，例如剛剛提到的 function F，就可以檢查物件的世系。

    ```js
    Foo.isPrototypeOf(a);
    ```
    在這裡就是檢查在 a 的整個 `[[Prototype]]`串鏈中，是否有出現 `Foo.prototype`。
    <br/>

    不過在 ES5 中的標準做法是使用 `Object.getPrototypeOf()`：
    ```js
    Object.getPrototypeOf(a) === Foo.prototype; // true
    ```
    <br/>

    而大多數的瀏覽器都有一個支援非標準的替代方式，用以存取內部的 `[[Prototype]]`：
    ```js
    a.__proto__ === Foo.prototype; // true
    ```
