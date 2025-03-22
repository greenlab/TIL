# 2025/03/22～

## Reactをフロントエンドにして場合のStrapiの例

### 2025/03/22

### Reactの初期構築

```
$ npm create vite@latest my-react-app -- --template react-ts
$ cd my-react-app
$ npm install
```

### 開発用サーバー起動

```
$ npm run dev -- --host
```

後ろの「-- --host」を入れることで、開発環境からのみではなく、全てのホストからのアクセスを受け入れる。

初期構築はここまで。やらないと行けないことがほぼ無い。

### アクセス

```
http://(サーバーのIP):5174
```

### React.js に、postcss として、Tailwind cssをインストールする。

よくわからないことだらけ、かつVue.jsでも嵌った場所

#### Tailwindをインストール

プロジェクトルート上で下記を実施。
※つまり、my-react-appフォルダ下で。

```
npm install tailwindcss @tailwindcss/postcss postcss
```

#### post css の設定に、Tailwindcss を追加する。

そのために、プロジェクトルートに、postcss.config.cjs を作成

```
sudo nano
```

```
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  }
}
```

Ctrl+o で保存。Ctrl＋xでエディタ終了。
※なお、この後、文法エラーが発生します。

#### index.css を書き換え。

```
cd src  …　ルート直下のsrcフォルダに移動
sudo nano index.css
```

全部消して、下記の一文を追加。
nano で一括削除はどうやって行うのか？

```
@import "tailwindcss";
```

#### React.js 再起動

```
$ npm run dev -- --host

> my-react-app@0.0.0 dev
> vite --host

8:21:02 PM [vite] (client) Re-optimizing dependencies because lockfile has changed
Port 5173 is in use, trying another one...

  VITE v6.2.2  ready in 473 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: http://192.168.11.76:5174/
  ➜  press h + enter to show help
(node:363211) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
(Use `node --trace-warnings ...` to show where the warning was created)
8:21:16 PM [vite] (client) Pre-transform error: Failed to load PostCSS config (searchPath: /home/mi/my-react-app): [SyntaxError] Unexpected token 'export'
/home/mi/my-react-app/postcss.config.cjs:1
export default {
^^^^^^

SyntaxError: Unexpected token 'export'
    at internalCompileFunction (node:internal/vm:73:18)
    at wrapSafe (node:internal/modules/cjs/loader:1274:20)
    at Module._compile (node:internal/modules/cjs/loader:1320:27)
    at Module._extensions..js (node:internal/modules/cjs/loader:1414:10)
    at Module.load (node:internal/modules/cjs/loader:1197:32)
    at Module._load (node:internal/modules/cjs/loader:1013:12)
    at ModuleWrap.<anonymous> (node:internal/modules/esm/translators:202:29)
    at ModuleJob.run (node:internal/modules/esm/module_job:195:25)
    at async ModuleLoader.import (node:internal/modules/esm/loader:336:24)
    at async req$3 (file:///home/mi/my-react-app/node_modules/vite/dist/node/chunks/dep-B0fRCRkQ.js:14751:13)
  Plugin: vite:css
  File: /home/mi/my-react-app/src/index.css

  (以下、略)
  ```

  なかなかどういうこと？な感じ。

### Chat GPTに聞いてみる

```
https://strapi.io/integrations/react-cms

このサイトを参考にしていますが、postcss.config.cjs　で、「export」部分でsyntax error が発生します。
どのように修正すればよいのでしょうか
```
なかなか言葉足らずな質問だったが、回答してくれる。

```
postcss.config.cjs ファイルで export の部分で syntax error が発生する場合、CommonJS と ES Module の書き方の違いが原因の可能性が高いです。
（以下、略）
```
そうなの・・・！！？  
半信半疑で postcss.config.cjs を修正。

```
module.exports = {    ←ココ
  plugins: {
    "@tailwindcss/postcss": {},
  }
}
```

エラーが出なくなった。  
AIはわかっていて使うべきものと考えているが、このようなケースの場合、使う使わないどちらが良いのか悩ましいと感じた。  
※結局は深堀りするのかしないのか…といった要素に左右されるんだろうけど。

### リクエストのためにHTTPクライアントをインストールする。

やっぱりよくわからない。  
Axios と Fetch が紹介されているが、どちらも知らない。  
※DBをさわっている身として、名前的には、Fetch は敬遠したい。

といいつつ、Fetchは何もしなくても良いようなので、こちらで。

ひたすら写経しつつ、明らかに変なので、Githubのサンプル見たら、やっぱり違っていた。

と、それはそうと、一通り直してReact.js起動してみたが、一瞬だけ文字が出て、白紙になる。

おそらくデータ取れていないのだろうが、React.jsでのデバッグ方法がわかっていないので、いったん終了。

