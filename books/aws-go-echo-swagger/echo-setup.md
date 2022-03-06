---
title: "Echoのインストールとセットアップ"
free: false
---

このページではGoのWebフレームワーク EchoのインストールとAPIの開発に向けたセットアップを行います。
<!-- Step -->
:::details 手順だけ見たい方はこちら
1. ##### Echoをインストールする
```
$ go get github.com/labstack/echo/v4
```
2. ##### `main.go`を編集する
```go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
)

func main() {

  e := echo.New()

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

3. ##### Webサーバーを起動。ブラウザでhttp://localhost:1323にアクセスして表示確認
```
$ go run main.go
```

4. ##### ページを増やしてみる
```diff go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
)

func main() {

  e := echo.New()

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

+  // localhost:1323/foo へGETでアクセスした時の処理
+  e.GET("/foo", func(c echo.Context) error {
+    return c.String(200, "I am Foo")
+  })

+  // localhost:1323/bar へGETでアクセスした時の処理
+  e.GET("/bar", func(c echo.Context) error {
+    return c.String(200, "I am Bar")
+  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

5. ##### Webサーバーを再起動。ブラウザでhttp://localhost:1323/foo, /bar にアクセスして表示確認
```
$ go run main.go
```

6. ##### ミドルウェア（Logger、Recover）の追加
```diff go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
+  "github.com/labstack/echo/v4/middleware"
)

func main() {

  e := echo.New()

+  // ミドルウェア追加
+  e.Use(middleware.Logger())
+  e.Use(middleware.Recover())

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

  // localhost:1323/foo へGETでアクセスした時の処理
  e.GET("/foo", func(c echo.Context) error {
    return c.String(200, "I am Foo")
  })

  // localhost:1323/bar へGETでアクセスした時の処理
  e.GET("/bar", func(c echo.Context) error {
    return c.String(200, "I am Bar")
  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
7. ##### ホットリロードのため「Air」をインストールして、PATHを通す
```
$ go get github.com/cosmtrek/air
```
```
# bashの場は、~/.bashrc または ~/.bash_profileに下記を追記。
# zshの場合は、~/.zshrc または ~/.zprofileに下記を追記。
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

8. ##### Airのバージョン確認と起動
```
# バージョン確認
$ air -v

# 起動
$ air
```

9. ##### ホットリロードの確認
`main.go`の **"Hi, We are Echo"** を **"Hello, We are Echo"** に変更しブラウザで表示確認
:::
<!-- //Step -->

## Echoとは？
GoのWebフレームワークです。**「High performance, extensible, minimalist Go web framework」** のキャッチコピーの通り、高性能かつ軽量なのが特徴です。
他にも[Gin](https://github.com/gin-gonic/gin)などGo製のWebフレームワークはいくつかありますが今回はEchoを使用します。
https://echo.labstack.com/

## Echoのインストール
まずは、下記のコマンドでEchoをインストールします。
```
$ go get github.com/labstack/echo/v4
```
:::message alert
インストールの前後にエラーなどが起きましたら **go mod tidy** を実行してみてください。
:::

## Webサーバーを起動する
インストールが完了しましたら`main.go`を下記のように編集します。

```go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
)

func main() {

  e := echo.New()

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

`main.go`を実行してWebサーバーを立ち上げます。

```
$ go run main.go
```

`main.go`を実行するとコマンドラインに下記のように表示されます。この表示になればEchoが正常に起動しWebサーバーが立ち上がったことになります。

```

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v4.6.3
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:1323
```

ブラウザで[http://localhost:1323](http://localhost:1323)にアクセスしてみましょう。
**"Hi, We are Echo"** と表示されるはずです。

これでWebサーバーが起動できました。
今度はページを増やしてみましょう。`main.go`を編集します。

```diff go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
)

func main() {

  e := echo.New()

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

+  // localhost:1323/foo へGETでアクセスした時の処理
+  e.GET("/foo", func(c echo.Context) error {
+    return c.String(200, "I am Foo")
+  })

+  // localhost:1323/bar へGETでアクセスした時の処理
+  e.GET("/bar", func(c echo.Context) error {
+    return c.String(200, "I am Bar")
+  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
編集できましたら、Echoを再起動します。
ターミナルで **「control + c」** を入力し、一度Echoを終了させてから、`go run main.go`を実行します。次に、ブラウザでそれぞれのURLにアクセスして、表示を確認してみましょう。

下記の表のように表示されます。
| URL | 表示 |
| ---- | ---- |
| http://localhost:1323 | Hi, We are Echo! |
| http://localhost:1323/foo | I am Foo |
| http://localhost:1323/bar | I am bar |

簡単にページが増やせてとても便利ですね。

これで、Echoを使ってWebサーバーを起動することはできましたが、開発を進めるにはこのままではちょっと不便ですので、代表的なEchoのミドルウェアである下記の2つを追加しておきます。

#### Loggerについて
ユーザーエージェントやメソッド情報（GETやPOST）など様々なリクエスト時の情報が取得できます。

#### Recoverについて
サーバーが予期しないエラーによりプログラムを停止させてしまっても、サーバーは落とさずエラーレスポンスを返せるようにリカバリーしてくれます。

`main.go`を編集します。
```diff go:main.go
package main

import (
  "net/http"

  "github.com/labstack/echo/v4"
+  "github.com/labstack/echo/v4/middleware"
)

func main() {

  e := echo.New()

+  // ミドルウェア追加
+  e.Use(middleware.Logger())
+  e.Use(middleware.Recover())

  // localhost:1323 へGETでアクセスした時の処理
  e.GET("/", func(c echo.Context) error {
    return c.String(200, "Hi, We are Echo")
  })

  // localhost:1323/foo へGETでアクセスした時の処理
  e.GET("/foo", func(c echo.Context) error {
    return c.String(200, "I am Foo")
  })

  // localhost:1323/bar へGETでアクセスした時の処理
  e.GET("/bar", func(c echo.Context) error {
    return c.String(200, "I am Bar")
  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
再度、Echoを起動し直してブラウザでアクセスしてみて下さい。
ミドルウェアを追加したので、ターミナルにユーザーエージェントやメソッドなど様々な情報が表示されています。

下記は私の環境でのアクセスログです。
```
{"time":"2022-02-23T15:35:39.478296+09:00","id":"","remote_ip":"::1","host":"localhost:1323","method":"GET","uri":"/bar/","user_agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36","status":200,"error":"","latency":10756,"latency_human":"10.756µs","bytes_in":0,"bytes_out":8}
```

ミドルウェアではありませんが、最後にもう一つ。
今は`main.go`を編集するたびに **「control + cで終了→main.goを再度実行」** する必要がります。

これは、正直面倒ですのでファイルに変更があった場合、**自動的にmain.goを再度実行（ホットリロード）** できるように、[Air](https://github.com/cosmtrek/air)とゆうパッケージを追加しておきます。

まずは、**「control + c」** でEchoを終了し、下記のコマンドでAirをインストールします。
```
$ go get github.com/cosmtrek/air
```

続いて、PATHを通します。
bashまたは、zshをどちらを使用しているかによって記述するファイルが変わってきますのでご自身の環境に合わせてください。

```
# bashの場は、~/.bashrc または ~/.bash_profileに下記を追記。
# zshの場合は、~/.zshrc または ~/.zprofileに下記を追記。
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

PATHを通したらターミナルで次のコマンドを入力してAirの動作を確認してみます。
下記のように表示されていればAirのセッティングは完了です。
```
$ air -v

  __    _   ___
 / /\  | | | |_)
/_/--\ |_| |_| \_ , built with Go

```

ではホットリロードができているか確認してみましょう。
ターミナルで`air`と入力してAirを起動します。
```
$ air
```
ブラウザでhttp://localhost:1323にアクセスしてみます。今はまだ変更がありませんので、**"Hi, We are Echo"** と表示されるはずです。

次に、`main.go`の **"Hi, We are Echo"** を **"Hello, We are Echo"** に変更し、保存します。
再度ブラウザでhttp://localhost:1323にアクセスすると表示が変わっています。

これで、いちいちファイルを変更するたびに **「control + cで終了→main.goを再度実行」の必要がなくなりました。**
開発時はできるだけホットリロードの状態で開発を進めていきましょう。

次のページでは、CRUD処理を実装する前の準備を行います。

