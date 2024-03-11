# golangのAOPとProxy Pattern
今回は失敗したことについて話します。

私は前からgolangで少しやりたいことがありました。
1. Proxy Pattern
2. AOP
両方ともWEB開発に意外と使われるパターンですし、結局似たようなもんです。JavaとKotlinは素晴らしいライブラリがありましたので楽でしたが、golangはあまりなさそうです・・・

# library search
- https://github.com/gogap/aop
- https://github.com/AkihiroSuda/aspectgo

上のrepositorygが私が調べた奴らですが、ちょっと使うには曖昧かなと思いました。もう管理されてないプロジェクトであることも気になりますし、Javaと比べたら貧弱な機能でした。

もう構造体を実装する時点でAOPに関するロジックを共に入れるってAOPの長所があるの？ってなったlibraryもありますし、もう一個は私が求めたやつでしたが、8年前から管理されてないですね。

Proxy Patternができるようにしますって言ってるものも、色々微妙でした。JavaのJPA、Hibernateみたいにはできないのか・・・

# 作るか・・・
自分が欲しかった機能のライブラリを作るには以下が必要な認識でした。
1. 既存のメソッドや関数を裏でいじつこと
2. ライブラリを利用する開発者は自分が実装した構造体を使っていると思うが実際はProxyを使うようにする

この要件を想定して、自分で作ってみようともしました。

## 1. 既存のメソッドや関数を裏でいじつこと
```go
testFunc := func() string {
    return "original"
}

testFunc = func() string {
    return "modified"
}

// ---

type StructA struct {
    TestFunc func() string
}
```
上のようにglobal変数や構造体のフィルドにしたら、言語の限界を超えてできます。しかし、普段そんなことしないから、解決にはならないです。

あと、reflectを利用する方法も色々調査しました。ある程度可能性が見えたのですが、golangが元の関数やメソッドをいじることを止めてます。完全に不可能ではなかったですが、assemblyまで掘り出していじらないといけなさそうです。

- https://bou.ke/blog/monkey-patching-in-go/
- https://github.com/bouk/monkey

マジかよ・・・これをやった人はすごいですがそこまで・・・？少し目的がズレていると考えました。

dynamic typeの言語じゃないと中々面倒なことになりそうでした。Javaはある意味ですごいな・・・

## 2. ライブラリを利用する開発者は自分が実装した構造体を使っていると思うが実際はProxyを使うようにする
これはgolangの言語の特徴で難しかったです。golangは継承がない言語です。似たようなことはできますが、それは厳密に言うとduck typingです。

あと、golangは構造体に対して、is-aはできなく、has-aだけサポートする言語です。

なので、proxy構造体を作ったり、返したりすることはできませんでした。interfaceだと簡単な話ですが、構造体は難しそう。

# 結論
無理かな・・・w