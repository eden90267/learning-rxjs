# 函數響應式編程

## 一個簡單的 RxJS 例子

先讀代碼來學習編程吧！

“時間感覺” 網頁應用，考察你對一秒鐘感覺確認度為何？

![](https://imgur.com/CyYHEWc.png)

按著測量是否到一秒鐘的按鈕，然後鬆手，應用會告訴你鬆開和按下按鈕的時間差：

```html
<div>測試你對時間的感覺</div>
<button id="hold-me">按著我一秒鐘然後鬆手</button>
<div>你的時間：<span id="hold-time"></span>豪秒</div>
<div id="rank"></div>
```

```javascript
var startTime;

$('#hold-me').mousedown(function() {
  startTime = new Date();
});

$('#hold-me').mouseup(function() {
  if (startTime) {
    const elapsedMilliseconds = (new Date() - startTime);
    startTime = null;
    $('#hold-time').text(elapsedMilliseconds);
    $.ajax('https://timing-sense-score-board.herokuapp.com/score/' + elapsedMilliseconds).done((res) => {
      $('#rank').text('你超過了' + res.rank + '% 的用戶');
    })
  }
});
```

兩個函數交叉訪問 startTime 變數，感覺這段代碼的 “味道” 不大好，因為不得不小心處理對變數的訪問。

接下來看看用 RxJS 實現的話，代碼會怎樣：

```javascript
const holdMeButton = document.querySelector('#hold-me');
const mouseDown$ = Rx.Observable.fromEvent(holdMeButton, 'mousedown');
const mouseUp$ = Rx.Observable.fromEvent(holdMeButton, 'mouseup');

const holdTime$ = mouseUp$.timestamp().withLatestFrom(mouseDown$.timestamp(), (mouseUpEvent, mouseDownEvent) => {
  return mouseUpEvent.timestamp - mouseDownEvent.timestamp;
});

holdTime$.subscribe(ms => {
  document.querySelector('#hold-time').innerText = ms;
});

holdTime$.flatMap(ms => Rx.Observable.ajax('https://timing-sense-score-board.herokuapp.com/score/' + ms))
.map(e => e.response)
.subscribe(res => {
  document.querySelector('#rank').innerText = '你超過了' + res.rank + '% 的用戶';
});
```

這裡開始逐一解釋。

RxJS 有個特殊的對象：流 (Stream)，這裡會以：“資料流” 或者 “Observable 對象” 稱呼這種對象實例。

可以把資料流對象理解為一條河流，資料就是這條河流中流淌的水。

> 代表流的變數