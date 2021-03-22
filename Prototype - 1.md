# Prototype - 1
## `[[Prototype]]`
所有 JavaScript 的物件都有一個內部屬性，用來儲存對另一個物件的參考，通常以 `[[Prototype]]` 表示。

如果想要調用某個物件上的屬性，`[[Get]]` 作業會先在物件本身上尋找，如果找不到該屬性，就會沿著 `[[Prototype]]` 連結尋找。

* 如果該屬性不存在於 `[[Prototype]]` 連結上，最後 `[[Get]]` 作業會回傳 `undefined`。
* `for...in` 語法會沿著 `[[Prototype]]` 上尋找，只要是可以在這個連結上找到且是可列舉 ( `enumerable: true` ) 的話，都會被印出來。
<br/>

## Object.prototype
那麼，`[[Prototype]]` 串鏈到底終結於何處呢?每條正常的 `[[Prototype]]` 串鏈最頂層的尾端都是內建的 `Object.prototyp`。

在 JavaScript 中，所有的物件都是 `Object`，所以一般情況下 `[[Prototype]]` 串鏈會指向 `Object.prototype`。

`Object.prototype` 這個物件中，會有一些原本就預設 ( 存在 ) 的方法，如下面列舉的這些：
* `.toString()`
* `.valueOf()`
* `.hasOwnProperty()`
* `.isPropertyOf()`
* `.__proto__`

所以建立的物件建立之後就可以直接使用這些方法。

## Shadowing 遮蔽
如果要設定一個物件的 property，而 `[[Property]]` 又可以追溯到某個已經存在的 property 時，會發生什麼事情呢?這裡舉了一個例子，首先有一個 seal 物件，再來我們使用 `Object.create()` 語法來複製一個物件給 copySeal。最後，改變 copySeal 物件的 `name` 屬性，賦值為 Copy Seal。
```js
var seal = {
    name: 'Seal'
};

var copySeal = Object.create(seal);

// 使用模板語法印出名稱
console.log(` seal name： ${seal.name} \n copySeal name： ${copySeal.name}`);
console.log(copySeal.hasOwnProperty('name'));

console.log('----------');
copySeal.name = 'Copy Seal';

console.log(` seal name： ${seal.name} \n copySeal name： ${copySeal.name}`);
console.log(copySeal.hasOwnProperty('name'));
```
![ ](/images/prototype-1.png)

可以看到，copySeal 會在自己的屬性上建立另一個 `name` 屬性，且不影響 Seal 物件的。

但這種結果只限定在那個已經存在的 property 屬性是 `writable: true`。如果我們將他的 `writable` 屬性設定為 `false` 的話會發生什麼情況?
```js
var seal = {};
Object.defineProperty(seal, 'name', {
    value: 'Seal',
    writable: false
});

var copySeal = Object.create(seal);

// 使用模板語法印出名稱
console.log(` seal name： ${seal.name} \n copySeal name： ${copySeal.name}`);
console.log(copySeal.hasOwnProperty('name'));

console.log('----------');
copySeal.name = 'Copy Seal';

console.log(` seal name： ${seal.name} \n copySeal name： ${copySeal.name}`);
console.log(copySeal.hasOwnProperty('name'));
```
![ ](/images/prototype-2.png)

這時候 `name` 這個屬性是不能被覆寫 ( 修改 ) 的。

在 YDKJS 這本書中有提到3個不同的狀況：
> 1. 如果在 `[[Prototype]]` 串鏈的較高層上找到一個同名的正常資料存取器特性 ( 屬性 )，且沒有被標示為**唯讀** ( read only, `writable: false` )，那麼新的同名特性就會直接被新增到新建立的物件上，造成遮蔽的效果 ( shadow property )。

> 2. 如果在 `[[Prototype]]` 串鏈的較高層上找到一個同名的正常資料存取器特性 ( 屬性 )，但被標示為**唯獨** ( read only, `writable: false` )，那麼在新物件上建立同名屬性遮蔽這件事情是不被允許的。如果程式碼是在 strict model ( 嚴格模式 ) 中執行，就會有一個錯誤被拋出。否則，設定該特性值的動作，就會無聲無息地被忽略。不管哪個方式，都不會出現遮蔽。

> 3. 如果在 `[[Prototype]]` 串鏈的較高層上找到一個屬性，且是設值器 ( setter )，無法覆蓋掉該屬性，且呼叫的也永遠是那個設值器。

第 1. 點跟地 2. 點，就像是我們上面的實驗結果，只要物件的屬性使用 `Object.defineProperty()` 去設定 `writable: false`，就無法被遮蔽，且這個先決條件下，在嚴格模式會出錯，一般模式下則是會被默認失敗 ( silently fail )，不會出現遮蔽。

那麼還有什麼情況會不小心出現遮蔽呢?來看一下這個例子。
```js
var count = {
    num: 1
};

var copyCount = Object.create(count);
console.log(` count： ${count.num} \n copyCount： ${copyCount.num}`);
console.log('----------');

copyCount.num++;
console.log(` count： ${count.num} \n copyCount： ${copyCount.num}`);
```
![ ](/images/prototype-3.png)

這樣也會導致遮蔽的情況發生，實際上做的事情是往上在 `[[Prototype]]` 串鏈尋找同名屬性，並在本身物件上新建立一個屬性，執行 `num++` 的動作後，賦值給這個新屬性。


## 參考
https://lichi-chen.medium.com/javascript-prototype-%E5%85%A8%E8%A7%A3%E6%9E%90-39c081e405c2