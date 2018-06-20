# RxJS

リアクティブ・プログラミング用のライブラリ。

非同期処理やクリックなどのイベント駆動の処理を、簡潔で一貫性を持たせ、可読性が高いコーディングができることを主な目的としている。

## リアクティブ・プログラミングとは

- 時間とともに変化する値に対する操作を宣言的に記述していくプログラミング。
- 通知されてくるデータを受け取る度に、関連したプログラムが反応して処理を行うようにするプログラミング。

## 基本概念（コアな機能）

- Observable
- Observer
- Subscription
- Operators
- Subject
- Schedulers

## Observable/Observable-Sequence（ストリーム）

- 時間とともに変化する値（データ）やイベントの流れを表したもの。
- タイマー、スクロールの位置、イベントや HTTP 経由で送られてくるデータなど、時間とともに変化する値やそのまとまりのこと。

以下のメソッドで生成可能。

- Create
- Defer
- Empty/Never/Throw
- From
- Interval
- Just
- Range
- Repeat
- Start
- Timer

### Observable のサンプル１

以下は subscribe 時に同期的に値 `1`、`2`、`3` をプッシュし、subscribe 呼び出しから 1 秒後（非同期）に値 `4` をプッシュする Observable を生成して subscribe したもの。

```js
const observable = Rx.Observable.create(observer => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
});

// ↑のObservableを呼び出して、値を確認するためには以下のようにsubscribeする
console.log('just before subscribe');
observable.subscribe({
  next: x => console.log('got value ' + x),
  error: err => console.error('something wrong occurred: ' + err),
  complete: () => console.log('done'),
});
console.log('just after subscribe');

// console.logの出力は以下の通り
// just before subscribe
// got value 1
// got value 2
// got value 3
// just after subscribe
// got value 4
// done
```

### Observable のサンプル２

以下は`fromEvent()`でイベントを Observable に変換したもの。

```js
const input = document.getElementById('input');
Rx.Observable.fromEvent(input, 'input')
  // .map（Operator）に渡されるevent（戻り値）がObservable
  // .mapでObservableを変換する
  // Operatorの戻り値もObservableのため、`map().map()`のようにチェインもできる
  .map(event => event.target.value);
```

### Observable の概念、仕組みを理解する

- Creating Observables
- Subscribing to Observables
- Executing the Observable
- Disposing Observables

#### Creating Observables

`Rx.Observable.create` は `Observable` コンストラクタのエイリアスであり、`subscribe` 関数を引数としてとる。

以下は Observable を作成し、Observer に 1 秒ごとに文字列`'hi'`を配信する。

```js
const observable = Rx.Observable.create(function subscribe(observer) {
  const id = setInterval(() => {
    observer.next('hi');
  }, 1000);
});
```

Observables は`create`だけではなく、`of`、`from`、`interval`などの[creation operators](http://reactivex.io/rxjs/manual/overview.html#creation-operators)を利用して生成できる。

#### Subscribing to Observables

上記の Observable である`observable`は以下のように subscribe できる。

```js
const observable = Rx.Observable.create(function subscribe(observer) {
  const id = setInterval(() => {
    observer.next('hi');
  }, 1000);
});

observable.subscribe(x => console.log(x));
```

subscribe をすることで、Observable execution が開始され、その execution の Observer（`subscribe`関数に渡された`observer`）に値またはイベントを渡す。

#### Executing Observables

`Observable.create(function subscribe(observer){...})`内のコードは Observable execution（subscribe する Observer ごとにのみ発生する遅延計算）である。

遅延計算とは、すぐには評価されず、結果が必要なときに評価される計算（処理）のこと。

Observable execution は、時間の経過とともに、同期的または非同期的に複数の値を生成し、Observer に通知とそれに応じた値を配信する。

Observable execution が送信する通知のタイプは以下の３つ。

- "Next" notification: 数値、文字列、オブジェクトなどの値を配信する。
- "Error" notification: JavaScript エラーまたは例外を配信する。
- "Complete" notification: 通知はするが、値は配信しない。

#### Disposing Observable Executions

`observable.subscribe` が呼び出されると、Observer は新しく作成された Observable execution にアタッチされ、Subscription であるオブジェクトも返す。

```js
const subscription = observable.subscribe(x => console.log(x));
```

Subscription は実行中の Execution を表し、その Execution を取り消すことができる最小限の API を備えている。Subscription のタイプは[こちら](http://reactivex.io/rxjs/manual/overview.html#subscription)を参照。

以下のように`subscription.unsubscribe()`を利用すると、実行中の Execution を取り消せる。

```js
const observable = Rx.Observable.from([10, 20, 30]);
// subscribeすることで、進行中のExecutionを表すSubscriptionが返される。
const subscription = observable.subscribe(x => console.log(x));

// 実行をキャンセルする
subscription.unsubscribe();
```

`create()`を使用して Observable を作成するときに、その実行のリソースを処分する方法を定義する必要がある。

関数`subscribe()`内から独自で定義した`unsubscribe`関数を返すことで、リソースを処分する。

```js
const observable = Rx.Observable.create(function subscribe(observer) {
  const intervalID = setInterval(() => {
    observer.next('hi');
  }, 1000);

  // intervalをクリアする手段を提供する
  return function unsubscribe() {
    clearInterval(intervalID);
  };
});

const subscription = observable.subscribe(x => console.log(x));
subscription.unsubscribe();
```

## Observer

Observable によって配信された通知と値を listen（observe）し、通知に応じた処理を実行するコールバックが集まったオブジェクトのこと。

Observable の通知は`next`、`error`、`complete`の 3 種類がある。そのため、基本的な Observable は以下のようなものになる。

```js
const observer = {
  // next通知に対するコールバック
  next: x => console.log('Observer got a next value: ' + x),
  // error通知に対するコールバック
  error: err => console.error('Observer got an error: ' + err),
  // complete通知に対するコールバック
  complete: () => console.log('Observer got a complete notification'),
};
```

Observer を利用するためには、Observable が提供する`subscribe()`メソッドに渡す必要がある。

`subscribe()`は Observable から配信された通知や値に応じた処理を登録するメソッドである。

これに`observer`を渡すことによって、コールバックが実行される。

```js
const observer = {
  // 新しい値がプッシュされたときに実行されるコールバック
  next: num => console.log('onNext: ' + num),
  // エラーが起きたときに実行されるコールバック
  error: error => console.log('onError: ' + error),
  // 全て値がプッシュされたときに実行されるコールバック
  complete: () => console.log('onCompleted'),
};

// subscribe()でObservableをSubscribeする
// 引数のobserverにObservableからの通知と値が配信され、コールバックが実行される
Rx.Observable.from([1, 2, 3, 4]).subscribe(observer);
// => onNext: 1
// => onNext: 2
// => onNext: 3
// => onNext: 4
// => onCompleted
```

Observer はただのオブジェクトのため、以下のように`subscribe()`に記述することも可能。

```js
Rx.Observable.from([1, 2, 3, 4]).subscribe({
  // 新しい値がプッシュされたときに実行されるコールバック
  next: num => console.log('onNext: ' + num),
  // エラーが起きたときに実行されるコールバック
  error: error => console.log('onError: ' + error),
  // 全て値がプッシュされたときに実行されるコールバック
  complete: () => console.log('onCompleted'),
});
// => onNext: 1
// => onNext: 2
// => onNext: 3
// => onNext: 4
// => onCompleted
```

オブジェクトではなく、コールバックだけ渡すこともできる。

```js
Rx.Observable.from([1, 2, 3, 4]).subscribe(
  // 新しい値がプッシュされたときに実行されるコールバック
  num => console.log('onNext: ' + num),
  // エラーが起きたときに実行されるコールバック
  error => console.log('onError: ' + error),
  // 全て値がプッシュされたときに実行されるコールバック
  () => console.log('onCompleted')
);
```

### Subscription

Observable の実行を表現したもの、主に実行をキャンセルするのに利用する。

### Operators

`map`、`filter`、`concat`、`flatMap`などの操作でコレクションを処理する関数型プログラミングスタイルを可能にする純粋関数。

### Subject

EventEmitter と同等のもの。複数のオブザーバに値またはイベントをマルチキャスト（同時に送る）できるのは Subject のみ。

Subject は Observable と Observer の両方を継承しているため、Observable でありながら Observer でもある。

### Schedulers

中央処理された dispatcher で、並行性を制御し、計算がいつ起こるかを調整できる（`setTimeout`や`requestAnimationFrame`など）。

## サンプルを実装しつつ RxJS の特徴を見ていく。

以下は native の JS で`button`のイベントリスナを登録しているサンプル。

```js
const button = document.querySelector('button');

button.addEventListener('click', () => console.log('Clicked!'));
```

これを RxJS で記述すると以下のようになる。

```js
const button = document.querySelector('button');

Rx.Observable.fromEvent(button, 'click').subscribe(() =>
  console.log('Clicked!')
);
```

これだけだと、native の JS と何も代わり映えしないため、徐々に機能を追加していく。

### Purity

RxJS の強みは、純粋な関数を使用して値を生成できること。そのため、コードのエラーが発生しにくくなる。

上記のサンプルを、クリック毎にカウントを加算するものに変更してみる。

**native の JS**

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

Rx.Observable.fromEvent(button, 'click')
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
// => 0
// => 1
// => 2
```

`scan`演算子は、配列の`reduce`と同様に機能する。

<!-- それはコールバックにさらされる値をとります。コールバックの戻り値は、次にコールバックが実行されるときに公開される次の値になります。 -->

### Flow

RxJS には、Observable にイベントがどのように流れるかを制御するためのさまざまな演算子が存在する。

以下は native の JS で、1 秒間に 1 回のクリックを許可するサンプル。

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

RxJS だと以下の通り。

```js
const button = document.querySelector('button');

Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
```

その他のフロー制御演算子は、`filter`、`delay`、`debounceTime`、`take`、`takeUntil`、`distinct`、`distinctUntilChanged`などが存在する。

### Values

Observable を通って渡された値を変換できる。

以下は native の JS で、クリックごとに現在のマウス x の位置を追加するサンプル。

```js
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
const button = document.querySelector('button');

button.addEventListener('click', event => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count);
    lastClick = Date.now();
  }
});
```

RxJS だと以下の通り。

```js
const button = document.querySelector('button');

Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .map(event => event.clientX)
  .scan((count, clientX) => count + clientX, 0)
  .subscribe(count => console.log(count));
```

他の値を生成する演算子は、`pluck`、`pairwise`、`sample`などが存在する。

## RxJS（リアクティブプログラミング）の思想、考え方

関数型プログラミング。

RxJS が提供する **ストリームを他のストリームに変換する関数** を利用する。

以下は`filter()`を利用してストリームを変換している例。

```js
// ストリームを生成して、生成したストリームを変換する処理
const source = Rx.Observable.of(1, 2, 3, 4, 5).filter(val => val < 4);

const subscribe = source.subscribe(val => console.log(val));
// => 1
// => 2
// => 3
```

最終的なストリームは 4 未満の数値である 1,2,3 を出力するものになったため、`console.log()`の出力も 1,2,3 が出力される。

### `fromEvent`

要素とイベント名をパラメータとして取り込み、その要素で発生するその名前のイベントをリッスンする。

これらのイベントを発生させる Observable を返します。

```js
var input = $('#input');

// 要素とイベント名をパラメータとして渡す
var source = Rx.Observable.fromEvent(input, 'click');

var subscription = source.subscribe(
  function(x) {
    console.log('Next: Clicked!');
  },
  function(err) {
    console.log('Error: ' + err);
  },
  function() {
    console.log('Completed');
  }
);

input.trigger('click');
```
