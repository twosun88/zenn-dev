---
title: "Read（GET）の実装"
free: false
---

このページではCRUDのReadの実装を行い、Postmanで動作確認を行います。

## Read（GET）の実装
データの取得（Read）は、GETリクエストでクエリパラーメータを受け取り、その内のメールアドレスでデータベースを検索し、データがあるか確認します。（データがない場合は404を返す）

現在の構造体はJSONデータは受け取れますが、クエリパラメータを受け取ることができないので、まずは、構造体にタグを追加しクエリパラメータを受け取れるようにしておきましょう。

:::message
GETリクエストのみクエリパラーメータでデータを受け取りますが、POST、PUT、DELETEはJSONで受け取ります。
:::

構造体「User」のEmail フィールドに **query:"email"** を追加します。
```diff go:structs/structs.go
type User struct {
  ID        int       `gorm:"autoIncrement" json:"id"`
  Name      string    `gorm:"type:text; not null" json:"name"`
+ Email     string    `gorm:"type:text; not null" json:"email" query:"email" validate:"required"`
  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```
これで、クエリを受け取れるようになりました。

つづいて、`main.go`にGET用のエンドポイントを追加します。
なおPOST時の処理は変更ありませんので省略してあります。

:::message alert
Create（POST）時と**型の整合性チェックの処理が異なる**のでご注意ください。
:::

```diff go:main.go
package main

import (
  "encoding/json"
  "example-golearn/structs"
  "example-golearn/utilities"

  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

// バリデーション用メソッド
type CustomValidator struct {
  validator *validator.Validate
}

func (cv *CustomValidator) Validate(i interface{}) error {
  return cv.validator.Struct(i)
}

func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

  // バリデーター登録
  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへPOSTリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {
    〜 省略 〜
  })

+  // /usersへのGETリクエスト時の処理
+  e.GET("/users", func(c echo.Context) error {+

+    user := new(structs.User)+

+    /* バインド処理（クエリの値はstring型で送られてくるので型はマッチする） */
+    if err := c.Bind(user); err != nil {
+      return c.JSON(400, err.Error())
+    }+

+    /*
+     emailの値をチェックする。
+     空もしくはメールアドレスとして正しくない場合はエラーを返す。
+    */
+    if err := c.Validate(user); err != nil {
+      return c.JSON(400, err.Error())
+    }+

+    /*
+      バリデーションがOKなら下記の処理を実行していく
+    */
+    // データベース接続
+    db, err := utilities.DB()+

+    // データベース接続エラー時の処理
+    if err != nil {
+      return c.JSON(400, err.Error())
+    }+

+    // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
+    var u structs.User
+    r := db.Where(&structs.User{Email: user.Email}).First(&u)+

+    // データ取得エラー時の処理
+    if r.Error != nil {
+      return c.JSON(400, r.Error)
+    }

+    // 取得したデータを返す
+    return c.JSON(200, u)
+  })


  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
今回はRead（GET）ですので、**`db.Where(&structs.User{Email: user.Email}).First(&u)`** でデータを取得しリターンしています。

## Postmanで動作確認する
では、PostmanでGETリクエストを送り動作確認してみましょう。
設定方法は下記の画像をご参考ください。
<！！！POSTMANの画像！！！>

送信したらPostmanの画面下部にレスポンスが表示されます。
下記のようなレスポンスが返ってきてると思います。

```json
{
  "id": 1,
  "name": "Taro",
  "email": "golan@example.com"
}
```
ちゃんと、指定したメールアドレスのユーザー情報が返ってきてますね。

型の整合性やメールアドレスの形式を確認したい場合は、前回と同じやり方でOKです。
Postmanで、**email: ""** （空の値）、**email: "golearnexample.com"** （@マークがない状態）の2パターンを送信して動きをチェックしてみましょう。

同じようにエラーメッセージが返ってくると思います。

最後に、Read処理の流れをおさらいしてみましょう。
> 1. /usersへGETリクエストが送られてくる。
> 2. バリデーションチェック（emailが空、またはメールアドレスの形式が正しいか？）
> 3. バリデーションエラーの場合、処理を中断しエラーを返す（ステータス 400）
> 4. バリデーションがOKならデータベースへ接続しデータを取得する
> 5. データがなければ404を返す。あればデータを取得
> 7. 取得したデータを返す（ステータス 200）

この先、実装するUpdateとDeleteでもそうですが、**基本的には先にデータのエラーチェックを行い、エラーがなければ正常時の処理を実行する**とゆう流れは変わりありませんね。

次は、Update（PUT）を実装していきます。

