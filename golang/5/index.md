# deferの挙動
## deferについて
書いてあるラインの順番の通りではなく、関数の外部に値をリターンする直前に指定した内容を処理させる予約語(reserved word)

### 簡単な例
```go
func main() {
	fmt.Println("a")
	defer fmt.Println("b")
	defer fmt.Println("c")
	fmt.Println("d")
}
```

```
a
d
c
b
```

## 留意点
変数を共に使う場合、予期せぬ値で実行される恐れがあるため、こちらではその例と理由について説明する
### 例１
```go
func main() {
	testFunc()
}

func testFunc() (s string) {
	defer func(ss string) {
		fmt.Println("ss =", ss)
		fmt.Println("s =", s)
	}(s)

	s = "banana"
	return
}
```

```
(間違いやすいケース)
ss = banana
s = banana
(正しい出力)
ss =  
s = banana
```
### 例２

```go
func main() {
	testFunc()
}

func testFunc() (s string) {
	defer func(ss string) {
		fmt.Println("ss =", ss)
		fmt.Println("s =", s)
	}(s)

	s = "banana"
	return "apple"
}
```

```
(間違いやすいケース)
ss = banana
s = banana
(正しい出力)
ss =  
s = apple
```

### deferの挙動
`defer`は３ステップで動作する
1. `defer`の呼び出し + 関数のパラメータを評価して決める
2. `defer`を実行して関数stackに入れる
3. リターンまたはpanicの後にある関数stack(=`defer`に指定した関数)を実行する
   **※ deferで実行される関数の内部は１の時点で評価されない**

この流れで１例と２例を分析すると、

#### １例の解説
1. `s`が`""`に評価さえる。そのため`ss`も`""`に評価される
2. function stackに`func(ss string) { (関数内部は評価されない領域) }("")`を入れる
3. `return`の後`defer`で指定した関数が実行される
    1. `ss`は1により`""`となる
    2. anonymous function内部の`s`は今の時点で`"banana"`と評価される
```
ss =  
s = banana
```
#### ２例の解説
1. `s`が`""`に評価さえる。そのため`ss`も`""`に評価される
2. function stackに`func(ss string) { (関数内部は評価されない領域) }("")`を入れる
3. `return`の後`defer`で指定した関数が実行される
    1. `ss`は1により`""`となる
    2. anonymous function内部の`s`は今の時点で`"apple"`(`return "apple"`により)と評価される
```
ss =  
s = apple
```