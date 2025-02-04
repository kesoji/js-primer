---
author: laco 
description: "markedパッケージを使ってMarkdownファイルをHTMLに変換します。"
---

# MarkdownをHTMLに変換する {#md-to-html}

前のセクションではコマンドライン引数で受け取ったファイルを読み込み、標準出力に表示しました。
次は読み込んだMarkdownファイルをHTMLに変換して、その結果を標準出力に表示してみましょう。

## markedパッケージを使う {#use-marked-package}

JavaScriptでMarkdownをHTMLへ変換するために、今回は[marked][]というライブラリを使用します。
markedのパッケージはnpmで配布されているので、commanderと同様に`npm install`コマンドでパッケージをインストールしましょう。

```shell-session
$ npm install --save marked@0.6
```

インストールが完了したら、Node.jsのスクリプトから読み込みます。
前のセクションの最後で書いたスクリプトに、markedパッケージの読み込み処理を追加します。

[import title:"main.js"](src/main-0.js)

markedパッケージをインポートして取得した`marked`関数は、引数のMarkdown文字列をHTML文字列に変換して返します。
次のように`readFile`関数で読み込んだファイルの文字列を引数として渡せば、HTMLに変換できます。

[import title:"main.js"](src/main-1.js)

## 変換オプションを作成する {#create-convert-option}

markedにはMarkdownの[変換オプション][]があり、オプションの設定によって変換後のHTMLが変化します。
いくつかのオプションについてアプリケーション中でのデフォルトの設定を決め、さらにコマンドライン引数から設定を切り替えられるようにしてみましょう。

今回のアプリケーションでは次の2つのオプションを扱います。

- gfm
- sanitize

### gfmオプション {#gfm-option}

`gfm`オプションは、GitHubにおけるMarkdownの仕様([GitHub Flavored Markdown][], GFM)に合わせて変換するかを決めるオプションです。
markedではこの`gfm`オプションがデフォルトで`true`になっています。GFMは標準的なMarkdownにいくつかの拡張を加えたもので、代表的な拡張がURLの自動リンク化です。
例として、次のようなMarkdownファイルを用意し、先ほどのスクリプトと、`gfm`オプションを`false`にしたスクリプトで結果の違いを見てみましょう。

[import, サンプルMarkdownファイル](src/sample.md)

`gfm`オプションが有効のときは、URLの文字列が自動的に`<a>`タグのリンクに置き換わります。

```html
<h1 id="-">サンプルファイル</h1>
<p>これはサンプルです。
<a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul>
```

一方、次のように`gfm`オプションを`false`にすると、単なる文字列として扱われ、リンクには置き換わりません。

[import gfmオプションを無効にする](src/main-2.js)

```html
<h1 id="-">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul>
```

自動リンクの他にもいくつかの拡張がありますが、詳しくは[GitHub Flavored Markdown][]のドキュメンテーションを参照してください。

### sanitizeオプション {#sanitize-option}

`sanitize`オプションは出力されるHTMLを安全な形にサニタイズするためのオプションです。
`sanitize`オプションが有効なとき、Markdownファイル中に書かれたHTMLタグはエスケープされ、単なる文字列として出力されます。
例として次のようなMarkdownファイルの変換が`sanitize`オプションによってどう変わるかを見てみましょう。

[import sample.md](src/sample-1.md)

`sanitize`オプションのデフォルト値は`false`です。
そのため、何も指定しなければMarkdownファイル中のHTMLはそのまま出力されるHTML中でもタグとして残ります。

```html
<h1 id="-">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<p>これはHTMLです</p>

<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul>
```

次のように`sanitize`オプションを有効にすると、`<`と`>`がエスケープされてHTMLタグとして機能しなくなります。
自由にHTMLを書かれては困る場合に有用なオプションです。

[import sanitizeオプションを有効にする](src/main-3.js)

```html
<h1 id="-">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<p>&lt;p&gt;これはHTMLです&lt;/p&gt;

</p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul>
```

### コマンドライン引数からオプションを受け取る {#receive-option}

それぞれの変換オプションについて、コマンドライン引数で制御できるようにします。
`gfm`オプションは`--gfm`、`sanitize`オプションは`--sanitize`と`-S`でコマンドラインから設定できるようにします。

<!-- 差分コードなので -->
<!-- doctest:disable -->
```js
program
    .option("--gfm", "GFMを有効にする")
    .option("-S, --sanitize", "サニタイズを行う");

program.parse(process.argv);
```

### デフォルト設定を定義する {#declare-default}

毎回すべての設定を明示的に入力させるのは不便なので、それぞれの変換オプションのデフォルト設定を定義します。
今回は`gfm`オプションと`sanitize`オプションをどちらもデフォルトで`false`にします。
アプリケーション側でデフォルト設定を持っておくことで、将来的にmarkedの挙動が変わったときにも影響を受けにくくなります。

markedのオプションはオブジェクトを渡す形式です。
オブジェクトのデフォルト値を明示的な値で上書きするときには`...`（spread構文）を使うと便利です。([オブジェクトのspread構文][]を参照)

次のようにデフォルトのオプションを表現したオブジェクトに対して、`program.opts`メソッドの戻り値で上書きします。

<!-- 差分コードなので -->
<!-- doctest:disable -->
```js
const markedOptions = {
    gfm: false,
    sanitize: false,
    // オプションのkey-valueオブジェクトをマージする
    ...program.opts()
};
```

あとは`markedOptions`オブジェクトからmarkedにオプションを渡すだけです。
スクリプト全体は次のようになります。

[import title:"main.js"](src/main-4.js)

定義したコマンドライン引数を使って、Markdownファイルを変換してみましょう。

```shell-session
# gfmオプションを有効にする
$ node main.js --gfm sample.md
# sanitizeオプションを短縮形で有効にする
$ node main.js -S sample.md
```

これでMarkdown変換の設定をコマンドライン引数でオプションとして与えられるようになりました。

## このセクションのチェックリスト {#section-checklist}

- markedパッケージを使ってMarkdown文字列をHTML文字列に変換した
- コマンドライン引数でmarkedの変換オプションを設定した
- デフォルトオプションを定義し、コマンドライン引数で上書きできるようにした

[marked]: https://github.com/chjj/marked
[変換オプション]: https://marked.js.org/#/USING_ADVANCED.md#options
[GitHub Flavored Markdown]: https://github.github.com/gfm/
[オブジェクトのspread構文]: ../../../basic/object/README.md#object-spread-syntax