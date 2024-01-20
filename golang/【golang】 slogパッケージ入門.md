# 【golang】 slogパッケージ入門

# slogパッケージとは？

Java、Kotlinで開発をしてきた私には、ログというのであれば、実装したものは何か関係なく`slf4j` interfaceによって開発していた。

ただし、golangはdefactoと言えるほどのログパッケージがない・・・なので社内でも`uber-go/zap`を使ったりするが、担当したところではslogが使われていた。

これを導入した人になぜ別のシステムと違うやつを使ったのかを聞いてみたら、v1.21から`log/slog`はgolangの標準ライブラリになる予定だったからだった(https://go.dev/blog/slog)。

まだ私には何が良いのか分からないが、とにかくそうなっていたので、こちらのパッケージをいじってみたことを語る。

## v1.21以前のバージョンの場合

`go get golang.org/x/exp/slog`を実行してimportする

# log/slogのログレベル

確認したら、slogには４つのログレベルが存在した。

- DEBUG (`slog.LevelDebug`)
- INFO (`slog.LevelInfo`)
- WARN (`slog.LevelWarn`)
- ERROR (`slog.LevelError`)

普段は TRACE/INFO/WARN/ERROR/FATAL５段階あるはずだが、何故か少ない。まぁ、確かに自分も今まで開発した時、この四つしか使ったなかったのですが、不思議でした。でもこれで限られるわけではなさそう。よければ定義することも可能である。

# Default Loggerの存在

slogはloggerを生成しなくても、default loggerがあり、何もしなくてもこのような関数を使うだけで、作動する。

```go
slog.Debug
slog.DebugContext
slog.Info
slog.InfoContext
slog.Warn
slog.WarnContext
slog.Error
slog.ErrorContext
```

しかし、以下の設定になるので注意

- property format
- log level: INFO

```go
2024/01/20 13:45:10 INFO debug-test key=value aa=1
```

# Loggerの生成

ログは個人的にjsonのフォーマットが良いと考えてるので、loggerを生成し、default loggerを変えて使っている。

```go
func main() {
	slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelDebug,
	})))
	slog.Info("info-test", slog.String("key", "value"), slog.Int("aa", 1))
}
```

`slog.SetDefault`: default loggerを変える。

`slog.NewJSONHandler`: json formatでログを記録するhandlerを生成

`slog.New`: loggerを生成する

`slog.HandlerOptions`:  loggerのhandlerのオプション

# `slog.HandlerOptions`

```go
func main() {
	slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		// ログを実行したソースコードのpathを記録する attribute key => source
		AddSource: true,
		Level:     slog.LevelDebug,
		ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
			if a.Key == "replaceAttr" {
				a.Value = slog.StringValue("replaced value")
			}

			return a
		},
	})))

	slog.Info(
		"info-test",
		slog.String("key", "value"),
		slog.Int("aa", 1),
		slog.String("source", "test"),
		slog.String("replaceAttr", "original value"),
	)
}
```

```json
{"time":"2024-01-20T14:05:22.676612+09:00","level":"INFO","source":{"function":"main.main","file":"/Users/h.lee/go/testProject/src/25_slog/main.go","line":23},"msg":"info-test","key":"value","aa":1,"source":"test","replaceAttr":"replaced value"}
```

- fields
    - `AddSource`: ログを実行したソースコードのpathを記録する attribute key => `source`
    - `Level`: ログレベルの指定
    - `ReplaceAttribute`: 特定のattribute keyの場合、こちらを実行し、記録内容を変更したりする。別に使われる機能ではないと思う

# slog.Group関数

```go
func main() {
	slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		AddSource: true,
		Level:     slog.LevelDebug,
	})))

	slog.Debug(
		"slog.Attrのgroup化",
		slog.Group("group1", slog.String("key1", "value1")),
		slog.Group("group2", slog.String("key2", "value2"), slog.String("key3", "value3")),
		slog.String("key4", "value4"),
	)
}
```

```json
{"time":"2024-01-20T14:10:10.503008+09:00","level":"DEBUG","source":{"function":"main.main","file":"/Users/h.lee/go/testProject/src/25_slog/main.go","line":30},"msg":"slog.Attrのgroup化","group1":{"key1":"value1"},"group2":{"key:"value2","key3":"value3"},"key4":"value4"}
```

- `slog.Attr`をgroupとして扱う

# Attribute生成関数

- `slog.Time`
- `slog.String`
- `slog.Int`
- `slog.Any`
- `slog.Bool`
- slog.Int64
- slog.Float64
- slog.Uint64