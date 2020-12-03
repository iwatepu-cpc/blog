# 前提環境  
[HUGO](https://gohugo.io/)が必要．

## macOSの場合
```
$ brew install hugo
```

# 記事の追加のしかた-書き方

```sh
# 記事MDを生成
$ hugo new posts/<記事のファイル名>.md

# 記事MDを編集(好きなエディタで)
$ code content/posts/<記事のファイル名>.md

# 記事MDからHTML生成
$ hugo

```

なお、記事のヘッダーを以下のように編集してください。  
tagsは必要に応じて追加してください。


```md
---
title: "記事名"
date: 2020-05-07T11:39:39+09:00
author: 筆者名
tags: ["記事の", "タグ", "を", "自由に入力"]
draft: false
---

```

画像は`content/images/path to image`として置いてください。
以下のように記述すると表示されます。

```md
![alt](/blog/images/path to image)
```

# 記事の表示確認
自分が書いた記事がWebでどう表示されるか確認したい．  
```
$ git submodule update --init --recursive
$ hugo server -D
```

上記にコマンドをうつと下記のように表示されると思う．  
```
...
Web Server is available at http://localhost:1313/blog/ (bind address 127.0.0.1)
```

この状態で，ブラウザで表示されてるURLにアクセスすればOK．  
```
http://localhost:1313/blog
```

# デプロイ(記事の公開)
生成後は`git add`, `git commit`, `git push`してデプロイしましょう。  
反映には数分時間が掛かることがあります。
