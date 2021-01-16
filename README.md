# JS-Notes
JavaScript個人學習筆記
- [ ] 閉包被做的時機
- [ ] 執行時期做的事情detail

```js
function print(i){
    function A(){
      console.log('counter is ' + i)
    }
    A();
  }
  
  function counter() {
    
    for (var i = 0; i< 5; i++) {
        
        setTimeout(print(i), 1000);
        i=7;
    }
  }
  
  counter();
```
