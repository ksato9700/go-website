+++
date = "2015-10-06T13:19:24+09:00"
draft = false
title = "スライス2"
author = "shanxia"
categories = ["中級者向け"]
+++

# スライス[後編]
ここからは、スライスの操作について説明していきます。

## スライスへの追加
スライスへ要素を追加する場合は、「append」関数を使用します。追加元には、「スライス」または「要素」を指定します。」append」は戻り値として、追加後のスライスを返します。

```go
// appenの書式1
追加後のスライス = append(追加先のスライス, 追加要素1, 追加要素2, ...)// ...は可変長パラメータを意味します。
// appenの書式2
追加後のスライス = append(追加先のスライス, 追加するスライス, ...)
```
「append」には注意点があります。
「追加先のスライス」が参照している配列に、十分なキャパシティがある場合は、その配列に追加要素は格納され、そこから作成したスライスを返します。しかし参照先の配列に十分なキャパシティがない場合は、別に新しい配列が作成され、それを参照してスライスが返されます。
このように、追加先スライスの参照先の配列によっては、追加元のスライスが参照していた配列と異なる可能性があります。

```go
package main

import (
	"fmt"
)

func main() {
	// スライスを作成
	s1 := []int{1, 2, 3, 4, 5}// スライスの内容を指定し、配列と同時にスライス作成
	fmt.Println("s1=", s1)

	// スライスに要素を追加
	s2 := append(s1, 6, 7)
	fmt.Println("s2=", s2)

	// スライスにスライスを追加
	s3 := append(s1, s2...)
	fmt.Println("s3=", s3)
}
```
実行結果は、次の通り。
>s1= [1 2 3 4 5]
s2= [1 2 3 4 5 6 7]
s3= [1 2 3 4 5 1 2 3 4 5 6 7]

## スライス要素のコピー
スライスの要素を、別のスライスにコピーするには、「copy」関数を使用します。

```go
// copy関数の書式
コピーした要素数 = copy(コピー先スライス, コピー元スライス)
```
コピー先スライスに、コピー元のスライスを上書きします。
copy関数の例は次の通り。

```go
package main

import (
	"fmt"
)

func main() {
	// スライスを作成
	src1 := []int{1, 2}
	dest := []int{97, 98, 99}

	// destへsrc1の内容をすべてコピー
	count := copy(dest, src1)
	fmt.Println("copy count=", count)
	fmt.Println(dest)

	fmt.Println()

	src2 := []int{3}
	// destの3つめのインデックスに、src2をコピー
	count = copy(dest[2:], src2)
	fmt.Println("copy count=", count)
	fmt.Println(dest)
}
```
実行結果は、次の通り。
>copy count= 2
[1 2 99]

>copy count= 1
[1 2 3]

## 可変長パラメータ（可変個引数）へのスライスの渡し方
Goでは、関数の可変長パラメータに渡された値は、呼び出された関数内ではスライスとして認識されます。新しいスライスが作成され、可変長パラメータとして指定されたそれぞれの値が、スライスの要素となります。
つまりスライスを渡すと、それを格納したスライスが渡されてしまいます。これを避けるには、呼び出す際に、スライスに「...」を付けます。

## makeを使用したスライスの作成
スライスは参照型なので、常に実体となる配列が必要です。そのためスライスと配列を同時に作成するための「make」関数があります。

```go
// make関数の書式
作成したスライス = make(スライスの型, 長さ, キャパシティ）
// キャパシティは省略可
作成したスライス = make(スライスの型, 長さ）
```
make関数を利用すると、初期化時にデータがなくても1度でスライスが作成できます。
makeの使用例は次の通りです。

```go
package main

import (
	"fmt"
)

func main() {
	// string型スライスを作成
	s1 := make([]string, 5, 10)
	fmt.Println("len=", len(s1))
	fmt.Println("cap=", cap(s1))

	fmt.Println()
	// キャパシティを省略
	s2 := make([]string, 5)
	fmt.Println("len=", len(s2))
	fmt.Println("cap=", cap(s2))
}

```
実行結果は、次の通り。
>len= 5
cap= 10

>len= 5
cap= 5
