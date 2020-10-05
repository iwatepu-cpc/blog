# 使い方

```sh
# 記事MDを生成
hugo new posts/<記事のファイル名>.md

# 記事MDを編集
code posts/<記事のファイル名>.md

# 記事MDからHTML生成
hugo

```

なお、記事のヘッダーを以下のように編集してください。  
tagsは必要に応じて追加してください。


```md
---
title: "記事名"
date: 2020-05-07T11:39:39+09:00
tags: ["記事の", "タグ", "を", "自由に入力"]
draft: false
---

```

生成後は`git add`, `git commit`, `git push`してデプロイしましょう。  
反映には数分時間が掛かることがあります。
