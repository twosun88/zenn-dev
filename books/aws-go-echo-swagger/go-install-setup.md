---
title: "Goのインストールとセットアップ"
free: false
---

このページではGoのインストールとセットアップを行います。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. [https://go.dev/dl/](https://go.dev/dl/) からGoをダウンロードしてインストールする
2. インストール後のバージョン確認（バージョンが表示されればOK）
```
$ go version
go version go1.15.15 darwin/amd64
```
3. `golarn`ディレクトリを作成し、その中に`main.go`を作成し、下記を記述
```go
package main

import (
  "fmt"
)

func main() {
  fmt.Println("Hello World")
}
```
5. `golarn`ディレクトリに移動し`main.go`を実行する（Hello World）が出力されればOK
```
# ディレクトリを移動する
$ cd ~/golarn/

# main.goを実行する
$ go run main.go
```
6. go.modを作成する（example-golarnは変更可）
```
$ go mod init example-golarn/
```
7. go mod init ~ を実行後に **「go: to add module requirements and sums:」** などのメッセージが表示されたら **go mod tidy** を実行
```
$ go mod tidy
```
:::
<!-- // Step -->
## Goのインストール

まずはご自身のPCにGoをインストールしましょう。

Goはコマンドを使わずとも下記のサイトからファイルをダウンロードしてインストールすることができます。Windows, Mac, Linuxなどご自身のOSに合ったファイルをダウンロードして下さい。

なお、Goのバージョンは1.15以降を選択してください。

:::message
Goのダウンロード：[https://go.dev/dl/](https://go.dev/dl/)
:::

ダウンロード後は解凍したファイルを実行してインストーラーを進めていきます。

インストーラー完了後はターミナルなどで下記のコマンドを入力してみて下さい。
バージョンが表示されていればGoのインストールは完了です。
```
$ go version
go version go1.15.15 darwin/amd64
```

## セットアップ
次に、この先で使用する開発用ディレクトリを作成して簡単なプログラムを動かしてみます。
ディレクトリの作成場所はユーザーのホームディレクトリの配下であればどこでも構いません。
今回はホームフォルダ直下に`golean`ディレクトリを作成し、今後はこのディレクトリ内で作業していきます。

では、簡単なプログラムを書いてGoを動かしてみましょう。
`golean`ディレクトリの中に```main.go```ファイルを作ります。
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     └─ main.go
```
`main.go`を開いてお使いのエディタで下記を記述します。
```go:main.go
package main

import (
  "fmt"
)

func main() {
  fmt.Println("Hello World")
}
```
単純に **"Hello World"** を出力するだけのプログラムです。
では、実行してみましょう。
```
# ディレクトリを移動する
$ cd ~/golarn/

# main.goを実行する
$ go run main.go
```

下記のようにコマンドラインに **"Hello World"** と表示されればOKです。
```
Hello World
```

これでGoの動作チェックは完了です。

:::message alert
今後は今作成した`golarn`ディレクトリ内でコマンドを実行します。
コマンド入力時にエラーが出た際には、まずはどのディレクトリで実行しているかをご注意ください。
:::


つづいてモジュール管理のため```go.mod```を作成します。
下記をコマンドラインに入力してください。
```
# example-golarnの部分はお好みで変更してもよい。
$ go mod init example-golarn/
```

```go.mod```とゆうファイルが作成され中身は下記のようになっています。
```
module example-golarn

go 1.15
```

これで今後インストールするGORMやEchoなどのモジュールを```go.mod```で管理することができるようになりました。
::: message alert
$ go mod init ~ を実行後に **「go: to add module requirements and sums:」** などのメッセージが表示された場合は、コマンド **go mod tidy** を実行して下さい。
:::

ここまでの```golarn```ディレクトリの中身は下記の構成になります。
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ go.mod
     ├─ go.sum（ 環境によって生成されるため無くてもよい ）
     └─ main.go
```
以上でGoのインストールとセットアップは完了です。

## 次に進む前に。
この先、何かエラーが起きてうまくいかなかった場合はコマンドラインで```go mod tidy```を実行してみて下さい。モジュールの互換性などが原因のエラーなど、様々なエラーを解決できる場合が多々あります。
```go mod tidy``` 必ず覚えておきましょう。

次にGORMを使ってデータベースとの接続、作成などを行います。
