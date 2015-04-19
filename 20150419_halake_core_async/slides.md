----

# ElectronとClojureScriptではじめてのcore.async

----

## 発表者

- 原 一浩 : [@kara_d](https://twitter.com/kara_d)
- Greative LLC
- Play frameworkでずっと開発を続けてきたが、Clojureの魅力にとりつかれ、今に至る
- 直近の著書 : [Play Framework 2徹底入門 JavaではじめるアジャイルWeb開発](http://www.amazon.co.jp/dp/4798133922)

----

## 概要

[Atom Editor](https://atom.io/)のベースにもなっている、HTML5/CSS3/JavaScriptでデスクトップアプリを作れるライブラリ/環境であるElectron（旧Atom-Shell）の開発をClojureでやりたいと思い、テンプレートを制作、公開した。

今日はcore.asyncをElectron上で使ってみる。

<blockquote class="twitter-tweet" lang="en"><p>もしかして Electron <a href="https://t.co/RRZ3F3hkTN">https://t.co/RRZ3F3hkTN</a></p>&mdash; Yasushi Abe (@yasushia) <a href="https://twitter.com/yasushia/status/589650566873161729">April 19, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

サンプルは、Electronアプリ上で5秒経つとファイルの開くダイアログを表示する。

### 注意

- 僕もはじめて使いました
- ライブコーディングで進めます
- 現在、このテンプレートは動作しません（Atom-Shellをatom-shellディレクトリ以下に配置すれば動く）
  - 本家にissue作成、反映待ち [https://github.com/atom/grunt-download-atom-shell/issues/22](https://github.com/atom/grunt-download-atom-shell/issues/22)

----

## あらすじ

昨日descjop（デスクジョップ）をリリース。

**「descjopは、HTML5/CSS3/JSでデスクトップアプリを作れるAtom-ShellのClojureScript版Leiningenテンプレート」**



GitHub : [lein-template-descjop](https://github.com/karad/lein_template_descjop)

clojars : [clojars/descjop/lein-template](https://clojars.org/descjop/lein-template)

descjopを使って、プロジェクトを作成（今回は'halake1'）

```
lein new descjop YOUR_APP_NAME
```

ビルド

```
npm install -g grunt-cli
npm install
grunt download-atom-shell
lein externs > app/js/externs.js
lein cljsbuild once
```

----

## ファイルを開くダイアログを出してみる

core.cljsの-mainに追加

```clojure
(let [dialog (nodejs/require "dialog")]
  (.showOpenDialog dialog
    (clj->js {:properties ["openFile" "openDirectory" "multiSelections"]})))
```

----

## core.asyncを使って5秒後にネイティブのダイアログを表示

project.cljに追加

```clojure
[org.clojure/core.async "0.1.346.0-17112a-alpha"]
```

core.cljsのnsに追加

```clojure
(:require-macros [cljs.core.async.macros :refer [go]])
```

core.cljsのrequireに追加

```clojure
[cljs.core.async :as async :refer [>! <! put! timeout chan]]
```

core.cljsの-mainに追加

```clojure
(let [ch (chan) dialog (nodejs/require "dialog")]
  (go
    (while true
      (let [v (<! ch)] ; 5秒後に実行
       (.showOpenDialog dialog (clj->js
         {:properties ["openFile" "openDirectory" "multiSelections"]})))))
  (go
    (<! (timeout 5000))
    (>! ch 1) ; これ大事
    ))
```
