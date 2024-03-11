# 【AWS】アカウントを跨いてリソースにアクセスする方法：Assume Role
AのアカウントのAWSリソースからBのアカウントのリソースにアクセスする場合がありました。AWS初心者みたいなレベルでありながら、こういうシステムを構築するって経験したこともありませんでしたので、困った経験を記載します。

# 状況
私が直面したケースは以下でした。

## 状況１

- アカウントA：ECSコンテナ
- アカウントB：SQS
- AのECSリソースがBのSQSにアクセス

## 状況２

- アカウントA：EC2
- アカウントB：SQS
- AのEC2がBのSQSにアクセス

１も２も実は対応の方法は同じです。ネットで検索したら結構出ると思いますが、Assume Roleという機能を使います。

# Assume Roleってなに？

![Assume Role](/stocks/images/aws_assume_role.png)

これが一番わかりやすい図だと思います。

AがBのロールを借りて、Bのリソースにアクセスできる権限を得ることだという認識です。正直表現がダサいですが、ロールAが一時的にロールBに変身するイメージが正しいと思います。もちろん得た権限も該当するリソースの権限がない場合、無駄なことですね。

# 実装方法

1. リソースにアクセスをしようとする側のロールにはポリシーを適用する。
2. アクセスされるリソースを持ってる側のロールにはTrust Relationshipを修正する。

## ポリシー作成（アカウントA側のロールAに）

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"sts:AssumeRole"
			],
			"Resource": [
				"arn:aws:iam::${ACCOUNT_B}:role/${ロールB}"
			]
		}
	]
}
```

## Trust Relationship作成 (アカウントB側のロールBに)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com",
        "AWS": "arn:aws:iam::${ACCOUNT_A}:role/${ロールA}"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

# まとめ

経験が浅いので、こういうことを経験して少しでもAWSのロールとポリシーについてわかるようになりました。初心者にとても有益な経験でした。

ただし、こういう場合ってどうしようもないケースが多いかなという認識。できる限りこの状況を作らない方がベストかな・・・と思いました。