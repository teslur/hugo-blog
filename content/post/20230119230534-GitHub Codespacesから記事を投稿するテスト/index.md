---
title: "GitHub Codespacesから記事を投稿するテスト"
date: 2023-01-19T14:05:35Z
draft: false

#description: "Example article description"
categories:
  - "技術"
tags:
  - "Hugo"
  - "GitHub Codespaces"
#menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
#thumbnail: "img/placeholder.png" # Thumbnail image
#lead: "Example lead - highlighted near the title" # Lead text
#comments: false # Enable Disqus comments for specific page
#authorbox: true # Enable authorbox for specific page
#pager: true # Enable pager navigation (prev/next) for specific page
#toc: true # Enable Table of Contents for specific page
#mathjax: true # Enable MathJax for specific page
#sidebar: "right" # Enable sidebar (on the right side) per page
#widgets: # Enable sidebar widgets in given order per page
#  - "search"
#  - "recent"
#  - "taglist"
---

このブログの執筆環境を試行錯誤するシリーズ。  
これまでは

- ローカルマシンでgit管理したmarkdownファイルをvimで編集してPushする
- [Netlify CMS](https://www.netlifycms.org/)を導入してブラウザでmarkdownファイルを編集してPublishする

を試した。  
今回は[GitHub Codespaces](https://github.co.jp/features/codespaces)を使ってブラウザからmarkdownファイルを編集しGUIでPushを行う方法を試す。

<!--more-->

## やること
### Codespacesの有効化
このブログ記事を管理しているGitHubリポジトリでCodespacesを有効にする。  
具体的には「Code」タブの「Code」ボタンからCodespacesを有効にする。

{{< figure src="./images/activate.png" class="center" caption="Codespacesを有効化する（画像ではすでに有効化されている）" alt="Codespacesを有効化" >}}

これだけで以下のようにVSCodeがブラウザで開く。

{{< figure src="./images/vscode_on_browser.png" class="center" caption="ブラウザ上で起動したVSCode。ターミナルも使える。" alt="ブラウザ上で起動したVSCode" >}}

CodespacesではデフォルトでHugoがインストール済みなので気軽に使い始められる。  
バージョンなどが自分の欲しいものと合っているかは要確認。

```shell
$ hugo version
hugo v0.108.0-a0d64a46e36dd2f503bfd5ba1a5807b900df231d linux/amd64 BuildDate=2022-12-06T13:37:56Z VendorInfo=gohugoio
```

また、テーマをgit submoduleで使用している場合はsubmoduleをPullしておく。

```shell
$ git submodule update --init
```

これで環境自体は作れた。  
早い。凄い。

### デフォルトの記事ディレクトリ構成を変える
これまで記事はすべてHugoのルートディレクトリ直下の`content`ディレクトリ内にmarkdownファイルが作成されていた。  
マークダウンファイルは`{ファイル作成日時}-{記事タイトル}.md`の形式で命名されている。

Netlify CMSも（少なくともデフォルトでは）この形式だったが、これだと記事ファイル以外の扱いが面倒。  
例えば画像ファイルは`static/img/xxx.png`のような形式で格納するのだが、この形式だと当然ファイル名の重複がNGになる。  
じゃあ`static/img/{サブディレクトリ名}/xxx.png`の形式にすればよいかというと、今度はサブディレクトリ名をどうするか問題が出てくる。  
記事ファイルと同名（＝記事タイトル）にするとディレクトリ名が冗長になって記事ファイルから参照するのがとても面倒。

```markdown
![記事で使う画像](/img/20230119230534-GitHub\ Codespacesから記事を投稿するテスト/xxx.png)
```

少なくとも補完が聞かない限り使いたくない。  
ついでに記事生成時に同名ディレクトリを作ってくれないと困るのでカスタマイズが必要そうでダルい。

要は記事と記事で使っている画像は同一ディレクトリに入っていてほしいし、記事から画像ファイルを参照するときは相対パスで指定したい。  
ので、そういった構成になるよう`archetypes`ディレクトリ内にファイルを作成する。

```shell
$ tree archetypes
archetypes
├── default.md
└── post          # 作る
    ├── images    # 作る
    │   └── .keep # 作る
    └── index.md  # 作る。default.mdのコピーでいい。
```

こうしておくと以下のように`post/`配下に記事を作成するコマンドを実行したときに`post/{記事タイトル}/index.md`ファイルと`post/{記事タイトル}/images/`ディレクトリを作成してくれる。

```shell
$ hugo new post/sample
Content dir "/workspaces/hugo-blog/content/post/sample" created
$ tree content/post/sample/
content/post/sample/
├── images
└── index.md

1 directory, 1 file
```

上記はHugoの機能（[参考](https://gohugo.io/content-management/archetypes/)）。

この状態なら`images`ディレクトリ内に画像ファイルを配置し、参照は以下のようにできる。

```markdown
![記事で使う画像](./images/xxx.png)
```

Hugoで使えるFigure Shortcodeも相対パスに対応しているので使える。

```
{{</* figure src="./images/xxx.png" */>}}
```

### 記事ディレクトリを生成する
VSCodeのターミナルで記事を作成するhugoコマンドを実行する。  
記事名ディレクトリに日時を含みたいので、今の所以下のようなコマンドを叩いている。

```shell
$ hugo new "post/`TZ=Asia/Tokyo date +%Y%m%d%H%M%S`-GitHub Codespacesから記事を投稿するテスト"
```

ダブルクォートで囲んでいるのはスペースを引数の区切りと認識させないため。

このコマンドで記事ディレクトリを生成すると記事のタイトルにも日時が入ってしまうので、生成された`index.md`を編集する。  
色々めんどくさいのでこの辺は適当なシェルスクリプトにしてしまったほうが良いかもしれない。  
オレオレVSCode拡張とかにしてボタンを割り当てたらもっと幸せ…なのかもだけど道のりは遠そう。

### 記事を書く
VSCodeをマークダウンエディタとして利用して記事を書く。  
デフォルトでオートセーブに対応している。

多分Hugoを使いやすくする拡張機能があるようなので、今後導入してみる。

## 便利機能
GitHub Codespacesはコンテナ環境でポート転送に対応している。  
何を言っているかというと、`hugo server`コマンドで立ち上げたサーバーにブラウザアクセスして、作成中の記事のプレビューが見れる。

ただしサーバーの`baseurl`がGitHubから払い出されるものになるのでそれを指定して起動する。

```shell
$ hugo server --baseURL https://$CODESPACE_NAME-1313.githubpreview.dev
```

そうするとVSCodeのターミナルの隣の「ポート」タブにポート転送で利用できるURLが「ローカルアドレス」として出てくるので、これを⌘+左クリックで開くと開発サーバー画面にアクセスできる。

{{< figure src="./images/port.png" class="center" caption="「ポート」タブ→「ローカルアドレス」でHugoの開発サーバーにアクセスする" alt="Hugoの開発サーバーのアドレスをポート転送" >}}

{{< figure src="./images/dev_server.png" class="center" caption="開発サーバー画面にアクセスできる" alt="開発サーバー画面にアクセス" >}}

## 締め
GitHub Codespacesを使った記事投稿をやってみる。  
使い勝手は悪くないので無料の範囲内でどこまでやれるかとワークフローの省力化（ボタン一発で記事生成とか）がどこまで進められるかかガギっぽい。
