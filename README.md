# RxJS
リアクティブ・プログラミング用のライブラリ。

非同期処理やクリックなどイベント駆動の処理を簡潔で可読性を高く、一貫性を持たせたコーディングが出来ることを主な目的としている。

## リアクティブ・プログラミングとは
- 時間とともに変化する値に対する操作を宣言的に記述していくプログラミングの考え方。
- 通知されてくるデータを受け取る度に、関連したプログラムが反応して処理を行うようにするプログラミングの考え方。

## 基本概念（コアな機能）
- Observable
- Observer
- Subscription
- Operators
- Subject
- Schedulers

### Observable（ストリーム）
タイマー、マウスカーソルやスクロールの位置、HTTP経由で送られてくるデータなど、時間とともに変化する値やそのまとまりのこと。
<!-- 将来の値やイベントを呼び出し可能なコレクションのアイデアを表します。 -->

### Observer
Observableによって配信された値をlisten（observe）するコールバックのコレクション(配列やオブジェクト)のこと。

### Subscription
Observableの実行を表現したもの、主に実行をキャンセルするのに利用する。

### Operators
`map`、`filter`、`concat`、`flatMap`などの操作でコレクションを処理する関数型プログラミングスタイルを可能にする純粋関数。

### Subject
EventEmitterと同等のもの。複数のオブザーバに値またはイベントをマルチキャスト（同時に送る）できるのはSubjectのみ。

### Schedulers
中央処理されたdispatcherで、並行性を制御し、計算がいつ起こるかを調整できる（`setTimeout`や`requestAnimationFrame`など）。

## サンプルを実装しつつRxJSの特徴を見ていく。
以下はnativeのJSで`button`のイベントリスナを登録しているサンプル。

```js
const button = document.querySelector('button');

button.addEventListener('click', () => console.log('Clicked!'));
```

これをRxJSで記述すると以下のようになる。

```js
const button = document.querySelector('button');

Rx.Observable
  .fromEvent(button, 'click')
  .subscribe(() => console.log('Clicked!'));
```

これだけだと、nativeのJSと何も代わり映えしないため、徐々に機能を追加していく。

### Purity
RxJSの強みは、純粋な関数を使用して値を生成できること。そのため、コードのエラーが発生しにくくなる。

上記のサンプルを、クリック毎にカウントを加算するものに変更してみる。

**nativeのJS**

```js
let count = 0;
const button = document.querySelector('button');

button.addEventListener('click', () => console.log(`Clicked ${++count} times`));
// => 0
// => 1
// => 2
```

**RxJS**

```js
const button = document.querySelector('button');

Rx.Observable
  .fromEvent(button, 'click')
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
// => 0
// => 1
// => 2
```

`scan`演算子は、配列の`reduce`と同様に機能する。
<!-- それはコールバックにさらされる値をとります。コールバックの戻り値は、次にコールバックが実行されるときに公開される次の値になります。 -->

### Flow
RxJSには、Observableにイベントがどのように流れるかを制御するためのさまざまな演算子が存在する。

以下はnativeのJSで、1秒間に1回のクリックを許可するサンプル。

```js
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
const button = document.querySelector('button');
button.addEventListener('click', () => {
  if (Date.now() - lastClick >= rate) {
    console.log(`Clicked ${++count} times`);
    lastClick = Date.now();
  }
});
```

RxJSだと以下の通り。

```js
const button = document.querySelector('button');

Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
```

その他のフロー制御演算子は、`filter`、`delay`、`debounceTime`、`take`、`takeUntil`、`distinct`、`distinctUntilChanged`などが存在する。

### Values
Observableを通って渡された値を変換できる。

以下はnativeのJSで、クリックごとに現在のマウスxの位置を追加するサンプル。

```js
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
const button = document.querySelector('button');

button.addEventListener('click', (event) => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count)
    lastClick = Date.now();
  }
});
```

RxJSだと以下の通り。

```js
const button = document.querySelector('button');

Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .map(event => event.clientX)
  .scan((count, clientX) => count + clientX, 0)
  .subscribe(count => console.log(count));
```

他の値を生成する演算子は、`pluck`、`pairwise`、`sample`などが存在する。

## Observable
Observablesは複数の値のlazy Pushコレクションです。彼らは次の表に欠けている箇所を埋める。


subscribed時に直ちに（同期的に）値1,2,3をプッシュし、subscribed呼び出しから1秒後に値4をプッシュするObservableを次に示します。

```js
const observable = Rx.Observable.create((observer) => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
});
```

Observableを呼び出して、値を取得するためにはそれをsubscribeする必要がある。

```js
const observable = Rx.Observable.create((observer) => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
});
console.log('just before subscribe');
observable.subscribe({
  next: x => console.log('got value ' + x),
  error: err => console.error('something wrong occurred: ' + err),
  complete: () => console.log('done'),
});
console.log('just after subscribe');

// 以下がconsoleに出力される
// just before subscribe
// got value 1
// got value 2
// got value 3
// just after subscribe
// got value 4
// done
```

### Pull versus Push
プルとプッシュは、データプロデューサがデータコンシューマとどのように通信できるかを記述する2つの異なるプロトコルです。

プルとは何ですか？プルシステムでは、コンシューマはデータプロデューサからデータを受信するタイミングを決定します。プロデューサ自体は、データがコンシューマにいつ配信されるかを認識していません。

すべてのJavaScript関数はプルシステムです。この関数はデータのプロデューサであり、関数を呼び出すコードは呼び出しから単一の戻り値を「引き出す」ことによってそれを消費しています。

ES2015では、プルシステムの別のタイプであるジェネレータ関数とイテレータ（関数*）が導入されました。 iterator.next（）を呼び出すコードはコンシューマであり、イテレータ（プロデューサ）から複数の値を引き出す。




<!-- 通常、コードの他の部分があなたの状態を台無しにすることがある、不純な関数を作成します。 -->

<!-- ストリームはobserveされるsubject、つまりObservableでもある。（??????） -->






## RxJS（リアクティブプログラミング）の思想、考え方
関数型プログラミング。

RxJSが提供する **ストリームを他のストリームに変換する関数** を利用する。

以下は`filter()`を利用してストリームを変換している例。

```js
// ストリームを生成して、生成したストリームを変換する処理
const source = Rx.Observable
  .of(1, 2, 3, 4, 5)
  .filter(val => val < 4);

const subscribe = source.subscribe(val => console.log(val));
// => 1
// => 2
// => 3
```

最終的なストリームは4未満の数値である1,2,3を出力するものになったため、`console.log()`の出力も1,2,3が出力される。

### `fromEvent`
要素とイベント名をパラメータとして取り込み、その要素で発生するその名前のイベントをリッスンする。

これらのイベントを発生させるObservableを返します。

```js
var input = $('#input');

// 要素とイベント名をパラメータとして渡す
var source = Rx.Observable.fromEvent(input, 'click');

var subscription = source.subscribe(
    function (x) { console.log('Next: Clicked!'); },
    function (err) { console.log('Error: ' + err); },
    function () { console.log('Completed'); });

input.trigger('click');
```