---
title: "zenn を始めてみた"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github"]
published: true
---

# zenn-cli と github で zenn を始める

## tl;dr

zenn 楽しそう。とりあえず github と連携させてみておいて、そのうち書くことあったら書いてみよう。
というわけで、zenn にアカウント作成しただけだったけど、初期設定した。ひとまず article の新規作成までできたので、まとめる。
当然、最も詳しいのは [公式ドキュメント](https://zenn.dev/zenn) なので、この記事は私の手元の環境であったことの記録です。
主に参照したドキュメントは以下のもの。

* [zenn: GitHubリポジトリでZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)
* [zenn: Zenn CLIをインストールする](https://zenn.dev/zenn/articles/install-zenn-cli)
* [github: Other authentication methods](https://docs.github.com/en/rest/overview/other-authentication-methods#basic-authentication)
* [github: Creating a personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token#using-a-token-on-the-command-line)
* [zenn: Zenn CLIで記事・本を管理する方法](https://zenn.dev/zenn/articles/zenn-cli-guide)

## github にリポジトリ新規作成

リポジトリ名っていつも困る。
とりあえず zenn-written とかにしてみたけど、そういえば複数のarticleやらbookやらを作成するとなると、リポジトリってどうなるんだろ。複数のリポジトリ？　単一のリポジトリで複数記事を管理できるのかな。
→ (from未来の自分): 単一リポジトリの中にarticleやbookといったディレクトリができるので、その中で複数記事を管理できる。

公式ドキュメントを見ていたら zenn-content という名前のリポジトリを作っていたので、ああ、その方が良い感じする、と思い直して、リポジトリを作り直した。

clone しておいた。まだリポジトリの中身は空。README も LICENSE も追加していない。

zenn と github リポジトリの連携については公式ドキュメント通りで、すんなり完了した。たぶん [「2. ダッシュボードから連携する」の手順](https://zenn.dev/zenn/articles/connect-to-github#2.-%E3%83%80%E3%83%83%E3%82%B7%E3%83%A5%E3%83%9C%E3%83%BC%E3%83%89%E3%81%8B%E3%82%89%E9%80%A3%E6%90%BA%E3%81%99%E3%82%8B) で All repositories ではなく Only select repositories を選ぶ必要があるところが、いちばん間違えそうなので、公式ドキュメントにもちゃんと書いてある。ありがとう公式さん。

## nodejs をインストールして zenn 用リポジトリを初期化

[nodejs 公式サイトのダウンロードページ](https://nodejs.org/ja/download/) から、手元の環境が windows なので、Windows Binary (.zip) の 64-bit 版をダウンロード。
[ドキュメント](https://zenn.dev/zenn/articles/install-zenn-cli#0.-%E4%BA%8B%E5%89%8D%E6%BA%96%E5%82%99) によると v15 は現時点（私が試したのは 2021-01-20）ではうまく動かないから、v13 か v14 を使ってねとのこと。LTS 版が v14.15.4 だったのでダウンロードして展開した。
いつも `C:\bin\` フォルダを作成しているので、そこに置いて PATH 環境変数を設定した。

### `npm init --yes` とか `npm install zenn-cli` とか（1回目）

ところが `npm init --yes` で package.json がリポジトリ内ではなく nodejs をインストールしたフォルダに作成されたり、`npm install zenn-cli` がエラーで中断される。
nodejs をインストールしたフォルダには zenn とか zenn.cmd とか zenn.ps1 とかいうファイルまで作成されて、これは何かとってもおかしいな、という気がしてくる。

とりあえず `npm install zenn-cli` のエラーは libvips.js と warmup.js が無いということだった。よく分からないまま、`npm install libvips warmup` とかしてみたけれど、まあダメだよね。libvipsだと思うけど、古いからサポートしてないとかいうメッセージも出ていたみたい（面倒くさくなってよく見てなかった）。

こりゃダメそうだ、となったけど、ちゃんと原因を調べるのは zenn で記事を書けるようにする目的からすると脇道なので、そっちに行くのはやめておく。そういうことするなら、[zenn-editor](https://github.com/zenn-dev/zenn-editor) にパッチを送る方が良いだろうしね、記事にするより。

### nodejs のひとつ前の LTS 版 v12.20.1 をインストール

で、結局は nodejs のバージョンを下げた。

実はこの過程で、すでに nodejs の v0.11.xx（細かいバージョン番号忘れた）とかいうのがインストール済みだったことに気づいてアンインストールしたりしていた。
v14 でのエラーについては、`node --version` でバージョン確認したうえで実行した結果なので、v0.11.xx の影響ではなさそうと思うけど、実のところは良く分からない。

nodejs の古いバージョンは、ダウンロードページの下の方にある [バージョンの一覧](https://nodejs.org/ja/download/releases/) から、ひとつ前の LTS 版 v12.20.1 にした。node-v12.20.1-win-x64.zip をダウンロード、展開、配置。LTS 版は偶数バージョンなのかな。

### `npm init --yes` とか `npm install zenn-cli` とか（2回目）

こんどはうまくいっている様子。package.json もリポジトリのフォルダ内にできるし、zenn-cli のインストールも完走した。

ドキュメントに従って、`npx zenn init` もうまくいき、`npx zenn preview` でうまくブラウザからプレビューも見れた。うれしい。

## github の personal access token を作成

さて、初期化が済んだようなので、ここでひとまず git に commit 作っておこう。そして github に push だ。

push できなかった。Authentication failed とのことだけど、いや何度も試したけど正しい ID とパスワードだよ。なぜだ。

どうやら MFA を設定していると personal access token なるものが必要になるらしい。そういや設定したな MFA とかいうやつ。
ユーザアイコンのところのメニューから settings で Developer settings から Personal access tokens で Generate new token ボタンから新規作成した。このとき表示されるトークンは、もう二度と表示しないからな、と言われるので、きっちり記憶しておく。というのは無理だからメモっておく。

で、ここからどうするんだろう、というのが web 検索でよく分からなかったけど、github のその Personal access tokens のページ下部にドキュメントのリンクがあって、それでなんとかなった。いやあ、ここでなんだか時間をくってしまった。このトークンをどこでどう使えばいいのか分からなくて。

[github の Other authentication methods ドキュメントの下の方](https://docs.github.com/en/rest/overview/other-authentication-methods#working-with-two-factor-authentication) にある Working with two-factor authentication に記載のリンクから [github の Creating a personal access token ドキュメントの最後の手順](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token#using-a-token-on-the-command-line) で `Password: your_token` の記載を見つけたときは、そこかよ、と思った。パスワードにさっき作ったトークンを入れれば良いのね。

GitBash から `git push -u origin main` すると、まず github へのログインダイアログが出るので、ここはトークンではなく、いつものログインの ID とパスワードを入れて、そのあとコンソールで ID を要求されるのでこれは github の ID を入れる。そのあとにパスワードを入れるダイアログが出るので、ここでメモったトークンを入れると、push できた。やれやれ。

## `npx zenn new:article --slug zenn-first-settings` でこの記事を作成

実は slug がなんのことか知らなかったので、web 検索してナメクジの画像をいっぱい見た。うわあ。
英単語としてはナメクジとか、のろのろした人とか、怠け者とか、そういうイメージらしい。銃弾、偽金、パンチ、とかでもあるらしい。なんかあんまり意味が一定しないな。活字で空白に使うやつとかも slug らしい。なんか小さい部品とか、たいして役に立たないような、あんまり意味のないものとか、そういうのを指す単語なのかな。外国語は不思議だ。
動詞だと、猛打するとか、ぶん殴るとかで使うらしい。ナメクジするって意味はないようで良かった。何もしていないことをしているような意味でも動詞で使えるらしい。

余談が過ぎた。url slug で検索しなおして、どうやら url の一部分になるもののことらしいと理解した。
というわけで、preview しながらこの記事を書いて、ここまできた。preview 楽しい。
書くのに飽きたら、読んで直す作業がすぐできて、直しているうちに次に何を書くか考えられて続きが書ける、のループ。良い。書いたらすぐ反映されるし。

さて、この続きは md ファイルの冒頭に自動生成された published が今は false になっているので、これを true にして commit して push すると zenn にデプロイされるらしい。どうなるかな。To Be Continued...

