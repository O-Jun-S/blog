---
title: "高校生が予算0円でブログを作成した話 【hugo】"
date: 2022-06-03T12:21:22+09:00
draft: false
---

今日、高校生である僕、Ojunがブログを予算0円で作成した。

他のブログとの差別化と言えば、WordPressを使わずにneovimでmarkdownをゴリゴリに書いている点だろう。なぜWordPressを使わなかったかというと、レンタルサーバーは面倒かつ、自分の家に物理的なサーバーがないためだ。~~HTMLとかCSSとか書くのめんどくさいし~~

この記事では、どのようにブログを立てたのか紹介しようと思う。参考にしてもらえたら幸いだ。

# 使用技術
静的サイトジェネレータ(SSG)として[hugo framework](https://gohugo.io/)を採用。デプロイには[netlify](https://app.netlify.com/)を用いた。どうやらhugo frameworkは爆速らしい。netlifyは基本無料で使えるサービスである。ドメインはfreenomで無料で取り、SSL化まで無料でできてしまった。

# 環境
Windows10上で、WSLのUbuntuを使用している。

# hugoを使って静的webサイトを作成
まず、hugoをインストールする。もちろんパッケージマネージャーは好きなものを使ってよい。僕はaptを使う。

```
$ sudo apt install hugo
```

次に、webサイトのひな形を作る。

```
$ hugo new site (サイト名)
```

によって作成できる。

そして、themeを入れる。hugoには多彩なthemeがあるが、僕は[terminal](https://github.com/panr/hugo-theme-terminal)を用いた。
theme一覧は[ここ](https://themes.gohugo.io/)にある。

```
$ git init # まだgitリポジトリでないなら実行
$ git submodule add -f https://github.com/panr/hugo-theme-terminal.git themes/terminal
```

次にconfigを設定する。プロジェクトルートにconfig.tomlファイルがあるだろう。僕はgithubのREADMEを見ながらconfigを設定した。個人個人、選んだthemeが異なるので、それぞれのREADMEやhugoのdocsを見てもらいたい。

ここで、一つ記事を作成してみよう。

```
$ hugo new posts/first-post.md
```

すると、次のようなファイルが作成される。

```
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
```

ここで、記事を書いてみよう。もちろん書き方は[markdown](http://www.markdown.jp/what-is-markdown/)である。記事を書いたらdraftをfalseにしてほしい。
次に、markdownからHTMLを作成しよう。以下のコマンド一発でできる。

```
$ hugo -D
```

最後に、localhostでサーバーを立ててみよう。これも以下のコマンドでできる。

```
$ hugo server -D
$ hugo server -t terminal (themeが僕と同じterminalの場合のみ。正直違いはよく知らない)
```

すると、localhost:1313にサーバーが立っているはずである。ひとまず、SSGを使ってサイトが建てられた。

# netlifyにデプロイ
まず、[netlify](https://app.netlify.com/)に登録した。正直、githubリポジトリを登録するだけで簡単にデプロイできるので、説明はあまり要らないだろう。

# ドメイン取得
ドメインは[freenom](https://www.freenom.com/ja/index.html?lang=ja)で無料で取得した。freenomは日本語が怪しいが、今まで特に障害はなかった。
