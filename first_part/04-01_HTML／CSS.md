
# 4 HTML／CSS

## 目次
+ [フロントエンドにおけるHTMLとCSS](#フロントエンドにおけるhtmlとcss)
  + [HTMLとは](#htmlとは)
  + [CSSとは](#cssとは)
+ [HTMLの書き方](#htmlの書き方)  
  + [HTMLの基本構造](#htmlの基本構造)  
  + [HTMLでよく使われるタグ](#htmlでよく使われるタグ)  
  + [HTMLを書いてみよう](#htmlを書いてみよう)  
+ [CSSの書き方](#cssの書き方)  
  + [CSSの基本構造](#cssの基本構造)  
  + [CSSでよく使われるタグ](#cssでよく使われるタグ)  
  + [CSSを書いてみよう](#cssを書いてみよう)

<br>

---

## フロントエンドにおけるHTMLとCSS

### HTMLとは
HTML（HyperText Markup Language）は、ウェブページを作成するための基本的な言語です。HTMLはページ内の構造や内容を定義するために使用されます。HTMLを使うことで、テキスト、リンク、画像、リストなどの要素をウェブページに組み込むことができます。HTMLは、ページを「マークアップ」するためのタグ（< >）を使って情報を記述します。

### CSSとは
CSS（Cascading Style Sheets）は、ウェブページのデザインを担当する言語です。CSSを使うことで、文字の色やサイズ、配置、背景色などのスタイルを指定できます。CSSはHTMLで作ったウェブページに視覚的なデザインを加える役割を果たします。

### HTMLとCSSの関係
HTMLはページの骨組みを作り、CSSはその見た目を整えます。例えば、HTMLで「タイトル」を作り、CSSでそのタイトルの文字色やフォントを変更するといった使い方をします。このように、HTMLとCSSは密接に連携して、ユーザーに魅力的なウェブページを提供します。

<br>

---

## HTMLの書き方

### HTMLの基本構造
HTMLは次のような基本的な構造で書かれます。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ページタイトル</title>
</head>
<body>
    <h1>見出し1</h1>
    <p>これは段落です。</p>
    <a href="https://www.example.com">リンクテキスト</a>
</body>
</html>
```

- `<!DOCTYPE html>`：HTML5文書であることを宣言します。
- `<html lang="ja">`：ページの言語を指定します（日本語の場合「ja」）。
- `<head>`：ページのメタ情報（文字コードやタイトルなど）を記述します。
- `<body>`：実際に表示されるページの内容を記述します。

### HTMLでよく使われるタグ
- `<h1>～<h6>`：見出しタグ。数字が小さいほど重要度が高い。
- `<p>`：段落タグ。
- `<a href="URL">`：リンクタグ。
- `<ul>`, `<ol>`, `<li>`：リストタグ。`<ul>`は順不同リスト、`<ol>`は順序付きリスト、`<li>`はリスト項目。
- `<img src="画像のURL" alt="画像の説明">`：画像タグ。

### HTMLを書いてみよう
以下の内容を含むHTMLページを作成してみましょう。
- 見出し（`<h1>`～`<h3>`）
- 段落（`<p>`）
- リスト（`<ul>`または`<ol>`）
- リンク（`<a>`）

例：
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>サンプルページ</title>
</head>
<body>
    <h1>私の趣味</h1>
    <p>私は以下の趣味があります。</p>
    <ul>
        <li>読書</li>
        <li>映画鑑賞</li>
        <li>プログラミング</li>
    </ul>
    <p>詳細については、<a href="https://www.example.com">こちらのリンク</a>を参照してください。</p>
</body>
</html>
```

<br>

---

## CSSの書き方

### CSSの基本構造
CSSはHTMLの要素にスタイルを適用するために使用されます。基本的な構文は次の通りです。

```css
セレクタ {
    プロパティ: 値;
}
```

- `セレクタ`：スタイルを適用したいHTML要素を指定します。
- `プロパティ`：スタイルの種類（例：文字色、背景色、フォントサイズ）。
- `値`：プロパティの値（例：赤、20px、丸ゴシック）。

例：
```css
h1 {
    color: blue;
    font-size: 2em;
}

p {
    color: gray;
    line-height: 1.5;
}
```

### CSSでよく使われるタグ
- `color`：文字色を変更します。
- `font-size`：文字の大きさを変更します。
- `background-color`：背景色を変更します。
- `text-align`：文字の配置を変更します（左寄せ、中央寄せなど）。
- `margin`、`padding`：外部余白（マージン）や内部余白（パディング）を調整します。

### CSSを書いてみよう
HTMLのページにCSSを追加して、見栄えを整えてみましょう。次の例のように、スタイルを加えてみてください。

```css
h1 {
    color: green;
    font-size: 2.5em;
    text-align: center;
}

ul {
    background-color: lightblue;
    padding: 10px;
}

a {
    color: red;
    text-decoration: none;
}

p {
    font-size: 1.2em;
    margin-bottom: 20px;
}
```

このように、CSSを使ってHTMLにデザインを施すことで、ウェブページがより視覚的に魅力的になります。

---
