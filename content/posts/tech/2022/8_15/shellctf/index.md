---
title: "SHELLCTF writeup"
tags: ["ctf", "ハッキング"]
keywords: ["reversing", "crypto"]
description: "SHELLCTFのwriteupです。"
cover: "img/solves.png"
date: 2022-08-15T12:13:36+09:00
draft: false
---

## 概要
SHELLCTFというCTFに参加しました。結果は1121位中294位でした。

時間がない中まぁまぁ健闘したんじゃないかなぁと思います。主にreversingを解きましたが、cryptoやforensicsも少し解けました。
この記事には解けた問題の解説を書いていこうと思います。

## reversing
reversingに主に使用したツールはradare2、ghidraです。

### Pulling the strings
なんかアセンブリを解析したらflagっぽいものがあったので、入力したらflagでした...
![assembly](img/food.png)

### Keygen
とりあえずアセンブリを解析しましょう。

![assembly](img/keygen.png)

いろいろ処理がありますが、とりあえず上手くいった場合getString関数を実行するみたいです。ではgetString関数の中身を見てみましょう。

![assembly](img/keygen2.png)

flagが直に書かれていることが分かりました。これを入力すれば正解です。

### warmup
少しプログラムが複雑なのでghidraでデコンパイルしました。

![source](img/warm.png)

local\_38は標準入力です。
長々とlocal\_a8というバイト列が定義されていて、その後ループ処理をしていることが分かりました。
このループ処理によってbVar1がtrueとなったときが正解のようです。ではどのような処理がされているのでしょうか。

```c
local_a8[local_ac] >> 2 == (int)local_38[local_ac]
```

ということは、$0 \le i \lt 27$において、local\_a8のi番目のバイトを右に2つシフトしたものと、標準入力local\_38のi番目の文字が等しければよいということになります。
よって、local\_a8のバイト列をすべて右に2つシフトさせれば正解です。
以下のようなスクリプトを書きました。

```python
lst = [0x1cc, 0x1a0, 0x194, 0x1b0, 0x1b0, 0x18c, 0x1d0, 0x198, 0x1ec, 0x188, 0xc4, 0x1d0, 0x15c, 0x1a4, 0xd4, 0x194, 0x17c, 0xc0, 0x1c0, 0xcc, 0x1c8, 0x104, 0x1d0, 0xc0, 0x1c8, 0x14c, 0x1f4]

print("".join(map(lambda x: chr(x >> 2), lst)))
```

### How to defeat a dragon
アセンブリを見てみましょう。

![assembly](img/dragon.png)

モロにflagがありますね...

{    }の中身が16進数なので、それを文字に戻したらflagです。

### tea
ちょっと難しいです。とりあえずアセンブリを見てみましょうか。

![assembly](img/tea_asm.png)

入力した文字列に以下の3つの操作が施されています。

- addSugar
- addTea
- addMilk

そして、最後のstrainAndServeを見てみると

![assembly2](img/tea_asm2.png)

3つの操作を施した文字列が、"R;crc75ihl\`cNYe\`]m%50gYhugow~34i"という訳のわからん文字列になったら正解ということです。
では、それぞれの操作を逆算すればよいですね。それぞれがどんな操作なのかというと。

- addSugar: 偶数番目(0-indexed)の文字を集めた文字列をA、奇数番目の文字を集めた文字列をBとし、ABという形に並べ替える。
- addTeaその1: 文字列の長さをnとすると、$0 \le i \lt \frac{n}{2}$番目の文字の文字コードに、$\lfloor \frac{i}{2} \times -3 \rfloor$を加算する。
- addTeaその2: 文字列の長さをnとすると、$\frac{n}{2} \le i \lt n$番目の文字の文字コードに、$\lfloor \frac{i}{6} \rfloor$を加算する。
- addMilk: 文字列のはじめから、5という文字が出てくる直前までをAとする。5から、Rという文字が出てくる直前までをBとする。Rから最後までをCとする。そして、それらをCABという順に並び変える。

以上の操作の逆をすれば良いことになります。addMilkに関しては、どこでCABという風に切ればよいのか定まらないので、少しbruteforce的な手法を取りました。
以下のスクリプトを書いてこの問題が解けました。

```python
length = 32

def removesugar(string):
    first = string[:int(length/2)]
    second = string[int(length/2):]

    result = ""

    for i in range(length):
        if(i%2 == 0):
            result += second[int(i/2)]

        else:
            result += first[int(i/2)]

    return result

def removetea(string):
    result = ""
    for i in range(int(length/2)):
        result += chr(ord(string[i]) + int(i / 2) * 3)

    for i in range(int(length/2), length):
        result += chr(ord(string[i]) - int(i/6))

    return result


def removemilk(string, partition, five):
    first = string[:partition]
    second = string[partition:five]
    third = string[five:]
    result = second + third + first
    return result


cipher = "R;crc75ihl`cNYe`]m%50gYhugow~34i"
for i in range(length):
    if cipher[i] == "5":
        five = i

        for partition in range(1, five+1):
            ans = removesugar(removetea(removemilk(cipher, partition, five)))
            if ans[:5] == "shell":
                print(ans)

```

