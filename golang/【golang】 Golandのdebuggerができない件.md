# 【golang】 Golandのdebuggerができない件

# 状況は？
- Machine: Macbook M2 Air
- OS: Ventura 13.4
- go: v1.21.1
- IDE: Goland (Jetbrains)

goで開発する際、私はGolandを使う。理由としてはすごく便利だから・・・。有料なのであまり使いたくない人もいるだろうが、IntelliJを使った時、EclipseやVScodeに比べてdebugger・plugin・gitなどがものすごく楽になった。Golandも一緒だろうって思って使ったが何故かdebuggerが効かない。

M1の自分のパソコンはうまくいったのにM2の社用のパソコンではできなくて、golandバージョンもperpetual licenseで2022の古いやつだし、apple siliconの問題かなと思いましたが、あまりにもそういう人がネットにいなかった。

# 問題点

goはdelveっていうdebugのツールを使うらしい。
https://github.com/go-delve/delve

それをgolandがwrapしてGUI上でdebugを楽にしてくれる。しかし、私にはbreakpointが効かない問題があった。dlvコマンドでやればできないわけでもないが、正直debugをコマンドでやるのは辛いな・・・と思いましたので、golandをなんとなくしたい。

# try 1: IDEの設定を変える。

まずは調査だった。この問題は実はapple siliconのmachineを使う開発者がネットに投稿した内容があって、IDEのpropertiesを修正したらできるっていう話が結構あった。
ネットにあった内容は以下になる。

```bash
go install github.com/go-delve/delve/cmd/dlv
```

としたら、GOPATHに実行可能なdlvがインストールされる。
あとgolandの`help > Edit Custom Properties…`にて以下のパスを追加する。

```bash
dlv.path=/gopath/bin/dlv
```

そしてgolandを再起動してdebugを行えば、良いとことだったが、私はできなかった。しかし、私のM1のやつはこれで解決できたので試しても損ではないはずです。

# try 2: goのPATHを修正する

私は社用の`.bashrc`は整理されてなくて、乱雑になっていた。brewでgoを直接インストールしたり、goenvでgoをインストールしたり、後で`anyenv`がわかりまたそれからの設定があったりして、私ってどのやつを使っていた？になった。

実際以下のコマンドを実施してもgoのバージョンが全く変わらなかった。

```bash
$ goenv global 1.20.7

$ goenv versions
  1.20.7
* 1.21.1 (set by $HOME/.goenv/version) # あれ？
```

それで、直接にインストールしたgoと`goenv`は削除し、`anyenv`のみで管理するように整理した。また、`.bashrc`もあれこれ複雑に書いていたのでgoの周りをリセットして`anyenv`(`goenv`)でのgoだけにしました。

```bash
# goenv
export GOENV=$HOME/.anyenv/envs/goenv
export PATH=$GOENV/bin:$PATH
eval "$(goenv init -)"
```

そうしたら上手く行けるようになった。

# 原因？

正直、原因は分かりづらかった。この問題を解決するために`dlv`コマンドを直接に打ったり、`gdlv`というツールでやってみたりしたけど、その時は別に問題にならなかった。つまりコマンドではbreakpointをかけることに問題がない。なので、根本的に何が問題かはまだ知らない・・・

ただし、`anyenv`で言語のバージョンを管理することにしたら、今まで使ってた言語の周りを整理する必要はあるということはわかった。