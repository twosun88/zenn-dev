---
title: "Update（PUT）の実装"
free: false
---

このページではCRUDのUpdateの実装を行い、Postmanで動作確認を行います。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. ##### `main.go`にPUT用のエンドポイントを追加
```diff go:main.go
package main

import (
  "encoding/json"
  "example-golarn/structs"
  "example-golarn/utilities"

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

  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへPOSTリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへGETリクエストが来た時の処理
  e.GET("/users", func(c echo.Context) error {
    〜 省略 〜
  })

+ // /usersへPUTリクエストが来た時の処理
+ e.PUT("/users", func(c echo.Context) error {
+
+   user := new(structs.User)
+
+   /*** エラーチェック ***/
+
+   // 送られてきたJSONデータをデコードし、型の整合性チェックする。
+   if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
+   if err := c.Validate(user); err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   /*** エラーがなければここから下の処理を実行する ***/
+
+   // データベース接続
+   db, err := utilities.DB()
+
+   // データベース接続エラー時の処理
+   if err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
+   var u structs.User
+   r := db.Where(&structs.User{Email: user.Email}).First(&u)
+
+
+   // データ取得エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新するデータの作成
+   d := structs.User{
+     Name:  user.Name,
+     Email: user.Email,
+   }
+
+   // u.IDでレコードを指定しデータを更新する
+   r = db.Model(&structs.User{ID: u.ID}).Updates(&d)
+
+   // 更新エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新したデータを再取得する
+   r = db.Where(&structs.User{Email: user.Email}).First(&u)
+
+   // 更新したデータの再取得エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新したデータを返す
+   return c.JSON(200, u)
+ })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
2. ##### Postmanで動作確認
:::
<!-- /Step -->

## Update（PUT）の実装
データの更新（Update）は、GETの時と同じようにPUTリクエストで送られてきたJSONデータを受け取り、メールアドレスでデータベースを検索し、まず、データがあるか確認します。

その後、データがあれば、データベースを更新し最終的に更新したデータを返す流れてになっています。（データがない場合は404を返す）

なお、構造体はすでにJSONでもクエリパラーメータでもデータを受け取れる状態になっているので編集は必要ありません。

`main.go`にPUT用のエンドポイントを追加します。
なおPOSTとGET時の処理は変更ありませんので省略してあります。

```diff go:main.go
package main

import (
  "encoding/json"
  "example-golarn/structs"
  "example-golarn/utilities"

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

  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへPOSTリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへGETリクエストが来た時の処理
  e.GET("/users", func(c echo.Context) error {
    〜 省略 〜
  })

+ // /usersへPUTリクエストが来た時の処理
+ e.PUT("/users", func(c echo.Context) error {
+
+   user := new(structs.User)
+
+   /*** エラーチェック ***/
+
+   // 送られてきたJSONデータをデコードし、型の整合性チェックする。
+   if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
+   if err := c.Validate(user); err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   /*** エラーがなければここから下の処理を実行する ***/
+
+   // データベース接続
+   db, err := utilities.DB()
+
+   // データベース接続エラー時の処理
+   if err != nil {
+     return c.JSON(400, err.Error())
+   }
+
+   // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
+   var u structs.User
+   r := db.Where(&structs.User{Email: user.Email}).First(&u)
+
+
+   // データ取得エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新するデータの作成
+   d := structs.User{
+     Name:  user.Name,
+     Email: user.Email,
+   }
+
+   // u.IDでレコードを指定しデータを更新する
+   r = db.Model(&structs.User{ID: u.ID}).Updates(&d)
+
+   // 更新エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新したデータを再取得する
+   r = db.Where(&structs.User{Email: user.Email}).First(&u)
+
+   // 更新したデータの再取得エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // 更新したデータを返す
+   return c.JSON(200, u)
+ })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
**`db.Model(&structs.User{ID: u.ID}).Updates(&d)`** でデータをアップデートしています。

## Postmanで動作確認する
では、Postmanで動作確認してみましょう。
**「名前」** を変更するJSONを記述しPUTリクエストを送ってみます。
（ここでは「Taro」から「Hanako」に名前を変更しています）

設定方法は下記の画像をご参考ください。
<！！！POSTMANの画像！！！>

送信したらPostmanの画面下部にレスポンスが表示されます。
下記のようなレスポンスが返ってきてると思います。

```json
{
  "id": 1,
  "name": "Hanako",
  "email": "golan@example.com"
}
```

**nameの部分が「Taro」から「Hanako」に変わっている**のでちゃんと更新した後のデータが返ってきてることがわかります。

今度はデータベースにデータがない場合の動きをチェックするため、メールアドレスを変更しPUTリクエストを送ってみます。（ここでは例として「xyz@example.com」に変更）

下記のようなレスポンスが返ってくると思います。
```json
"Not Found"
```
データが存在しないので意図した通り、**Not Found** が返ってきました。

最後に、Update処理の流れをおさらいしてみましょう。
> 1. /usersへPUTリクエストが送られてくる。
> 2. バリデーションチェック（emailが空、またはメールアドレスの形式が正しいか？）
> 3. バリデーションエラーの場合、処理を中断しエラーを返す（ステータス 400）
> 4. バリデーションがOKならデータベースへ接続しデータを検索する
> 5. データがなければ404を返す。データがあればデータを更新
> 6. 更新したデータを返す（ステータス 200）

GETの時とほぼ同じように、**エラーチェック→データ検索→処理**の流れになっています。

次は、Delete（DELETE）を実装していきます。

