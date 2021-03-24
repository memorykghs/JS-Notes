# Scope - 1
在傳統的編譯式語言處理過程中，原始碼 ( source code ) 在執行之前，通常會經歷3個步驟，我們將這段過程通稱為「編譯」。

1. **Tokenizing ( 語法基本單元化 ) 與 Lexing ( 語彙分析 )**：簡單來說就是將語法拆成一些更細小的單元，如 `var a = 2;` 可能會被拆成 `var`、`a`、`=`、`2`及`;`。

2. **Parsing ( 剖析，或「語法分析」)**：JavaScript engine 會為前面的基本語法單元 ( tokens ) 轉換為程式的文法結構 ( grammatical structure )，也就是 「AST」 ( Abstract Syntax Tree )。

3. **Code-Generation ( 產生目的程式碼 )**：接受一個 AST 並將之轉為可執行程式碼的過程。

而在一段 JavaScript 程式碼被執行的過程中，會有3個角色。
* **Engine (引擎)**
負責執行 JavaScript 程式。
<br/>

* **Complier ( 編譯器 )**
處理剖析 ( parsing ) 與程式碼產生，也就是上面提到的編譯過程所有事情。

* **Scope ( 範疇 )**
負責收集並維護所有由已宣告的變數所構成的一個查找清單，並且會有規則來規範這些變數對於目前正在執行的程式碼而言，是否可以取用。

上述的3個角色，白話一點來說就是當宣告一個變數 `a`，Engine 會先查找 Scope 中是否已經存在 `a` 這個變數，如果有，Engine 就會忽略這個宣告。否則就會在 Scope 中，新增一個叫做 `a` 的新變數，或是往他處尋找。

## RHS 與 LHS
上述提到的，如果今天在程式碼宣告了一個 `a` 變數 `var a;`，那麼 Engine 會去 Scope 中查找這個變數是否被宣告過。這時候我們會稱 Engine 會為變數 `a` 進行一種 LHS ( Lefthand side ) 查找動作；與之相對的式 RHS ( Righthand side )。

所謂的左手邊與右手邊並非代表在等號的左邊或是右邊，而是根據 Engine 進行的某個指定作業 ( an assignment operation ) 的左手邊或右手邊。

講到這邊其實會覺得有點抽象，簡單來說：
> * RHS 查找：查找某個變數的**值**
> * LHS 查找：試著找出該變數的容器本身 ( variable container )，並指定某個東西給它

我們先來看 LHS 的例子。我們會說下面這段程式碼是一個 LHS 參考。
```js
a = 2;
```

