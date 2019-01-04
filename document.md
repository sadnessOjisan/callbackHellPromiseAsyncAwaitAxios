---
theme: "black"
thransition: "default"
---

# API からデータを取ろう講座

Yuta Ide

---

## この発表の目的

- RLS の JS 勉強会のコンテンツの１つです。
- 最終目標は React-Redux 案件を回せるようになることです。
- 2 月の redux 講座までに、次の講座を、FE 共有会のコマを使って行われます。
  - ES6 の復習をしよう講座
  - React で Todo リスト作成を復習しよう講座
  - API からデータを取ろう講座
  - React プロジェクトの環境構築を  完全に理解しよう講座
  - React のスタイリングを行おう講座

---

## 今日話すこと

- API からデータを取る方法
- 非同期処理をどう対処していくかが主な話です
- `callback hell` -> `Promise` -> `axios` -> `Async Await` -> `redux-saga`(触れる程度)
- Async Await はどのツラミを解決したのかを皆で体験してきましょう

---

##  今日のお題

サーバーから Todo 一覧を取得して、画面に表示させる

---

##  理想

```
const data = fetch('url');
render(data);
```

これは上手くいかない

---

## 何が起こったのか

- `data` に data が代入される前に、render()が実行されている
- API にデータを取りに行っている間、実行は待ってくれない
- このような処理のことを **非同期処理** という

---

## じゃあ非同期処理にいろいろな方法で立ち向かっていきましょう

---

## コールバック関数で立ち向かう

- 一般的には非同期処理を扱うメソッドには、コールバック関数を設定できる
- ここでいうコールバック関数は、非同期処理が完了した時点で勝手に実行してくれる関数である
- コールバック関数では、非同期処理の結果得た値を使うことができる

---

## jQuery の Ajax を例にすると・・・

```
  <body>
    <ul class="todo-list"></ul>
    <script>
      $.ajax({
        url: "http://localhost:3000/todos",
        type: "GET"
      })
        .done(data => {
          for (d of data) {
            $(".todo-list").append(`<li>${d.task}</li>`);
          }
        })
        .fail(err => {
          console.log(err);
        });
    </script>
  </body>
```

この例では、`.done()` `fail()` の中にコールバック関数を登録し、データを取得できたら、そのデータを元に DOM を書き換えるようにしています。

---

## なんだ簡単じゃん（完）

---

## コールバック関数を使うデメリット

---

## コールバック関数を使うデメリット 1

- データを取得する処理の中に、DOM を書き換える処理が入っている
- 責務を分けていないので、テストもしづらく、見通しも悪い

---

## コールバック関数を使うデメリット 2

- いわゆる CALLBACK HELL
- コールバック関数内のデータを元に、新たな非同期処理を実行するためには、コールバック関数の中にコールバック関数を書くことになり、ネスト地獄になる。

---

## そこで Promise

- コールバック地獄を回避することができる
- **非同期処理を同期処理にするものではない！！！**
- Promise()で非同期処理を包むと、その処理が終わった後に .then()や.catch()という処理が呼ばれる

---

## 例

```
const asyncFunction = () => {
    return new Promise(resolve, reject) => {
        if(cond){
            resolve('ok') // 関数が成功した時に実行する処理をresolveとして登録できる
        }else{
            reject('ng') // 関数が成功した時に実行する処理をresolveとして登録できる
        }

    })
}

asyncFunction()
  .then(data => {
      // 非同期処理が成功した時の処理
      // つまりresolveされたら 'ok' が dataに入った状態でこのブロックが実行される
  })
  .catch(err => {
      // 非同期処理が失敗した時の処理
      // つまりrejectされたら 'ng' が errに入った状態でこのブロックが実行される
  })
```

---

## Promise を使って Todo を取得する

```
<body>
  <ul class="todo-list"></ul>
  <script>
    fetch("http://localhost:3000/todos", {
      method: "GET"
    })
      .then(res => {
        return res.json();
      })
      .then(data => {
        for (d of data) {
          $(".todo-list").append(`<li>${d.task}</li>`);
        }
      })
        .catch(err => console.log(err));
  </script>
</body>
```

注: `fetch()`と `res()`は `Promise` を返している

---

## 注目すべき点

- `fetch()` も `res()` も 非同期処理なのに、ネストさせずに書くことができて居る
- callback hell を回避

ちなみにこう書くこともできた（CALLBACK HELL)

```
fetch("http://localhost:3000/todos", {
  method: "GET"
})
  .then(res => {
    res.json().then(data => {
      for (d of data) {
        $(".todo-list").append(`<li>${d.task}</li>`);
      }
    });
  })
  .catch(err => console.log(err));
```

---

## 言われてみれば単純、でもつまづきやすい。なぜか。

- 基本的に自分で Promise オブジェクトを作ることは無い
- Promise の解決も意識しなくていいようなライブラリ(redux-saga)や仕組み(Async/Await)が出てきている
-  でもたまに Promise そのものを要求されるので地獄を見る

---

## axios の紹介

- promise ベースの HTTP Client
- いわゆる、`便利なメソッドがたくさんあるやつ`です。
- これを使うと、fetch()と res()と 2 回の Promise の解決をしなくて済みます。

---

```
<body>
  <ul class="todo-list"></ul>
  <script>
    axios.get("http://localhost:3000/todos").then(res => {
      const data = res.data;
      for (d of data) {
        $(".todo-list").append(`<li>${d.task}</li>`);
      }
    });
  </script>
</body>
```

---

## Promise は便利だけれども・・・

- Promise によって、CALLBACK HELL は解決できた
- しかし、データの取得と DOM の操作といった責務の分割はできていない

---

## 理想

```
const data = fetch('url');
render(data);
```

---

## 実はこうすれば上手く行く

```
async function getTodos() {
  const res = await axios.get("http://localhost:3000/todos");
  return res;
}

async function renderTodo() {
  const res = await getTodos();
  const data = res.data;
  for (d of data) {
    $(".todo-list").append(`<li>${d.task}</li>`);
  }
}

renderTodo();
```

---

## 何をやったか

- 非同期処理を含む関数に `async` をつける
- `async` が付いた関数内では、 `await` を使うことで処理を待つことができる
- つまり 非同期処理の結果を受け取るまで実行を止めることができる
- データを取得してから、それを元に描画するといった風に、同期**っぽく**書くことができて居る

---

## おまけ

redux-saga では generator を使って非同期処理を同期処理っぽく扱うことができる

```
import { fork, call, take, put } from 'redux-saga/effects';
import { types, actions } from 'modules/hoge';
import hogeAPI from 'services/API';

function* hogeSaga() {
  while (true) {
    const action = yield take(types.START_FETCH);
    const param = action.payload;
    const data = yield call(hogeAPI.hoge, param);
    const { res, error } = data;
    if (res) {
      yield put(actions.successHoge(res));
    } else {
      yield put(actions.failHoge(error));
    }
  }
}
```

---

## 宣伝

- 2 月は React-Redux 勉強会
- redux, react-redux, redux-saga を扱います
- 実務への橋渡しになる回です。ぜひご参加ください。
