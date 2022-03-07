---
title: "コントローラーとモデルに分割する"
free: false
---

このページでは処理を役割毎に分割しコントローラーとモデルを作成します。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. ##### controllersとmodelsディレクトリを作り、その中に新規ファイルを作成
```
// 現時点でのディレクトリ構成
~/
 └─ golearn/
     ├─ constrollers/ // 追加
         └─ controllers.go // 追加
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ models/ // 追加
         └─ models.go // 追加
     ├─ structs/
         └─ structs.go
     ├─ utilities/
         └─ utilities.go
     ├─ go.mod
     └─ main.go
```

2. ##### `main.go`を編集する
```diff go:main.go
package main

import (
  "encoding/json"
+  "example-golearn/controllers" // controller読み込み
  "example-golearn/structs"
  "example-golearn/utilities"


  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

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

+  // /usersへPOSTリクエストが来たら、controllers.goのPost関数を実行
+  e.POST("/users", controllers.Post)

+  // /usersへGETリクエストが来たら、controllers.goのGet関数を実行
+  e.GET("/users", controllers.Get)

+  // /usersへPUTリクエストが来たら、controllers.goのPut関数を実行
+  e.PUT("/users", controllers.Put)

+  // /usersへDELETEリクエストが来たら、controllers.goのDelete関数を実行
+  e.DELETE("/users", controllers.Delete)

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

3. ##### `controllers.go`を編集する
```go:controllers/controllers.go
package controllers

import (
  "encoding/json"
  "example-golearn/models"
  "example-golearn/structs"

  "github.com/labstack/echo/v4"
)

func Post(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // 必須パラメータ（email）の値が空かチェック。空の場合エラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのPost関数を実行
  r, err := models.Post(user)

  // models.goのPost関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのPost関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Get(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたクエリデータの型の整合性チェックする。
  if err := c.Bind(user); err != nil {
    return c.JSON(400, err.Error())
  }

  // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  // models.goのGet関数を実行
  r, err := models.Get(user)

  // models.goのGet関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのGet関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Put(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // 必須パラメータ（email）の値が空かチェック。空の場合エラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのPost関数を実行
  r, err := models.Put(user)

  // models.goのPost関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのPut関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Delete(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのDelete関数を実行
  r := models.Delete(user)

  // models.goのDelete関数からの返り値にエラーがあるかチェック（正常時はnilが返ってくる）
  if r != nil {
    return c.JSON(400, r)
  }

  // エラーがなければmodels.goのDelete関数からの返り値はないので、文字列を出力
  return c.JSON(200, "deleted")
}
```

4. ##### `models.go`を編集する
```go:models/models.go
package models

import (
  "example-golearn/structs"
  "example-golearn/utilities"
)

func Post(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // データベースに登録するデータの作成
  u = structs.User{
    Name:  user.Name,
    Email: user.Email,
  }

  // データベースに登録する
  p := db.Create(&u)

  // データベース登録エラー時の処理
  if err := p.Error; err != nil {
    return u, p.Error
  }

  // 登録したデータを返す
  return u, nil
}

func Get(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 取得したデータを返す
  return u, nil
}

func Put(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新するデータの作成
  d := structs.User{
    Name:  user.Name,
    Email: user.Email,
  }

  // u.IDでレコードを指定しデータを更新する
  r = db.Model(&structs.User{ID: u.ID}).Updates(&d)

  // 更新エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新したデータを再取得する
  r = db.Where(&structs.User{Email: user.Email}).First(&u)

  // 更新したデータの再取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新したデータを返す
  return u, nil
}

func Delete(user *structs.User) error {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return r.Error
  }

  // u.IDでレコードを指定しデータを削除する
  r = db.Delete(&structs.User{}, "id = ?", u.ID)

  // 削除エラー時の処理
  if r.Error != nil {
    return r.Error
  }

  // 削除したらnilを返す
  return nil
}
```
:::
<!-- /Step -->
## コントローラーとモデルとは？
先に「MVCモデル」について説明します。
MVCは、アプリケーション設定を整理するための概念の一つで多くのフレームワークに取り入れられています。
**Model, View, Controller** のそれぞれの頭文字をとって **「MVC」** といいます。

それぞれ次の役割を持ちます。

> - **Model（モデル）**
システム内のビジネスロジックを担当、データベース関連の処理などを行います。

> - **View（ビュー）**
実際にユーザーに画面を表示したり、入力する機能の処理を行うのがビューです。

> - **Controller（コントローラー）**
コントローラーは、ユーザーの入力に基づきModelとViewを制御する役割を担っています。「クライアント（ユーザー）」「モデル」「ビュー」の橋渡し役となります。

今回はAPIですので**Viewは不要です。ModelとControllerのみ作成**します。

## ディレクトリを作成する
まずは、**controllersとmodels**ディレクトリを作り、その中に新規ファイルを作成します。
```
// 現時点でのディレクトリ構成
~/
 └─ golearn/
     ├─ constrollers/ // 追加
         └─ controllers.go // 追加
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ models/ // 追加
         └─ models.go // 追加
     ├─ structs/
         └─ structs.go
     ├─ utilities/
         └─ utilities.go
     ├─ go.mod
     └─ main.go
```
では、まず`main.go`に書かれているPOST処理をコントローラーとモデルに分割します。
GET、PUT、DELETEの処理は変わりありませんので省略します。

まずは、`main.go`を下記のように変更します。

```diff go:main.go
package main

import (
  "encoding/json"
+  "example-golearn/controllers" // controller読み込み
  "example-golearn/structs"
  "example-golearn/utilities"


  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

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

+  // /usersへPOSTリクエストが来たら、controllers.goのPost関数を実行
+  e.POST("/users", controllers.Post)

  // /usersへGETリクエストが来た時の処理
  e.GET("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへPUTリクエストが来た時の処理
  e.PUT("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへDELETEリクエストが来た時の処理
  e.DELETE("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
POSTリクエスト時の処理を`controllers.Post`でコントローラーに任せています。
次は、`controllers/controllers.go`を編集します。

## コントローラーを作る
コントローラーの役割は、ユーザーから送られてきたデータのチェックやモデルへのデータの受け渡しなど、**橋渡しを安全に行う**のが主な役割になります。

```go controllers/controllers.go
package controllers

import (
  "encoding/json"
  "example-golearn/models"
  "example-golearn/structs"

  "github.com/labstack/echo/v4"
)

func Post(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // 必須パラメータ（email）の値が空かチェック。空の場合エラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのPost関数を実行
  r, err := models.Post(user)

  // models.goのPost関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのPost関数からの返り値をリターン
  return c.JSON(200, r)
}
```
ほぼ`main.go`に記載されていた内容と変わりありません。
`main.go`で行っていた型の整合性チェックや必須パラメータのチェックを担当し、問題なければ、`models.Post(user)`を実行します。

では、`modesl.go`を作っていきましょう。

## モデルを作る
モデルの役割は、データベースへの登録や呼び出しなどデータベース関連の処理が主な役割になります。

```go models/models.go
package models

import (
  "example-golearn/structs"
  "example-golearn/utilities"
)

func Post(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // データベースに登録するデータの作成
  u = structs.User{
    Name:  user.Name,
    Email: user.Email,
  }

  // データベースに登録する
  p := db.Create(&u)

  // データベース登録エラー時の処理
  if err := p.Error; err != nil {
    return u, p.Error
  }

  // 登録したデータを返す
  return u, nil
}
```
こちらもほぼ`main.go`に記載されていた内容をコピペしただけです。
`Post(user *structs.User)`にてコントローラーからのデータを受け取りデータベースの処理を行っています。

これでコントローラーとモデルができました。
実際にPostmanでPOSTリクエストを送り動作確認してみましょう。
<！！！POSTMANの画像！！！>

## GET、PUT、DELET用のコントローラーとモデルを作る
POSTの動作確認できましたら、GET、PUT、DELETも同様に処理を分割しましょう。
ここまできたら、細かい解説は不要だと思いますので下記に全ての処理を分割済みの`main.go`、`controllers.go`、`models.go`を載せておきます。
```go:main.go
package main

import (
  "example-golearn/controllers"

  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

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

  // /usersへPOSTリクエストが来たら、controllers.goのPost関数を実行
  e.POST("/users", controllers.Post)

  // /usersへGETリクエストが来たら、controllers.goのGet関数を実行
  e.GET("/users", controllers.Get)

  // /usersへGETリクエストが来たら、controllers.goのPut関数を実行
  e.PUT("/users", controllers.Put)

  // /usersへGETリクエストが来たら、controllers.goのDelete関数を実行
  e.DELETE("/users", controllers.Delete)

  e.Logger.Fatal(e.Start("localhost:1323"))
}

```

```go:controllers/controllers.go
package controllers

import (
  "encoding/json"
  "example-golearn/models"
  "example-golearn/structs"

  "github.com/labstack/echo/v4"
)

func Post(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // 必須パラメータ（email）の値が空かチェック。空の場合エラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのPost関数を実行
  r, err := models.Post(user)

  // models.goのPost関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのPost関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Get(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたクエリデータの型の整合性チェックする。
  if err := c.Bind(user); err != nil {
    return c.JSON(400, err.Error())
  }

  // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  // models.goのGet関数を実行
  r, err := models.Get(user)

  // models.goのGet関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのGet関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Put(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // 必須パラメータ（email）の値が空かチェック。空の場合エラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのPost関数を実行
  r, err := models.Put(user)

  // models.goのPost関数からの返り値にエラーがあるかチェック
  if err != nil {
    return c.JSON(400, err)
  }

  // エラーがなければmodels.goのPut関数からの返り値をリターンする
  return c.JSON(200, r)
}

func Delete(c echo.Context) error {

  user := new(structs.User)

  /*** エラーチェック ***/

  // 送られてきたJSONデータをデコードし、型の整合性チェックする。
  if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
    return c.JSON(400, err.Error())
  }

  // emailの値のチェック。空もしくはメールアドレスとして正しくない場合はエラーを返す。
  if err := c.Validate(user); err != nil {
    return c.JSON(400, err.Error())
  }

  /*** エラーがなければここから下の処理を実行する ***/

  // models.goのDelete関数を実行
  r := models.Delete(user)

  // models.goのDelete関数からの返り値にエラーがあるかチェック（正常時はnilが返ってくる）
  if r != nil {
    return c.JSON(400, r)
  }

  // エラーがなければmodels.goのDelete関数からの返り値はないので、文字列を出力
  return c.JSON(200, "deleted")
}
```

```go:models/models.go
package models

import (
  "example-golearn/structs"
  "example-golearn/utilities"
)

func Post(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // データベースに登録するデータの作成
  u = structs.User{
    Name:  user.Name,
    Email: user.Email,
  }

  // データベースに登録する
  p := db.Create(&u)

  // データベース登録エラー時の処理
  if err := p.Error; err != nil {
    return u, p.Error
  }

  // 登録したデータを返す
  return u, nil
}

func Get(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 取得したデータを返す
  return u, nil
}

func Put(user *structs.User) (structs.User, error) {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return u, err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新するデータの作成
  d := structs.User{
    Name:  user.Name,
    Email: user.Email,
  }

  // u.IDでレコードを指定しデータを更新する
  r = db.Model(&structs.User{ID: u.ID}).Updates(&d)

  // 更新エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新したデータを再取得する
  r = db.Where(&structs.User{Email: user.Email}).First(&u)

  // 更新したデータの再取得エラー時の処理
  if r.Error != nil {
    return u, r.Error
  }

  // 更新したデータを返す
  return u, nil
}

func Delete(user *structs.User) error {

  var u structs.User

  // データベース接続
  db, err := utilities.DB()

  // データベース接続エラー時の処理
  if err != nil {
    return err
  }

  // 送られてきたメールアドレスでデータベースを検索しヒットしたレコードを変数 u に格納（なければ404を返す）
  r := db.Where(&structs.User{Email: user.Email}).First(&u)

  // データ取得エラー時の処理
  if r.Error != nil {
    return r.Error
  }

  // u.IDでレコードを指定しデータを削除する
  r = db.Delete(&structs.User{}, "id = ?", u.ID)

  // 削除エラー時の処理
  if r.Error != nil {
    return r.Error
  }

  // 削除したらnilを返す
  return nil
}
```

これで全てのリクエストをコントローラーとモデルに分割できたので、ファイル毎の内容を薄くできました。1つのファイルの内容を薄くすることで、**"責任と関心の分離"** ができたのでプロジェクトの管理がしやすくなったと思います。

型の生合成チェックの処理や必須パラメータのチェックなどの共通処理を、`utilities/utilities.go`にまとめてもいいですが、ここでは動きがわかりやすいようにあえてそのままにしています（ご興味がある方はぜひトライしてみてください）

これで、APIを作るプログラムの作成は全て完了です。
最後にこれまで作ったプログラムをビルドしバイナリファイルを作って終わりにしたいと思います。