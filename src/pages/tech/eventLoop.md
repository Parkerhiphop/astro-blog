---
layout: "../../layouts/BlogPost.astro"
title: "那些我沒打好的前端基礎（一） - Event Loop"
description: "Event Loop Guide"
pubDate: "Nov 20 2022"
---

## 前言
工作到現在已經三年左右了，準備要換第三間公司，而且目標是日本的大外商，像是樂天，Line 之類的，這才發現我有很多沒補齊的知識，因此本系列會把我工作至今仍沒打好的基礎一一寫成文章。

這些主題可能都有很多人寫過了，但如題，這是我一直沒打好基礎的概念們，用自己的話寫一遍會更有印象之外，許多文章的作者可能是很厲害的前端了，會有一些視為理所當然的知識沒有解釋到，因此希望我的文章能幫助到跟我差不多階段的讀者們！

---

關於 Event Loop 其實一年前面試現在這間公司時有被問過一次，那時候回答得不太好，但因為其他部分表現不錯還是上了。

現在過了一年，大概知道概念是什麼，但要能在面試時好好表達出來還是不太足夠，所以第一篇就來講面試必考，實務上我們也是天天都在用的 Event Loop 機制吧！

講下去之前，不得不先說 2014 年的 [所以說event loop到底是什麼玩意兒？| Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ&list=WL&index=44&t=32s) 已經把 Event Loop 講解得挺清楚的，大家不妨先去看這部影片再回來讀一下我整理的小筆記！

## 為什麼我們需要知道 Event Loop ?

在解釋 Event Loop 之前，先來點 JS 的基礎知識。

由於 JS 是一個單執行緒（Single Thread）的語言，Single Thread 的意思就是「一次只能做一件事」。

概念就像是我們的眼睛只能聚焦在一個東西上，我們沒辦法同時看 A 又看 B，JS 也是如此，JS 不能同時執行 A 又執行 B，他是先執行 A 再接著執行 B。

### 為什麼 JS 是 Single Thread ？

JS 一開始是為了在瀏覽器運作而設計的一種腳本語言，若有多個 Thread（執行緒） 就會需要多考慮很多問題，像是：
1. 多個 Thread 同時修改 DOM
2. 多個 Thread 同時監聽 Event（像是 Click），怎麼安排執行順序？
3. Dead lock: 一次只有 Thread 能存取到資源，而多個Thread 在存取資源時互相卡死對方就叫 Dead Lock（死結）
4. ...

但是「一次只能做一件事」也意味著某段程式碼如果需要花時間來執行的話，整個程式就會停在那等它跑完（Blocking）。

在瀏覽器上的話，負責執行 JavaScript 的叫做 main thread，負責處理跟畫面渲染相關的也是 main thread。

想像一下，使用者按一個按鈕來打 API 拿資料，main thread 在等資料回來，這期間就會停止處理畫面，所以你怎麼點擊畫面都不會有反應，直到資料回來才能繼續使用，這樣的使用者體驗挺糟的吧？

要解決這個問題，就不能使用「同步」的方式來打 API，要「非同步」的打 API。

#### 同步 vs 非同步
![img](https://www.oxxostudio.tw/img/articles/201706/javascript-promise-settimeout-1.jpg)
1. 同步(sync)：一次做一件事，一個執行完才能接著執行下一個，事件的發生是「順序性的」。
2. 非同步(async)：一次可以做很多件事情，想要執行 B 事件，不用等 A 事件執行完，事件的發生是「亂序的」。

> 判斷一個行為是「同步」還是「非同步」，只要看它是不是依序執行的，或是我們是不是要等到它完成才能執行下一個事件。

像是 打 API 需要透過 HTTP Request 去跟資料庫拿或改資料的行為，在資料回來的這段時間需要確保程式還在運作，整個畫面不會被卡住，就需要「非同步」地去執行了。

等等，上面剛說完 JS 是 Single Thread，「一次只能做一件事」，這不就是「同步」的意思嗎？

沒錯！

所以為了因應這種「非同步」行為，提供更好的使用者體驗，瀏覽器才衍生出了 Event Loop 機制，來幫助 JS 能「非同步」的執行。

因此，Event Loop 並不是 JS 的機制，而是 **瀏覽器** 這個執行 JS 的環境提供的功能。
- JS 這個語言能寫在前端也能寫在後端。寫在前端的話，主要會由「瀏覽器」來執行，寫在後端的話則是由 「Node.js」來執行 ，雖然兩者都有實作 Event Loop 機制來輔助 JS 的非同步功能，但其實背後原理跟執行細節是不太一樣的哦！

以下皆是在解釋「瀏覽器」的 Event Loop。

## Event Loop 是什麼？

上面概述完了 Event Loop 誕生的背景，來談談 Event Loop 到底是什麼吧！

Event Loop 照字面上的意思來看就是「事件的循環」，因為它就像下圖一樣是「不斷在循環的事件處理機制」。

再講仔細一點，Event Loop 是一個持續調度 Call Stack 與 Task Queue 之間狀態的機制
- 當 Call Stack 為空時，Event Loop 會抓取 Task Queue 最上層任務，到 Call Stack 去執行。


![Simple Event Loop](https://ithelp.ithome.com.tw/upload/images/20190906/20103565ytBAHYgvSV.png)

用程式碼來看的話就像這樣：
```js
// Demo code from: https://blog.huli.tw/2019/10/04/javascript-async-sync-and-callback/
while(true) {
  if (callStack.length === 0 && callbackQueue.length > 0) {
    // 拿出 callbackQueue 的第一個元素，並放到 callStack 去
    callStack.push(callbackQueue.dequeue())
  }
}
```

### Event Loop in big Picture

讓我們用更大的視野來看看 Event Loop 在整個瀏覽器運作中扮演怎樣的角色

![Event Loop](https://res.cloudinary.com/practicaldev/image/fetch/s--I8K4E512--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tg7893fgvd0q8im1fy3s.png)

從圖中可以看到：
1. Event Loop 就是那個調度從 Queue 過來的資料到 Stack 去執行的一個循環(loop) 機制。
2. Queue 在這邊也分成了兩種：Micro Task Queue 和 Macro Task Queue
3. Call Stack 有在跟 Heap 往來

### Work Flow

> Sync > Async Microtask > Async Macrotask

1. Function 被呼叫後，會被推進 Call Stack 中
   - 同步的行為（Synchronous callback）會直接執行
   - 非同步行為(Asynchronous callback) 把要執行的 callback 丟到 Web API
2. Web API 的非同步行為開始執行（例如等待 3 秒），處理完後會再把要執行的 Callback 丟到 Queues
3. 當 Call Stack 清空時（同步行為都執行完畢)，Queue 的東西會透過 Event Loop 丟到 Call Stack 執行，被丟進去的 callback 就會跟一般的同步操作一樣直接執行了

不囉唆，直接上 PJ 大大的影片：

- 可以看到 log 馬上被執行
- setTimeout 則是在 Web API 那邊執行了 3 秒之後，再跑到下面的 Queue
- 等到 Call Stack 都清空後，才透過 Event Loop 推上去執行
<div class="videobox">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/N0Au8yc5IOw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>


## Event Loop 角色介紹

### Queues From Broswer

為了循序漸進，上面的流程其實省略了 Queue 內部的運作，因為 Queue 還有分 Mircotask 和 Macrostack 兩種。

![Micro & Macro](https://i.imgur.com/UGsPMv8.png)

#### Macrotask Queue / Task Queue  / Event Queue

- 如果看到人只講 Task ，通常就是指 Macrotask，以下也會簡化成 task
- 詳細 macrotask 在 render 前執行的順序範例可參考 [Event Loop 運行機制解析 - 瀏覽器篇](https://yu-jack.github.io/2020/02/03/javascript-runtime-event-loop-browser/)
- 但大意來說，DOM Manipulation 只會吃到最後一個值來 render，像是重複改變 `document.body.style`的值，只會顯示最後一個。
- 而 setTimeout 裡的 callback 如果等的時間大於瀏覽器更新的頻率時，則會在下次 render 才執行。
  - 如果瀏覽器是以 60 fps 進行的話, 代表說這個 setTimeout 時間只要大於間隔 16.7 ms，就會在下次 render 才執行

Examples
1. initial script
   - 頁面載入要執行 JS 的 script tag 這件事情，瀏覽器將之視為一個 Macro task
2. `setTimeout`
3. `setInterval`
4. The event callback of user interaction
   - ex: click event 裡的 callback
5. The DOM manipulation
   - ex: `document.body.style = 'background:yellow'`
6. The networking
   - ex: ajax 觸發時的 callback
7. The history traversal
   - ex: `history.back()` 

#### Mircotask Queue / Job Queue

- 每一輪 Event Loop 中會確保當前的 Microtask 全數執行完畢
- 與 task 不同，microtask 一定都會在這次 render 內執行，不會出現像 setTimeout 那樣在下次 render 才執行的情況
- 當 Microtask queue 清空後，頁面才會進行 re-render

Examples
1. `Promise`
   - 但其實 Promise 中的程式會直接執行，不會進入 microTask， `.then`  中的程式才會進 microTask
2. `MutationObserver`
3. `queueMicrotask`
4. Any techniques based on promise API such as fetch API

#### 在一次 Event Loop 中，Queue 的運作流程

因為 [JS 原力覺醒 Day15 - Macrotask 與 MicroTask](https://ithelp.ithome.com.tw/articles/10222737) 留言中[Nono](https://ithelp.ithome.com.tw/users/20111995/profile) 的解釋很清楚，以下直接節錄過來：

1. 先從 task queue 中，找出一個最優先被加入的 macrotask (oldest task)，並放進 event loop。
   - 若 task queue 中沒有 macrotask 存在，則直接跳過 2. 到步驟 3.
2. 執行 macrotask，把 callback 丟到 task queue 中
3. macrotask 執行完後，此時 callback stack 應該要是空的，才執行 microtask
4. 執行 microtask 之中，若又有新的 microtask 被產生，則回到 3. 執行到 microtask queue 為空為止
5. microtask 執行完才開始做頁面的渲染 (不一定每次都做)
6. 重複 1-5

Example
```js
setTimeout(() => alert("timeout"));

Promise.resolve()
  .then(() => alert("promise"));

alert("global ex. context");
```

1. 發現 marcotask - delay time 為 0 的 setTimeout，直接把它的 callback 放入 task queue 中
2. Promise.resolve() 會產生 microtask，放入 microtask queue 中
3. 執行 alert
4. 此 task 執行完畢，發現有 microtask queue 中有 microtask，且此時 callback stack 也為空，執行 promise 的內容
5. 確認 micro queue 為空後，開始做頁面的重繪 (此次 event loop 結束)
6. 再來是新的一圈，發現 task queue 中有 macrotask，所以拿出來執行 setTimeout 的 callback

#### 觀念釐清
1. 在每個 macroTask 執行完之後會執行 microTask
2. 當有 microTask 執行時，任何瀏覽器的渲染（render）行為必須等到 MicroTask queue 清空後才執行，而且 microＴask 會一直堆疊，只要有 microTask 一直產生，頁面就會被迫停著等待，所以使用 Promise 時要小心。
3. 步驟 2. 中被放在 task queue 中的 task，是在下一次的 event loop 的一開始才執行，很多人會因此誤會 macrotask 是在 microtask 之後執行，其實恰恰相反


### Heap & Call Stack - From JS Engine

![Stack & Heap](https://miro.medium.com/max/720/1*JwkWC-SlF4hbvyZJP33duQ.png)

這邊的 Call Stack 和 Heap 是指記憶體的空間，而不是指資料結構的 Stack & Heap。

這兩個屬於 JS Engine 的功能，而不是瀏覽器所提供的，畢竟任何一個程式語言都會需要與記憶體互動。

但其實 Event Loop 主要是在 監控 Call Stack 跟 調用 Queue 的一個機制，JS Engine 中 Call Stack 與 Heap 的運作已經不屬於今天討論的範圍了，有興趣的可以參考 [Kylo Mo -Day26 X Memory Management In JavaScript](https://ithelp.ithome.com.tw/articles/10280288)。（也可以直接去買「今晚來點Web前端效能優化大補帖」這本書哦！）

### Web API

- Browser 提供了很多不同的 API（例如：DOM、AJAX、Timeout），讓我們能夠同時（concurrently）處理多項任務
- 當完成 Web APIs 的內部函式後（如 `setTimeout()`），便將任務傳遞至 Queue。
    - setTimeout 在 call stack 執行完後，會將要執行的東西丟到 Queue 計時
- 非同步的行為都是 Web API 所提供。
- 更多 API 請參考 [MDN - Web APIs](https://developer.mozilla.org/zh-TW/docs/Web/API)

## 實戰演練

理解了 Event Loop 之後，我們就能知道 非同步事件 是不會馬上執行的，而只是被丟到 Queue 等著被呼叫，而如果 Call Stack 還有 function 在執行時，會被 pending。

### 演練一: `setTimeout`

`setTimeout` 時間到時不會馬上執行，因為它前面還有 call stack 的東西再執行，它的執行時間是「 call stack 執行時間 ＋ 等待的秒數」

因此確保的是在「至少」3 秒後，而不是真的 3 秒後，因為它需要等一般的同步行為都先執行玩，才會執行。

```jsx
const s = new Date().getSeconds();

// 應該要在 500ms 後執行
setTimeout(function() {
  // 但因為被下面的 while 擋了兩秒，所以會在 2.5 秒後才執行
  console.log("Ran after " + (new Date().getSeconds() - s) + " seconds");
}, 500)

while (true) {
  // 兩秒後執行 log
  if (new Date().getSeconds() - s >= 2) {
    console.log("Good, looped for 2 seconds")
    break;
  }
}
```

### 演練二：Promise
Promise 中的程式會直接執行，不會進入 microtask，then 中的程式才會進 microtask

```js
main();

function main() {
  console.log('------ start ------');

  asyncFn(1);
  console.log('------ middle ------');

  setTimeout(() => console.log('[macrotask] - Timeout', 0));

  console.log('------ end ------');
}

async function asyncFn(taskNumber) {
  console.log(`[microtask - ${taskNumber}] - in async start`);
  const _taskNumber = await promiseFn(taskNumber);
  console.log(`[microtask - ${_taskNumber}] - after await`);

  console.log(`[microtask - ${_taskNumber}] - in async end`);
}

function promiseFn(taskNumber) {
  return new Promise((resolve) => {
    console.log(`[microtask - ${taskNumber}] - in promise`);
    resolve(taskNumber);
  }).then((taskNumber) => {
    console.log(`[microtask - ${taskNumber}] - in promise.then()`);
    return taskNumber;
  });
}

/**
------ start ------
[microtask - 1] - in async start
[microtask - 1] - in promise
------ middle ------
------ end ------
[microtask - 1] - in promise.then()
[microtask - 1] - after await
[microtask - 1] - in async end
[macrotask] - Timeout 0
*/
```
Demo Code from [Event loop, micro-task, macro-task, async JavaScript 筆記](https://pjchender.dev/javascript/note-event-loop-microtask/#promise-%E4%B8%AD%E7%9A%84%E7%A8%8B%E5%BC%8F%E6%9C%83%E7%9B%B4%E6%8E%A5%E5%9F%B7%E8%A1%8C%E4%B8%8D%E6%9C%83%E9%80%B2%E5%85%A5-microtaskthen-%E4%B8%AD%E7%9A%84%E7%A8%8B%E5%BC%8F%E6%89%8D%E6%9C%83%E9%80%B2-microtask)

### 演練三： Async await
```js
main();

function main() {
  console.log('------ start ------');

  asyncFn(1);

  console.log('------ middle ------');

  setTimeout(() => console.log('[macrotask] - Timeout', 0));

  console.log('------ end ------');
}

async function asyncFn(taskNumber) {
  console.log(`[microtask - ${taskNumber}] - in async start`);

  const _taskNumber = await promiseFn(taskNumber);

  console.log(`[microtask - ${_taskNumber}] - after await`);

  console.log(`[microtask - ${_taskNumber}] - in async end`);
}

function promiseFn(taskNumber) {
  return new Promise((resolve) => {
    console.log(`[microtask - ${taskNumber}] - in promise`);
    resolve(taskNumber);
  });
}

/*
------ start ------
[microtask - 1] - in async start
[microtask - 1] - in promise
------ middle ------
------ end ------
[microtask - 1] - after await
[microtask - 1] - in async end
[macrotask] - Timeout 0
*/
```

Demo Code from [Event loop, micro-task, macro-task, async JavaScript 筆記](https://pjchender.dev/javascript/note-event-loop-microtask/#async-function-%E4%B8%AD-await-%E5%BE%8C%E7%9A%84%E5%85%A7%E5%AE%B9%E6%9C%83%E9%80%B2%E5%85%A5-microtask)


另外還有這些地方有提供案例：
1. [JavaScript 中的同步與非同步（上）：先成為 callback 大師吧！](https://blog.huli.tw/2019/10/04/javascript-async-sync-and-callback/) 
2. [[筆記] 理解 JavaScript 中的事件循環、堆疊、佇列和併發模式（Learn event loop, stack, queue, and concurrency mode of JavaScript in depth）](https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html)
   
也可以用這個視覺化工具玩玩看： [✨♻️ JavaScript Visualized: Event Loop - DEV Community 👩‍💻👨‍💻](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif?fbclid=IwAR3ryonWDJ-31yrr-s1vucfRmRlQc0db2YWLzxSnoWKWkEiUSLkz1zE9jVQ)

## 結語 - 2W1H

### Why?
- Event Loop 是為了解決 JS Single Thread 而誕生的機制，由執行環境（瀏覽器、Node.js）所提供。

### What?
- Event Loop 是一個不斷在 監控 Call Stack 跟 調用 Queue 的一個循環機制

### How?
- 同步行為會直接丟到 Call Stack 執行，非同步則是會先丟到 Queue
- Event Loop 會去監控 Call Stack 空了沒，空了的話就會去調用 Queue 裡的 callback 來執行
- 在 Queue 當中又有分成 macrotask(task) 跟 microtask
  - 被丟進 macrotask queue 的 callback 會在下一輪 Event Loop 的一開始所執行
  - 被丟進 mircotask queue 中的 callback 待全部執行完畢後才會進行畫面的渲染（所以也有可能在這階段因為無限產生的 microtask 而卡住瀏覽器的 render）
- 瀏覽器渲染後，宣告一輪 Event Loop，接著繼續新一輪的 Event Loop，以此不斷循環

但其實除了 Event Loop 之外，還有一個叫 Worker 的東西可以來解決 JS Single Thread 的限制，但就留待下一篇文章介紹囉！

## Reference

1. [所以說event loop到底是什麼玩意兒？| Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ&list=WL&index=44&t=32s)
2. [JS 原力覺醒 Day15 - Macrotask 與 MicroTask](https://ithelp.ithome.com.tw/articles/10222737)
3. [Day5 [JavaScript 基礎] Event Loop 機制](https://ithelp.ithome.com.tw/articles/10214017)
4. [JavaScript 中的同步與非同步（上）：先成為 callback 大師吧！](https://blog.huli.tw/2019/10/04/javascript-async-sync-and-callback/)
5. [Event Loop 運行機制解析 - 瀏覽器篇](https://yu-jack.github.io/2020/02/03/javascript-runtime-event-loop-browser/)
6. [✨♻️ JavaScript Visualized: Event Loop](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif)
7. [Why is this microtask executed before macrotask in event loop?](https://stackoverflow.com/questions/52019729/why-is-this-microtask-executed-before-macrotask-in-event-loop)
8. [https://html.spec.whatwg.org/#microtask-queue](https://html.spec.whatwg.org/#microtask-queue)

