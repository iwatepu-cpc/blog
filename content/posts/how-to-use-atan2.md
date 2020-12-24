---
title: "atan2ってどう使うの？"
author: "tomatoaiu"
date: 2020-12-24T13:44:13+09:00
tags: ["岩手", "岩手県立大学", "プログラミングサークル", "競技プログラミングサークル"]
draft: false
summary: "
  atan2をどう使うかについて触れます．
  "
---

# atan2って何？

JavaScriptには`Math`という組み込みオブジェクトの`atan2()`という関数が存在している。

この`Math.atan2(y, x)`という関数の戻り値は、MDN[1]によると

> 点 (0, 0) から点 (x,y) までの半直線と、正の x 軸の間の (−𝜋<𝑥<𝜋内の) ラジアン単位の角度

らしい。  
  
**？**  
  
そもそも、atan2という言葉がいまいち分からない。  
意味について調べるとWikipedia[2]より

> 二つの引数を取るアークタンジェント

ということらしい。

## アークタンジェントって何？

簡単に言うとタンジェントの逆関数。範囲は、`−𝜋/2<𝑥<𝜋/2`。  
理系大学生の数学駆け込み寺[3]によると

> 𝑡𝑎𝑛𝑥  は 𝑥 という角度の直角三角形において、底辺と対辺の２辺の比を表していますが、𝑎𝑟𝑐𝑡𝑎𝑛𝑥 はこの２辺の比に対しての角度 𝑥 を考えています。

とわかりやすく説明されている。つまりアークタンジェントは角度を求めることができる！

## atan と atan2 の違いって何？

よし、これでatan2を使えるようになった。と思って調べるともう一つ似たような関数が出てくる。それが`atan`。MDN[4]によると

> Math.atan() 関数は、引数として与えた数値のアークタンジェントをラジアン単位で返します。

らしい。範囲は、`−𝜋/2<𝑥<𝜋/2`。  
  
---
|  |atan(x)|atan2(y, x)|
|:---|:---|:---|
|引数|(数値)|(点のy座標, 点のx座標)|
|戻り値|与えられた数値のアークタンジェント (ラジアン単位) |点(0, 0)から点(x,y)までの半直線と、正のx軸の間の(−𝜋<𝑥<𝜋内の)ラジアン単位の角度|
|範囲|−𝜋/2<𝑥<𝜋/2|−𝜋<𝑥<𝜋|

---

したがって、atanの場合には、タンジェントを表す数値が判明している場合に利用でき、atan2の場合は点のx座標, 点のy座標が判明している場合に利用できる。

## atan2ってどんなところで使えるの？

例えば、3次元空間内で自分から見て相手がある角度の範囲内にいるときに、何か特別なアクションをさせる場合に使える。FPSでいうとショットガンの角度の有効範囲的な。  
  
具体的にどう使うかというと、自分から見て前後をy座標として左右をx座標とする。自分の座標をoとする。自分は相手が`𝜋/3<𝑥<2𝜋/3`にいるときにショットガンを撃つと相手に当たようにする。


```javascript
const enableShotgun = (y, x) => {
  const enemyPosition = Math.atan2(y, x)
  if(Math.PI / 3 < enemyPosition && enemyPosition < 2 * Math.PI / 3) {
    return true;
  } else {
    return false;
  }
}
enableShotgun(4, 2); // true
enableShotgun(0, 3); // false
```

## おわりに

中々みかけない、atan2について触れてみた。これでみんなもatan2マスターだ！  
  
この記事は[岩手県立大学 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ipu)の24日目です。

## 参考文献

- [1] Math.atan2() - JavaScript | MDN, [https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/atan2](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/atan2)
- [2] atan2 - Wikipedia, [https://ja.wikipedia.org/wiki/Atan2](https://ja.wikipedia.org/wiki/Atan2)
- [3] たったの1分でわかる！アークタンジェントの基本【微分・積分】 | 理系大学生の数学駆け込み寺, [https://univ-study.net/arctan/](https://univ-study.net/arctan/)
- [4] Math.atan() - JavaScript | MDN, [https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/atan](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/atan)

