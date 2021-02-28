```js
var prototypeB = {
    a: 'a'
};

Object.defineProperty(prototypeB, 'a', {
    writable: false
});

var B = {
    b: 'b'
};

Object.setPrototypeOf(B, prototypeB);
B.a = 10;
```
最後我們有去改變 `B` 物件內的 `a` 屬性值，但在 console 寫下 `B.a` 或是 `protorypeB.a` 會發現他們的值都沒有被改變。
![ ](/images/prototype-1.png)

https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

會破壞原型鏈：
```js
var rebelBoy = function(name, age){
    this.name = name;
    this.age = age
};
rebelBoy.prototype.sayHi = 'Hi';

var Dimo = new rebelBoy('Dimo',31);
console.log(Dimo.__proto__);
```

如果我們對在設定 `rebelBoy` 時，是指定一個物件的話，原本 `prototype` 物件裡一些預設的方法就會被覆蓋掉。
```js
var rebelBoy = function(name, age){
    this.name = name;
    this.age = age
};
rebelBoy.prototype = {sayHi: 'Hi'};

var Dimo = new rebelBoy('Dimo',31);
console.log(Dimo.__proto__);
```
