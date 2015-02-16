---
layout: post
title: "GithubPages独自ドメイン設定"
date: 2015-02-13 20:40:46 +0900
comments: true
categories: [octpress,github,Domain]
---
###ドメイン取得
まずはお名前.comでドメイン登録 ▶︎
[お名前.com](http://www.onamae.com)

今回はmodernevolve.meを取得

###DNSの設定
* お名前.comでドメインタブのDNS関連機能の設定を選択

* CNAMEに**tsuetsugu.github.io**を設定


###CNAMEファイル作成
sourceディレクトリに移動
```
cd source 
```

ドメインを設定してCNAMEファイル作成
```
echo www.modernevolve.me > CNAME
```

設定はこれで完了

[ブログ](http://www.modernevolve.me/)で接続できたら完了。

初めてカスタムドメインを取得して、設定を行えたので、
目標の一つは完了・・・。
