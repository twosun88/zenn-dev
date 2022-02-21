---
title: "Goのインストールとセットアップ"
free: false
---

## Goのインストール

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. [https://go.dev/dl/](https://go.dev/dl/) からGoをダウンロードしてインストールする
2. インストール後のバージョン確認（バージョンが表示されればOK）
```
$ go version
go version go1.15.15 darwin/amd64
```
:::
<!-- Step -->

まずはご自身のPCにGoをインストールし動かしてみましょう。

Goはコマンドを使わずとも下記のサイトからファイルをダウンロードしてインストールすることができます。Windows, Mac, Linuxなどご自身のOSに合ったファイルをダウンロードして下さい。

なお、Goのバージョンは1.15以降を選択してください。

:::message
Goのダウンロード：[https://go.dev/dl/](https://go.dev/dl/)
:::

ダウンロード後は解凍したファイルを実行して案内通りにインストールを進めていけばOKです。

インストールが完了後はターミナルなどで下記のコマンドを入力しバージョンが表示されていればインストールは正常に完了しています。
```
$ go version
go version go1.15.15 darwin/amd64
```

## セットアップ
<!-- Step -->
:::details 手順だけ見たい方はこちら

1. 【golarn】ディレクトリを作成し移動する
```
$ cd ~/golarn/
```

2. 【golarn】内に【main.go】を作成し下記を記述
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello World")
}
```

3. main.goを実行する（Hello World）が出力されればOK
```
$ go run main.go
Hello World
```
4. go.modを作成する（example-golarnは変更可）
```
$ go mod init example-golarn/
```
5. go mod init ~ を実行後に **「go: to add module requirements and sums:」** などのメッセージが表示されたら **go mod tidy** を実行
```
$ go mod tidy
```

:::
<!-- // Step -->

次に作業用ディレクトリを作成して簡単なプログラムを動かしてみます。
ディレクトリの作成場所はユーザーのホームディレクトリの配下であればどこでも構いません。
今回はホームフォルダ直下に【golean】ディレクトリを作成し、その中に【main.go】ファイルを作ります。
```
~/user/
  └──golean/
      └── main.go
```
【golarn】ディレクトリにcdコマンドで移動しておきます。
```
# ディレクトリを移動する
$ cd ~/golarn/
```
:::message alert
今後は今作成した【golarn】ディレクトリ内でコマンドを実行します。
コマンド入力時にエラーが出た際には、まずはどのディレクトリで実行しているかをご注意ください。
:::
一度、Goを動かして動作チェックしてみます。
【main.go】を開いてお使いのエディタで下記を記述します。
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello World")
}
```
単純に **"Hello World"** を出力するだけのプログラムです。
実行してみましょう。
```
# main.goを実行する
$ go run main.go
```

下記のようにコマンドラインに **"Hello World"** と表示されればOKです。
```
Hello World
```

これでGoの動作チェックは完了です。

つづいてモジュール管理のため【go.mod】を作成します。
下記をコマンドラインに入力してください。
```
# example-golarnの部分はお好みで変更してもよい。
$ go mod init example-golarn/
```

【go.mod】とゆうファイルが作成され中身は下記のようになっています。
```
module example-golarn

go 1.15
```

これで今後インストールするGORMやEchoなどのモジュールを【go.mod】で管理することができるようになりました。
::: message alert
$ go mod init ~ を実行後に **「go: to add module requirements and sums:」** などのメッセージが表示された場合は、コマンド **go mod tidy** を実行して下さい。
:::

ここまでの【golarn】の中身は下記の構成になります。
```
~/user/
  └──golean/
      ├── go.mod
      ├── go.sum（環境によって生成されるため今は無くてもよい）
      └── main.go
```
以上でGoのインストールとセットアップは完了です。

次にGORMを使ってデータベースとの接続、作成などを行います。
