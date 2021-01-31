# JavaScript 編譯過程
當 js 在瀏覽器被載入後，基本上的流程是長的像下面這張圖這樣的：

![ ](/images/js_engine-1.png)

到目前為止，我們已經了解了執行環境，但其實 JS 直譯器每次創造/調用一個執行環境，可以再細分為兩個步驟：

*  **建立階段**
當 function 被呼叫了但在開始執行內部程式碼之前，會建立：
   1. Scope 作用域 ( 因為這原因才會有範圍鏈 Scope Chain )
   2. 為 `function` 和參數 `var` 空出記憶體空間
   3. 設定 this 的值
* **執行階段**
賦值，設定 function 的參考和解譯執行程式碼
* **回收階段**

先回憶一下上一章提到過的全域環境，在打開瀏覽器網頁時就會被建立的預設環境，一開始會幫我們設定一些東西。而打開瀏覽器頁面的動作，等同於開始載入我們寫的 HTML 及 JavaScript 原始碼，就像下面這些東西：
```html
<script src="/lib/jquery/2.1.3/jquery.min.js"></script>
<script src="https://... .js"></script>
```

所以全域執行環境建立過程中，也可以細分成上面提到的**建立階段**與**執行階段**，只是做的事情與函式執行環境稍微不同。

還記得全域環境會做什麼事嗎?以上面兩個階段來區分的話，建立階段會做：

* 建立 `global object` 全域物件
* 將 `this` 指向 `global object`
* 建立外部參考環境 ( reference environment，也就是 Scope )
* 建立參數 `var` 與函式 `function` 的記憶體空間

全域環境比函式環境多做的事情就是建立**全域物件**。而執行階段的部分可以視為開始賦值給參數，並執行 .js 檔裡面的內容了。

下面的部分我們會以函式執行環境作為主要說明兩個步驟的基礎。

### 建立階段
上面提到在進入函式執行環境 ( 也就是 `function` 被呼叫後 ) 的階段，在程式碼被執行之前，直譯器會先針對這一個執行環境作一些事：  
1. 建立外部參考環境
   也就是作用域 Scope，因為這原因才會有範圍鏈 Scope Chain。此階段會提到的一些關鍵字有：
   * **Lexical Scope 詞彙範疇**
   * **Scope** ( 這裡指外部參考環境的值 )
<br/>

2. 為 `function` 和參數 `var` 空出記憶體空間
   相關概念：
   * **Hoisting 提升**
   * **Variable Object ( VO )**
<br/>

3. 設定 this 的值
`this` 的情況相對複雜，以目前來說，只要沒有物件包著都是指向 `window`
<br/>

這3個東西是跟著函式執行環境的，概念上可以把函式執行環境想像成是 JS 直譯器底層實作的一個物件，這個物件裡面含有這些屬性：

```js
functionExecutionContext = {
  scope: {},
  variableObject: {},
  this: {}
}
```

### 執行階段
此階段做的事情相對簡單，AO 與 VO 的比較暫時放在這邊做整理。
   * **賦值**
   * **Active Variable ( AO )**
   * **暫時性死區**

以上是對 JS 編譯的過程做一個最粗略的架構劃分，下一章會提到在建立階段中的 Lexical Context 跟 Scope 分別是什麼。

## 小結
![ ](/images/js_engine-2.png)

## 參考
* https://andyyou.github.io/2015/04/18/what-is-the-execution-context-in-javascript/