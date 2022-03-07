---
title: "Create（POST）の実装"
free: false
---

このページではCRUDのCreateの実装を行い、Postmanで動作確認を行います。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. ##### 構造体にタグを追加する
```diff go:structs/structs.go
type User struct {
+  ID        int       `gorm:"autoIncrement" json:"id"`
+  Name      string    `gorm:"type:text;" json:"name"`
+  Email     string    `gorm:"type:text; not null" json:"email"`
+  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
+  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```

2. ##### Create（POST）の実装
```go:main.go
package main

import (
  "encoding/json"
  "example-golearn/structs"
  "example-golearn/utilities"

  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)


func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

  // /usersへのPOSTリクエスト時の処理
  e.POST("/users", func(c echo.Context) error {

    user := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、userにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
      return c.JSON(400, err.Error())
    }

    /*
      型チェックがOKなら下記の処理を実行していく
    */
    // データベース接続
    db, err := utilities.DB()

    // データベース接続エラー時の処理
    if err != nil {
      return c.JSON(400, err.Error())
    }

    // データベースに登録するデータの作成
    d := structs.User{
      Name:  user.Name,
      Email: user.Email,
    }

    // データベースに登録する
    p := db.Create(&d)

    // データベース登録エラー時の処理
    if err := p.Error; err != nil {
      return c.JSON(400, err.Error())
    }

    // 登録したデータを返す
    return c.JSON(200, d)

  })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

3. ##### Postmanで動作確認

4. ##### Validatorをインストールし、`structs.go`と`main.go`を編集
```
$ go get github.com/go-playground/validator/v10
```
```diff go:structs/structs.go
type User struct {
  ID        int       `gorm:"autoIncrement" json:"id"`
  Name      string    `gorm:"type:text; not null" json:"name"`
+ Email     string    `gorm:"type:text; not null" json:"email" validate:"required"`
  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```
```diff go:main.go
package main

import (
  "encoding/json"
  "example-golearn/structs"
  "example-golearn/utilities"

+  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

+// バリデーション用メソッド
+ type CustomValidator struct {
+   validator *validator.Validate
+ }

+ func (cv *CustomValidator) Validate(i interface{}) error {
+   return cv.validator.Struct(i)
+ }

func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

+  // バリデーター登録
+  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへのPOSTリクエスト時の処理
  e.POST("/users", func(c echo.Context) error {

    user := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、userにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
      return c.JSON(400, err.Error())
    }

+    /*
+     emailの値をチェックする。
+     空もしくはメールアドレスとして正しくない場合はエラーを返す。
+    */
+    if err := c.Validate(user); err != nil {
+      return c.JSON(400, err.Error())
+    }

    /*
      型チェック&バリデーションがOKなら下記の処理を実行していく
    */
    // データベース接続
    db, err := utilities.DB()

    // データベース接続エラー時の処理
    if err != nil {
      return c.JSON(400, err.Error())
    }

    // データベースに登録するデータの作成
    d := structs.User{
      Name:  user.Name,
      Email: user.Email,
    }

    // データベースに登録する
    p := db.Create(&d)

    // データベース登録エラー時の処理
    if err := p.Error; err != nil {
      return c.JSON(400, err.Error())
    }

    // 登録したデータを返す
    return c.JSON(200, d)

  })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
5. ##### Postmanでバリデーターの動作確認

:::
<!-- /Step -->

## CRUDとは？
**Create, Read, Update, Delete** の頭文字を取った言葉で、データベースを操作する上での基本的な機能のことを指します。
ブラウザからのリクエストと照らし合わせると下記の図のようになります。
| リクエスト | CRUD | データ形式 | 具体的な処理 |
| ---- | ---- | ---- | ---- |
| POST | Create | JSON形式 | データを登録 |
| GET | Read | クエリ形式 | データを取得 |
| PUT | Update | JSON形式 | データを更新 |
| DELETE | Delete | JSON形式 | データを削除 |

:::message
GETとPOSTリクエストだけでも更新や削除はできますが、今回はREST APIをモデルにしたAPIを作成しますので「POST、GET、PUT、DELETE」の全てを使います。
:::

## 構造体にタグを追加する
POSTやPUTで送られてきたJSONデータを受け取り、リターンするためには、構造体にタグを追加し、送られてくるデータと構造体をマッピングする必要があります。
まずは、`structs.go`を開き、構造体「User」にタグを追加します。

```diff go:structs/structs.go
type User struct {
+  ID        int       `gorm:"autoIncrement" json:"id"`
+  Name      string    `gorm:"type:text;" json:"name"`
+  Email     string    `gorm:"type:text; not null" json:"email"`
+  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
+  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```
IDやNameのフィールドに、**json:"name"、json:"email"** などを追加してマッピング完了です。
これで送られてくるJSONデータの型をチェックできるようになりました。
ちなみに、**json:"-"** となっているフィールドはリターン時に含まない設定です。


## Create（POST）の実装
構造体とデータのマッピングが終わったら「ユーザー登録」用の処理を書いていきます。

**送られてきたデータの型をチェックして、** 問題なければ「users」テーブルへ登録します。
今回、登録するのは「ユーザー名（name）とメールアドレス（email）」の2つです。

では、`main.go`を編集しましょう。
変更箇所が多いので差分は非表示にしています。

:::message
id、created_at、updated_atは自動的に設定されます。
:::

```go:main.go
package main

import (
  "encoding/json"
  "example-golearn/structs"
  "example-golearn/utilities"

  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)


func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

  // /usersへのPOSTリクエスト時の処理
  e.POST("/users", func(c echo.Context) error {

    user := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、userにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
      return c.JSON(400, err.Error())
    }

    /*
      型チェックがOKなら下記の処理を実行していく
    */
    // データベース接続
    db, err := utilities.DB()

    // データベース接続エラー時の処理
    if err != nil {
      return c.JSON(400, err.Error())
    }

    // データベースに登録するデータの作成
    d := structs.User{
      Name:  user.Name,
      Email: user.Email,
    }

    // データベースに登録する
    p := db.Create(&d)

    // データベース登録エラー時の処理
    if err := p.Error; err != nil {
      return c.JSON(400, err.Error())
    }

    // 登録したデータを返す
    return c.JSON(200, d)

  })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```

では、**Postman**を使ってチェックしてみましょう。
:::message
MAMPまたはXAMPPを起動しておいて下さい。
:::

## Postmanで動作確認する
Postmanのインストールがまだの方は下記からインストールしてください。
[https://www.postman.com/](https://www.postman.com/)

まずは、Webサーバーを起動しっぱなしにしておくため、下記のコマンドをターミナルで入力します。
```
$ air
```
つづいて、Postmanで**http://localhost:1323/user**にPOSTでJSONデータを送信します。
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
初めての登録なので当然、idは1が返ってきてますね。
CreatedAtとUpdatedAtはタグに**json:"-"** を指定したのでレスポンスには含まれていません。

ブラウザでphpMyAdminにアクセスしデータを確認してみると実際にデータが登録されているのが確認できます。

これでひとまず、登録する処理はできました。

型の整合性チェックの動きも確認してみましょう。**emailはstring型なので、int型である数字を送信してみます。** PostmanでJSONデータ **"email:1"** と変更し送信します。

下記のようなエラーが返ってくれば、型のチェックが正常に動いていますので問題ありません。
:::message alert
"json: cannot unmarshal number into Go struct field User.email of type string"
:::

型の整合性チェックはできましたが、今の段階だと1つ問題があります。
**必須パラメータを設定していないので、空のデータが送られてきた場合でも登録できてしまいます。**
これでは困りますよね。（確認する場合は、Postmanで **email:""** を送信してみください。）

**emailを必須パラメータに設定し、値が空、または、メールアドレスの形式が正しくない場合は、**
エラーを返す処理を追加します。


## Validatorをインストールする
まずは、コマンドで下記を実行し、validatorをインストールします。
```
$ go get github.com/go-playground/validator/v10
```
https://github.com/go-playground/validator

インストールが完了したら、まずはEmailフィールドに **validate:"required"** タグを追加します。
```diff go:structs/structs.go
type User struct {
  ID        int       `gorm:"autoIncrement" json:"id"`
  Name      string    `gorm:"type:text; not null" json:"name"`
+ Email     string    `gorm:"type:text; not null" json:"email" validate:"required"`
  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```
つづいて、`main.go`にvalidatorをインポートし処理を追加します。

```diff go:main.go
package main

import (
  "encoding/json"
  "example-golearn/structs"
  "example-golearn/utilities"

+  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

+// バリデーション用メソッド
+ type CustomValidator struct {
+   validator *validator.Validate
+ }

+ func (cv *CustomValidator) Validate(i interface{}) error {
+   return cv.validator.Struct(i)
+ }

func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

+  // バリデーター登録
+  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへのPOSTリクエスト時の処理
  e.POST("/users", func(c echo.Context) error {

    user := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、userにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&user); err != nil {
      return c.JSON(400, err.Error())
    }

+    /*
+     emailの値をチェックする。
+     空もしくはメールアドレスとして正しくない場合はエラーを返す。
+    */
+    if err := c.Validate(user); err != nil {
+      return c.JSON(400, err.Error())
+    }

    /*
      型チェック&バリデーションがOKなら下記の処理を実行していく
    */
    // データベース接続
    db, err := utilities.DB()

    // データベース接続エラー時の処理
    if err != nil {
      return c.JSON(400, err.Error())
    }

    // データベースに登録するデータの作成
    d := structs.User{
      Name:  user.Name,
      Email: user.Email,
    }

    // データベースに登録する
    p := db.Create(&d)

    // データベース登録エラー時の処理
    if err := p.Error; err != nil {
      return c.JSON(400, err.Error())
    }

    // 登録したデータを返す
    return c.JSON(200, d)

  })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
これで、**emailの値が空、またはメールアドレス形式が正しくない** 場合はエラーを返す処理ができました。

Postmanで**email: "" （空の値）、email: "golearnexample.com" （@マークがない状態）** の
2パターンを送信して動きをチェックしてみましょう。

それぞれ、下記のエラーが返ってきます。

##### ■値が空の場合
:::message alert
"Key: 'User.Name' Error:Field validation for 'Name' failed on the 'required' tag"
:::

##### ■メールアドレスの形式が正しくない場合
:::message alert
"Key: 'User.Email' Error:Field validation for 'Email' failed on the 'email' tag"
:::

これで、バリデーションの処理が追加できました。

最後に、Create処理の流れをおさらいしてみましょう。
> 1. /usersへPOSTリクエストが送られてくる。
> 2. データの型をチェックする（name、emailがstringか？）
> 3. 型が違っている場合、処理を中断しエラーを返す（ステータス 400）
> 4. バリデーションチェック（emailが空、またはメールアドレスの形式が正しいか？）
> 5. バリデーションエラーの場合、処理を中断しエラーを返す（ステータス 400）
> 6. バリデーションがOKならデータベースへ接続しデータを登録する
> 7. 登録が完了したら、登録したデータを返す（ステータス 200）

これで、CRUDの内のCreate（POST）は完了です。
型や必須パラメータ、データ名などが、構造体から一目でわかると思います。
私がGoを選んだ最大の要因はここです。

次は、Read（GET）を実装していきます。
