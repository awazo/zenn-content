---
title: "virt-install で GUI を使わないゲスト OS のインストール"
emoji: "🧅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "alpine", "qemukvm", "virtinstall", "ubuntu" ]
published: true
---

# virt-install を使ってゲスト OS をインストールする
**ただし GUI ツール (virt-manager, remote-viewer, etc...) は使わない**

## tl;dr
ここだけでも長いなあ。

### やりたかったこと
* ホスト OS: Alpine Linux v3.15
* qemu-kvm を virsh で操作する
* でもゲスト OS のインストールは virt-install を使う
* でも CLI で完結させる
* ゲスト OS は、Alpine Linux v3.17.0 と Ubuntu Server 22.04.1 を入れてみる

### やったこと

1. 必要パッケージのインストールから libvirtd の開始
    * `# apk add libvirt-daemon virt-install qemu-img qemu-system-x86_64 qemu-modules openrc cdrkit`
        * isoinfo コマンドが必要になるため cdrkit パッケージを入れる
    * `# rc-update add libvirtd` `# rc-update add libvirt-guests`
    * `# echo "uri_default=\"qemu:///system\"" >> /etc/libvirt/libvirt.conf`
    * `# rc-service libvirtd start`

2. ストレージプールの作成
    * `# virsh pool-define-as default dir --target /path/to/vmdisk/dir`
    * `# virsh pool-start default`
    * `# virsh pool-autostart default`

3. ゲスト OS のインストール
    * `# osinfo-query os` で virt-install の --os-variant に指定する値を探す
        * 2023-01-10 現在は alpinelinux3.14 、ubuntu21.04 までしかなかったので、迷ったけど、より汎用そうな linux2020 を指定した

    *  ゲスト OS: Alpine Linux v3.17.0 の場合
        * [alpine linux の VIRTUAL の x86_64 の iso ファイルをダウンロード](https://alpinelinux.org/downloads/)
        * `# virt-install --name test --memory 2048 --vcpus 2 --disk size=2 --cdrom /path/to/alpine-virt-3.17.0-x86_64.iso --os-variant linux2020 --network type=direct,source=eth0,source_mode=bridge,model=virtio --nographics`
        * あとはいつもどおりに alpine のインストールを行う

    * ゲスト OS: Ubuntu Server 22.04.1 の場合
        * [ubuntu server の iso ファイルをダウンロード](https://ubuntu.com/download/server)
        * `# isoinfo -f -i /path/to/ubuntu-22.04.1-live-server-amd64.iso | grep -i -e vmlinuz -e initrd` で次の --location オプションに付加する kernel= と initrd= のパスを確認する
        * `# virt-install --name test --memory 2048 --vcpus 2 --disk size=4 --location /path/to/ubuntu-22.04.1-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd --os-variant linux2020 --network type=direct,source=eth0,source_mode=bridge,model=virtio --nographics --extra-args="console=tty0 console=ttyS0,115200n8"`
        * あとはインストーラに沿ってインストールする

4. そのほか
    * `# virsh autostart test`
    * 最後に試しに作った test を削除した (そのうちちゃんと使う用のを作る)
        * `# virsh domblklist test` で次のコマンドの --storage オプションの値を調べる
        * `# virsh undefine --domain test --storage vda --snapshots-metadata --managed-save`
    * 作ったゲストを一覧するには `# virsh list --all`
    * ホストからゲストに入るには `# virsh console test`

## まずは公式
[alpine の wiki の KVM のページ](https://wiki.alpinelinux.org/wiki/KVM) を見て apk add する。けど、virt-install を使いたかった (virsh create だとゲストの定義を書いた xml ファイルとかを用意するらしく、それは大変そうだった) ので、それも。cdrkit については後述。
ネットワークはデフォルトで NAT になるらしく、ホスト OS が所属している LAN とは別に、ホスト OS とゲスト OS のみが所属するネットワークになるので、ゲストにアクセスするには必ずホストにログインしないとならなくなりそう。
それは面倒だったので、直接ゲストが見える状態にするため、macvtap というのがあるのを知った。[こちら](https://yskwkzhr.blogspot.com/2017/06/using-macvtap-with-kvm-on-debian-gnu-linux.html) が参考になったので、ありがとうございます。
実際に virt-install に渡すオプションは [virt-install の gist](https://gist.github.com/smurugap/163b3e2be7676a46c835339f8ba0710f) から `--network type=direct,source=enp6s0f0,source_mode=bridge,model=virtio` の部分をいただいた。先達が大くて助かります。もちろん `source=enp6s0f0` の部分はホスト OS のネットワークインタフェース名に合わせる必要があるので、`$ ip a` とかして調べたら良い。
alpine の wiki に戻って、rootユーザ以外でもいろいろしたかったら、libvirtグループにユーザを追加してねとのこと。今回の環境では、sudo を apk add していて、普段使いのユーザは wheel グループに入れてあるので、必要なら sudo すればいいやというわけで、libvirt グループには入れてない。
あとは 2 つ rc-update add して、ひとまず alpine wiki から得られる情報はおしまい。

## 準備はできたので、使いかた
使いかたの全体像としては、[こちら](https://endy-tech.hatenablog.jp/entry/kvm_setup_and_libvirt_cli) が参考になった。virt-manager や remote-viewer といった GUI ツールを使っているところは、今回の私の目的とは異なるけれど、/etc/libvirt/libvirt.conf の設定や、pool をまず作る必要があるとか、pool を作るコマンドといった情報は、とても助かりました。あと virsh undefine のオプションも、こちらから貰った。
ただ、やはり GUI 前提なので、書かれているように virt-install の --noautoconsole オプションだけでは、ゲストは動き出しているらしい (virsh list --all で確認) のに、iso ファイルから起動するはずのインストーラが表示されてこない。あたりまえだけども。
::: message
ほかに virsh のオプションとしては、[ubuntu のドキュメント](https://ubuntu.com/server/docs/virtualization-libvirt) も、ちょっと見はした。
:::
あ、でも alpine は --cdrom オプションで iso ファイルを指定して、--nographics オプションを付けるくらいで、すぐにインストールはできた。インストーラというよりもコンソールでログインして setup-alpine コマンドを実行するというやりかただからかな。
--memory オプションの値を 2048 にしているのは、--os-variant linux2020 を指定したら最低でも 2048 にしてね、というメッージが表示されたため。
--vcpus と --disk はてきとう。

## alpine はインストールできるけど ubuntu はできない、の解決
これがなかなか時間がかかった。[redhat のページ](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/virtualization_getting_started_guide/sec-cli-utilities-demonstration) の中盤あたりに、注記として --location オプションと --extra-args オプションを使う方法の記載があって、ようやく少しだけ進展しはじめる。
ちなみに --extra-args の後半は tty0 ではなく ttyS0 でシリアルコンソールを指定しているのでお気をつけて。115200n8 はボーレートとかなんとかそういうアレよ (実際にシリアルコンソールを使った経験が無いのであんまり実感は無い。シリアルケーブルでマイコンと接続とかは、数回くらいはしたけど、その程度なものですみません) 。
でも、↓こうなっちゃう。
```
vmhost:~$ sudo virt-install --name test --memory 2048 --vcpus 2 --disk size=4 --location /path/to/ubuntu-22.04.1-live-server-amd64.iso --os-variant linux2020 --network type=direct,source=eth0,source_mode=bridge,model=virtio --nographics --extra-args="console=tty0 console=ttyS0,115200n8"

Starting install...
ERROR    Couldn't find kernel for install tree.
Domain installation does not appear to have been successful.
If it was, you can restart your domain by running:
  virsh --connect qemu:///system start test
otherwise, please restart your installation.
vmhost:~$ 
```
これには、まったく困り果てていたのだけれど、[こちら](https://ngyuki.hatenablog.com/entry/2020/02/22/004227) が救世主さまであらせられた。
--location で iso ファイルを指定するだけではなく、その後ろに `,kernel=vmlinuzへのパス,initrd=initrd.imgへのパス` を付けると良いらしい。
となれば、`mount -t iso9660 -o loop /path/to/ubuntu-22.04.1-live-server-amd64.iso /mnt/point` で、てきとうなディレクトリにマウントしておいて、find コマンドとかで探してみた。
それが、ubuntu-22.04.1-live-server-amd64.iso では `kernel=casper/vmlinuz,initrd=casper/initrd` だった。のだけれど、これだけだと、isoinfo というファイルまたはディレクトリは無いというエラーが表示された。このとき、まだ cdrkit パッケージをホストにはインストールしていなかったから。
isoinfo がコマンドなのに最初は気づけずにいた (だってエラーメッセージがファイルとか言うから、iso 内にあるはずのファイルか何かかと思ったので、ubuntu はコンソールでゲストに入れられないのかと誤解するところだった) のだけれど、まあググったらコマンドなのが分かったので alpine の該当するパッケージを探してホストにインストールした。それが apk add cdrkit ということ。
で、isoinfo コマンドを使えば、直前でやったように iso ファイルをわざわざマウントしなくても、`# isoinfo -f -i /path/to/file.iso` で iso ファイル内にあるファイルを一覧表示してくれるので、 `#isoinfo -f -i /path/to/file.iso | grep -i -e vmlinuz -e initrd` のようにして簡単に kernel= と initrd= に書くパスが分かる。べんり。
これでようやく、ubuntu のインストーラが表示された。なんか最初に poor な表示のままやるかいって聞かれるけど、もちろんさと回答することにしてる。他の選択をしたらどうなるかは試してない。
ちなみに ubuntu のほうは --disk size=4 にしているのは、インストールの最小要件が [Disk: a minimum of 2.5 gigabytes](https://ubuntu.com/server/docs/installation) となっていたため。size=3 にしても良かったかもしんない。

## おわり
以上で、なんとかどうにか alpine と ubuntu をゲストにインストールすることができた。
他に参考にしたページとしては、[こちら](https://sysguides.com/kvm-guest-os-from-the-command-line/) がとても頼もしい。まさにタイトルどおり How to Install a KVM Guest OS from the Command-Line というわけだ。ここにも書いてある --extra-args の指定で、 `---` で区切る方法があるようなのだけれど、[こちら](https://zenn.dev/satoru_takeuchi/articles/8e377b6d2e2568) でも言及されているようにインストーラの機能らしい。
ただ、私の環境で試してみたところだと、`---` の前後を換えても、また付けても付けなくても、ゲスト ubuntu の /etc/default/grub の GRUB_CMDLINE_LINUX 付近には変化が見られなかった。バージョン違いとかなのかもしれないが、そのあたりは未検証。
最後に余談なんだけど、alpine ってコンテナ内で使うような OS なんだろうかという疑問が少しあって、busybox だったり libc が musl だったりで、まれによくコンテナ内に作ったアプリケーションがうまく動かないことがあり、alpine に合わせてアプリケーションを作るようにするべきってことはさすがに無いんじゃないかなという気がしている。当初はそういう目的で作られたのかもしれないけれど。任意のアプリケーション用の OS としてではなく、vm やコンテナの制御をするホスト側のような決まった用途としてこそ有用そうな気がするのですよ。とくに昨今は debian slim とか distroless とかいうのもあるらしいし (ろくに知らないけど) 。
ということで、自宅のハードウェアに alpine を入れて、その上に vm として alpine や ubuntu を入れてみたかったということでした。ほかの OS もゲストに入れてみようかな、なにが良いかなあ。

