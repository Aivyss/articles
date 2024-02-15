# 【Linux】 anyenvとxxxenv
私の個人のパソコンも業務用のパソコンもjavaやgo、nodeなど様々な言語をインストールし、開発をしている。ただ最近、同じ言語でも複数のバージョンが必要になり、バージョン変更をする際に`.bashrc`毎回いじることが面倒くさくなった。また、使う言語の数も増えてしまい、なんとかまとめて管理したいという希望ができた。

会社のドキュメントを読んだら、`anyenv`という良いやつがあったのでこれを使い、楽になったので一回まとめたいと思う。

# きっかけ

使ったきっかけは上に記述した通り会社の人々が使っていた。あと前回の会社たちはjavaとkotlinがメインで他言語はあまり触らなかったため、intelliJでなんとかできた。intelliJが強すぎるけど、全ての言語を対応することは難しいし、intelliJでインストールしたsdkって勝手にルートを決めてるし、intelliJの外側では使いにくかったから`anyenv`がいいと思った。

# 何をするものか？

`anyenv`自体は`goenv`、`jenv`、`nodenv`などを総合的に管理してくれるコマンドである。実際`anyenv`がなくても`brew install goenv`などで、個別の`XXXenv`は使えるようだが、`anyenv`で管理した方がいい気がする。

# 残念なところ

homebrewで`XXXenv`をインストールしたら、`.bashrc`をいじらなくても使えるが、`anyenv`でインストールすると`.bashrc`や`.zshrc`をいじらないといけない・・・

# `anyenv`のインストール

## mac

- ターミナルにて`brew install anyenv`し、`anyenv`をインストールする
- shellのrcファイルに`eval "$(anyenv init -)”`を追加する。

## linux (`git`が必要 )

- `git clone https://github.com/anyenv/anyenv ~/.anyenv`

# 言語の環境を構築する

- `anyenv`コマンドが使えるようになったら、言語の環境を構築することができる。私の場合、`goenv`、`rbenv`、`jenv`、`nodenv`、`pyenv`をインストールした。

```bash
anyenv install goenv
anyenv install rbenv
anyenv install jenv
anyenv install nodenv
```

上のようにインストールしたら、rcファイルを修正する必要がある・・・
```bash
# anyenv
eval "$(anyenv init -)"

# goenv
export GO_HOME="$HOME/.anyenv/envs/goenv"
export PATH="$GO_HOME/bin:$PATH"
eval "$(goenv init -)"

# tfenv
export TF_HOME =$HOME/.anyenv/envs/tfenv
export PATH=$TF_HOME/bin:$PATH
eval "$(tfenv init -)"

# jenv
export JAVA_HOME=$HOME/.anyenv/envs/jenv
export PATH=$JAVA_HOME/bin:$PATH
eval "$(jenv init -)"

# pyenv
export PYTHON_HOME="$HOME/.anyenv/envs/pyenv"
export PATH="$PYTHON_HOME:bin:$PATH"
eval "$(pyenv init -)"

# rbenv
RUBY_HOME=$HOME/.anyenv/envs/rbenv
export PATH=$RUBY_HOME/bin:$PATH
eval "$(rbenv init -)"
```