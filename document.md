---
theme: "black"
thransition: "default"
---

# API からデータを取ろう講座

Yuta Ide

---

## この発表の目的

- RLS の JS 勉強会のコンテンツの１つです。
- 非同期処理苦手そうな人が多いイメージ（おれもわからん）
- 2 月の redux 講座までに、次の講座を、FE 共有会のコマを使って行われます。
  - API からデータを取ろう講座
  - React プロジェクトの環境構築を 完全に理解しよう講座

---

## 今日話すこと

- API からデータを取る方法
- 非同期処理をどう対処していくかが主な話です
- `callback hell` -> `Promise` -> `axios` -> `Async Await`
- Async Await はどのツラミを解決したのかを皆で体験してきましょう

---

## 今日のお題

サーバーから Todo 一覧を取得して、画面に表示させる

---

##  理想

```js
const data = fetch("url");
render(data);
```

これは上手くいかない

---

## 何が起こったのか

- `const data` に data が代入される前に、render()が実行されている
- API にデータを取りに行っている間、実行は待ってくれない
- このような割り込みが発生する処理のことを **非同期処理** という

---

## じゃあ非同期処理にいろいろな方法で立ち向かっていきましょう

---

## コールバック関数で立ち向かう

- 一般的には非同期処理を扱うメソッドには、コールバック関数を設定できる
- ここでいうコールバック関数は、「非同期処理が完了した時点で勝手に実行してくれる関数」である
- コールバック関数では、非同期処理の結果得た値を使うことができる

---

## jQuery の Ajax を例にすると・・・

```html
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

---

## 例

```js
const asyncFunction = () => {
    return new Promise(resolve, reject) => {
        if(cond){
            resolve('ok') // 関数が成功した時に実行する処理をresolveとして登録できる
        }else{
            reject(new Error('ng')) // 関数が成功した時に実行する処理をresolveとして登録できる
        }

    })
}

asyncFunction()
  .then(data => { // この関数がresolveされる
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

```html
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

注: `axios.get()`は `Promise` を返している

---

## 言われてみれば単純、でもつまづきやすい。なぜか。

- 基本的に自分で Promise オブジェクトを作ることは無い
  - Promise を勝手に作ってくれる(Async)が出てきている
- でもたまに Promise そのものを要求されるので地獄を見る

---

## Promise は便利だけれども・・・

- Promise によって、CALLBACK HELL は解決できた
- しかし、データの取得と DOM の操作といった責務の分割はできていない

---

## 理想

```js
const data = fetch("url");
render(data);
```

---

## 実はこうすれば上手く行く

```js
async function getTodos() {
  const res = await axios.get("http://localhost:3000/todos");
  return res; // asyncがついた関数はPromiseオブジェクトを返す
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

## 宣伝

- 2 月は React-Redux 勉強会
- redux, react-redux を扱います
- 実務への橋渡しになる回です。ぜひご参加ください。
