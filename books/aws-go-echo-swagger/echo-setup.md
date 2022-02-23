---
title: "Echoのインストールとセットアップ"
free: false
---

このページではGoのWebフレームワークを使用してWebサーバーを立ち上げます。

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
何かエラーなどがでた場合は **go mod tidy** を実行して再度インストールを試してください。
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
  e.Logger.Fatal(e.Start(":1323"))
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

:::message
$ go run main.go実行後に **「アプリケーション“main”へのネットワーク受信接続を許可しますか?」** などのダイアログが出た場合は、
**e.Start(":1323")** の部分を **e.Start("localhost:1323")** に変えてから実行して下さい。
:::

確認ができましたら`main.go`を編集していくつかURLを追加してみましょう。

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

+  // localhost:1323/foo/ へGETでアクセスした時の処理
+  e.GET("/foo/", func(c echo.Context) error {
+    return c.String(200, "I am Foo")
+  })

+  // localhost:1323/bar/ へGETでアクセスした時の処理
+  e.GET("/bar/", func(c echo.Context) error {
+    return c.String(200, "I am Bar")
+  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start(":1323"))
}
```
編集できましたら、Echoを再起動します。
ターミナルで「control + C」を入力し一度Echoを終了させ、`go run main.go`を実行します。
Echoが再起動できましたらブラウザでそれぞれのURLにアクセスして表示を確認してみましょう。

下記の表のように表示されます。
| URL | 表示 |
| ---- | ---- |
| http://localhost:1323 | Hi, We are Echo! |
| http://localhost:1323/foo/ | I am Foo |
| http://localhost:1323/bar/ | I am bar |

簡単にページが増やせましたね。
これで、Echoを使ってWebサーバーを起動することはできました。

ですが、このまま開発を進めるにはちょっと不便ですので、代表的なEchoのミドルウェアである下記の2つを追加しておきます。

#### Logger
ユーザーエージェントやメソッド情報（GETやPOST）など様々なリクエスト時の情報が取得できます。

#### Recover
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

  // localhost:1323/foo/ へGETでアクセスした時の処理
  e.GET("/foo/", func(c echo.Context) error {
    return c.String(200, "I am Foo")
  })

  // localhost:1323/bar/ へGETでアクセスした時の処理
  e.GET("/bar/", func(c echo.Context) error {
    return c.String(200, "I am Bar")
  })

  // ポート番号は1323
  e.Logger.Fatal(e.Start(":1323"))
}
```
再度、Echoを起動し直してブラウザでアクセスしてみて下さい。
ミドルウェアを追加したので、ターミナルにユーザーエージェントやメソッドなど様々な情報が確認できます。

下記は私の環境でのアクセスログです。
```
{"time":"2022-02-23T15:35:39.478296+09:00","id":"","remote_ip":"::1","host":"localhost:1323","method":"GET","uri":"/bar/","user_agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36","status":200,"error":"","latency":10756,"latency_human":"10.756µs","bytes_in":0,"bytes_out":8}
```

ミドルウェアではありませんが、最後にもう一つ、今は`main.go`を編集するたびにターミナルでEchoを再起動する必要があるので、効率がよくないと思います。

そこで、ファイルに変更があった場合、自動的にEchoが再起動（ホットリロード）できるように[Air](https://github.com/cosmtrek/air)とゆうパッケージを追加しておきます。

まずは、「control + C」でEchoを終了し、下記のコマンドでAirをインストールします。
```
$ go get github.com/cosmtrek/air
```

続いて、PATHを通します。
bashまたは、zshをどちらを使用しているかによって記述するファイルが変わってきますのでご自身の環境に合わせてください。

```
# bashの場は、~/.bashrc または ~/.bash_profileに追記。
# zshの場合は~/.zshrc または ~/.zprofileに追記。
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

ではホットリロードできてるか確認してみましょう。
ターミナルで`air`と入力してAirを起動します。
```
$ air
```
ブラウザでhttp://localhost:1323にアクセスしてみましょう。今はまだ変更がありませんので、**"Hi, We are Echo"** と表示されるはずです。

次に、`main.go`の **"Hi, We are Echo"** を **"Hello, We are Echo"** に変更してみます。
再度ブラウザでhttp://localhost:1323にアクセスすると表示が変わっています。

これで、いちいちファイルを変更するたびにEchoを再起動する必要がなくなりましたので効率がよくなりました。開発時はできるだけホットリロードの状態で開発を進めていきましょう。

次のページからは、CRUD処理を実装していきます。

