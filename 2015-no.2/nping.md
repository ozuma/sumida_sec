# パケットのすき間に意味を持たせる

## TCPフラグの操作

TCPのSYNやACKなどは、それぞれ1ビットのフラグ領域が割り当てられており、このON/OFFで表現している。これは8bitなのでASCIIコードが入るぞ!
(昔は上位2bitのCRWとECEが無かったので、文献によっては6bitとなっているものもあり。またさらに上位3bitの予約とNSが1bitあるので、全体では12bitある)

このフラグは、npingコマンドの--flagsオプションで制御できる。

```
# nping --tcp -p 80 --flags ALL ozuma.sakura.ne.jp

Windowsだとダブルクォートでくくらないとダメ?
# nping --tcp -p 80 --flags "syn,ack,rst" ozuma.sakura.ne.jp

ASCIIコード41(アルファベットA)を送信
# nping -c 1 --tcp -p 80 --flags 0x41 ozuma.sakura.ne.jp
```

## IPヘッダのToSフィールド

npingで操作可能だが、ルータなどで書きかえられることがあるので万全ではない

## Windowsのping

`abc...qrstuvw`を繰り返すので、最後の文字を指定すれば以下のように`hello`くらいなら送れるよ

```
ping -n 1 -l 8 192.168.2.66 
ping -n 1 -l 5 192.168.2.66 
ping -n 2 -l 12 192.168.2.66 
ping -n 1 -l 15 192.168.2.66 
```
