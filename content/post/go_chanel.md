+++
date = "2015-10-07T11:30:47+09:00"
draft = false
title = "ゴルーチン間の通信"
author = "shanxia"
categories = ["並列処理"]
+++

# ゴルーチン間の連携方法
「ゴルーチン」（main含む）間で連携するには、「チャネル」と呼ばれる機能を利用します。「チャネル」を利用することで、「ゴルーチン」間で「通信」「同期処理」が可能になります。

## チャネルの作成方法
「チャネル」を使用するには、「チャネル型」の変数を作成し、送信側・受信側ともに、その変数に対して何らかのデータを送受信します。他の型変数と違うのは、送信・受信ができることだけです。送受信は他のプログラミング言語とは一風変わった書式で行います。

```go
// 宣言方法
chan 型 // 送受信可能なチャネル
chan <- 型 // 送信専用チャネル
<- 型 // 受信専用チャネル

// 作成方法
make(chan 型, キャパシティ)
make(chan 型)// キャパシティ0と同じ
```
送信専用・受信専用は、主に関数の引数として受け取る際に使用します。
また、「チャネル」はスライスやマップと同じく参照型のため、作成するには同じくmake関数を使用します。
チャネルに指定するキャパシティは、チャネルにバッファ可能な容量です。このサイズが送信データの上限となります。

## チャネルのクローズ
使用しなくなったチャネルは、「close」関数を使用してクローズします。ただし受信専用チャネルはクローズできません。

```go
// チャネルのクローズ方法
close(チャネル)
```
クローズ済みのチャネルに対して、データを送信するとランタイムパニックになります。
また、クローズ済みのチャネルから受信しようとすると、クローズ前までに送信されたチャネルに残っているデータがなくなるまで受信できます。
受信側で、チャネルがクローズされたかどうかを確認するには、受信操作時に受け取る2番目のbool値が「false」であれば、クローズ済みということになります。サンプルは、後述する「チャネルの使用方法」を参考にしてください。

## チャネルの使用方法
「チャネル」を利用して送受信を行うには、宣言と同じく「<-」演算子を使用します。サンプルコードを以下に記述します。

```go
package main

import (
	"fmt"
)

func main() {
	// int型チャネルの作成
	c := make(chan int)

	// 送信専用チャネルを受け取り、1〜10までの数値を送信する
	go func(s chan<- int) {
		for i := 0; i < 10; i++ {
			s <- i
		}
		close(s)
	}(c)

	for {
		// チャネルからの受信を待機
		val, ok := <-c
		if !ok {
			// チャネルがクローズしたので、終了する
			break
		}
		// 受信したデータを表示
		fmt.Println(val)
	}
}
```
実行すると、次の様な結果になります。
>0
1
2
3
4
5
6
7
8
9

## チャネルのキャパシティと要素数
「チャネル」のキャパシティを取得する場合は、「cap」関数を使用します。また、現在「チャネル」に保存されている要素数を取得する場合は、「len」関数を使用します。

```go
キャパシティ = cap(チャネル)
要素数 = len(チャネル)
```

## チャネルを利用した同期処理
チャネルのキャパシティ容量いっぱいまで、データがバッファ内に残っている場合に、次の送信操作を行うとそこで処理がブロックされ、他のゴルーチンによってデータが受信されてバッファに空きが出るまで待機します。バッファに空きが出ると、すぐに送信操作が終了します。チャネルのキャパシティが0の場合は、常にこのような動きをします。
また、チャネルのバッファがからの時に受信操作を行うと、そこで処理がブロックされ、送信されるまで待機します。バッファにデータが送信されると、受信操作がすぐに終了します。
ゴルーチン間の同期処理は、この仕組みを利用して行います。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// キャパシティ0で、int型チャネルの作成
	c := make(chan int)

	// 負荷のかかる作業（5秒待機）を3回繰り返した後、通知する
	go func(s chan<- int) {
		for i := 0; i < 3; i++ {
			time.Sleep(5 * time.Second)
			fmt.Println(i+1, "回完了")
		}
		// 適当な数値を送信
		s <- 0
	}(c)

	// 受信を待機
	<-c

	fmt.Println("終了")
}
```
これを実行すると、次の結果が表示されます。
>1 回完了
2 回完了
3 回完了
終了

main関数側は、ゴルーチンが送信操作するまで、受信操作で待機しています。

## ゴルーチン間のデータ共有
ゴルーチン間でデータを共有する場合、グローバル変数を使用することもできますが、チャネルを使用することによって、同時アクセスを防ぐことが簡単に実現できます。

```go
package main

import (
	"fmt"
)

// 全ゴルーチン数
const goroutines = 5

func main() {
	// 共有データを保持するチャネル
	counter := make(chan int)
	// 全ゴルーチン終了通知用のチャネル
	end := make(chan bool)

	// 5個のゴルーチンを実行する
	for i := 0; i < goroutines; i++ {
		// 共有データ(counter)を受信し、インクリメントする
		go func(counter chan int) {
			// チャネルから共有データの受信
			val := <-counter
			// +1する
			val++
			fmt.Println("counter = ", val)

			if val == goroutines {
				// 最後のゴルーチンの場合は、終了通知用のチャネルへ送信
				end <- true
			}
			// +1したデータを、他のゴルーチンへ送信
			counter <- val
		}(counter)
	}
	// 初期値をチャネルに送信
	counter <- 0
	// 全ゴルーチンの終了を待機
	<-end
	fmt.Println("終了")
}
```
実行結果は、次の通りです。
>counter =  1
counter =  2
counter =  3
counter =  4
counter =  5
終了

main関数からcounterへデータ送信後、各ゴルーチンで順にデータを処理しています。