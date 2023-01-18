---
title: "設定済みのリポジトリで zenn を再開"
emoji: "🔁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "ubuntu", "nodejs"]
published: true
---

# 以前に連携を設定済みの github リポジトリを使って、新環境で zenn する

## tl;dr
1. nodejs のインストール
    * ubuntu (lubuntu) の場合
        1. `$ sudo apt install npm -y`
        2. `$ sudo npm install n -g`
        3. `$ sudo n lts`
2. git clone 設定済みリポジトリ
3. clone したディレクトリ (package.json のある場所) で
    1. `$ sudo npm install zenn-cli`
    2. `$ sudo npm install zenn-cli@latest` # いらないかも
4. `$ npx zenn preview &`
5. ブラウザで http://localhost:8000/ を開く
6. あとは `$ npx zenn new:article --slug お好きに` みたいな新規記事作ったりなどする

## ごぶさたしてます
なんと [前に書いたもの](https://zenn.dev/awazo/articles/zenn-first-settings) から、もう 2 年が経とうとしていて、驚きでござい。
この間に引越すなどして、Lubuntu を使うようになり、以前に zenn の環境を作った PC はまだあるものの、新環境でも zenn できるようにしようとした話を書く。
別のことを書こうとして準備していたのだけれど、案外と nodejs のインストールに手間がかかったので、そのへんの話。なので、zenn というよりは Lubuntu (ubuntu) に nodejs をインストールする内容がメインになりそう。
しかし、前回も nodejs のインストールでひと手間かかってるんだよなあ。難儀なことよ。

## apt install nodejs
とりあえず nodejs をインストールしようと、単純にそのままやってみて、これは無事に完了した。
じゃあ `npm install zenn-cli` しようかとしたら、npm コマンド無いよとのこと。考えてみれば、それはそのはず。なので `apt install npm` もしてみた。試せていないけど、`apt install nodejs` 無しで `apt install npm` だけでも依存関係で nodejs もまとめて入れてくれるんじゃないかなという気がしている。たぶん。
さて、使っている Lubuntu (ubuntu) は 22.04.1 で、インストールされた nodejs のバージョンは、なんと驚きの 12 !? !? !? 
12 とか、前回 ( 2 年前だよ) のときもインストールしたし、それどころか、そのとき既にバージョン 14 あったじゃない。まだ 12 なんかい。
それでも ubuntu さんがそう言うなら使えるのかなと `npm install zenn-cli` したら、ちゃんと完了。いけるか、と思いきや、`npx zenn preview` がエラーになって、なんかいっぱいゴチャゴチャ吐き出してくる。アップデートしてね、とのこと。
で、`npm install zenn-cli@latest` にて、やっぱり nodejs のバージョンを 14 以上にしなさいとのことで、動かない。かなしい。

## npm install n
[n コマンドというのがあるらしい](https://qiita.com/cointoss1973/items/c000c4f84ae4b0c166b5) と知って、それを入れて lts の最新にしたら nodejs のバージョンが一気に 18 になった。
いやでも n というコマンド名には、なんか違和感あるけど。この用途のコマンドが、この名前なのか、っていうのが。いや、ありがたいんだけどね機能としては。でもなんか消化不良感があとをひくですます。

## あとはふつうに
[公式](https://zenn.dev/zenn/articles/install-zenn-cli) とか [公式](https://zenn.dev/zenn/articles/zenn-cli-guide) とかにあるものから、必要なところだけすれば良いので、`npm install zenn-cli` または `npm install zenn-cli@latest` して、前回 ( 2 年前) から更新されてないリポジトリを clone してくる。
今度はちゃんと `npm install zenn-cli@latest` も完走して、clone したリポジトリのルートディレクトリ ( articles と books のディレクトリが入っている場所) で `npx zenn preview` することで無事にプレビューを見ることができた。
で、`npx zenn new:article --slug zenn-with-configured-repository` して、これを書いているという次第。
nodejs で手間取らなければ、`npm install zenn-cli` と `npx zenn preview` と `npx zenn new:article --slug xxx` だけで済んだから、こんなの書かなかったのだけれどね。
当初に書こうとしたものは (そのために zenn の環境を作り直したのでした) また後日にでも。

