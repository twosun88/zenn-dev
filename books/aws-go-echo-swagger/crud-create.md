---
title: "CRUDのCreate（POST）を実装する"
free: false
---

このページではCRUDの実装を行い、Postmanで動作確認を行います。

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

## 構造体にタグを追加
POSTやPUTで送られてきたJSONデータを受け取り、リターンするためには、構造体にタグを追加し、送られてくるデータと構造体をマッピングする必要があります。
まずは、`structs.go`を開き、構造体「User」にタグを追加します。

```diff go:structs/structs.go
type User struct {
-  ID        int       `gorm:"autoIncrement"`
-  Name      string    `gorm:"type:text; not null"`
-  CreatedAt time.Time `gorm:"not null; autoCreateTime"`
-  UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
}

type User struct {
+  ID        int       `gorm:"autoIncrement" json:"id"`
+  Name      string    `gorm:"type:text; not null" json:"name"`
+  CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
+  UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}

```
IDやNameのフィールドに、**json:"id"、json:"name"** などを追加してマッピング完了です。
これで送られてくるJSONデータの型をチェックできるようになりました。
ちなみに、**json:"-"** となっているフィールドはリターン時に含まない設定です。


## Create（POST）の実装
構造体とデータのマッピングが終わったら「ユーザー登録」用の処理を書いていきます。

**送られてきたデータの型をチェックして、** 問題なければ「users」テーブルへ登録します。
今回、登録するのは「ユーザー名（name）」のみです。

では、`main.go`を編集しましょう。
変更箇所が多いので差分は非表示にしています。
:::message
id、created_at、updated_atは自動的に設定されます。
:::
```go:main.go
package main

import (
  "encoding/json"
  "example-golarn/structs"
  "example-golarn/utilities"

  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)


func main() {

  e := echo.New()

  // ミドルウェア追加
  e.Use(middleware.Logger())
  e.Use(middleware.Recover())

  // /usersへPostリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {

    newUser := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、newUserにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&newUser); err != nil {
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
    d := structs.User{Name: newUser.Name}

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

コメントにも記載してますが、上記のプログラムの動きは下記のようになっています。
> 1. /usersへPOSTリクエストが送られてくる。
> 2. データの型をチェックする（nameがstringであるか）
> 3. 型が違っている場合、処理を中断しエラーを返す（ステータス 400）
> 4. 問題なければデータベースへ接続しデータを登録する
> 5. 登録が完了したら、登録したデータを返す（ステータス 200）

では、**Postman**を使ってチェックしてみましょう。
:::message
MampまたはXamppを起動しておいて下さい。
:::

## Postmanで動作確認
まずは、AirでWebサーバーを起動しっぱなしにしておきます
```
$ air
```
つづいて、Postmanで**http://localhost:1323/user**にPOSTでJSONデータを送信します。
ここでは、**nameに"Taro"** を設定していますが、値はお好みで構いません。
設定方法は下記の画像をご参考ください。
![Postman](https://storage.googleapis.com/zenn-user-upload/ac116ca0183b-20220227.png)
[大きい画像はこちら](https://storage.googleapis.com/zenn-user-upload/ac116ca0183b-20220227.png)

送信したらPostmanのウィンドウ下部にレスポンスが表示されます。
下記のようなレスポンスが返ってきてると思います。
```json
{
  "id": 1,
  "name": " Taro"
}
```
初めての登録なので当然、idは1が返ってきてますね。
CreatedAtとUpdatedAtはタグに**json:"-"** を指定したのでレスポンスには含まれていません。

ブラウザでphpMyAdminにアクセスしデータを確認してみると実際にデータが登録されているのが確認できます。

これでひとまず、POST（Create）はできました。

型の整合性チェックの動きも確認してみましょう。**nameはstring型なので、int型である数字を送信してみます。** PostmanでJSONデータ **"name:1"** と入力し送信します。

下記のようなエラーが返ってくれば、型のチェックが正常に動いていますので問題ありません。
:::message alert
"json: cannot unmarshal number into Go struct field User.name of type string"
:::

型の整合性チェックはできましたが、今の段階だと1つ問題があります。
**nameを必須に設定していないので、nameが空の場合でも登録できてしまいます。**
これでは困りますよね。
:::message
確認されたい場合は、Postmanで **name:""** を送信してみください。
:::

nameを必須に設定し、空だった場合はエラーを返す処理を追加したいと思います。


## Validatorをインストールする
まずは、コマンドで下記を実行し、validatorをインストールします。
```
$ go get github.com/go-playground/validator/v10
```
https://github.com/go-playground/validator

インストールが完了したら、まずは **validate:"required"** タグをNameフィールドに追加します。
```diff go:structs/structs.go
type User struct {
}

type User struct {
ID        int       `gorm:"autoIncrement" json:"id"`
+  Name      string    `gorm:"type:text; not null" json:"name" validate:"required"`
CreatedAt time.Time `gorm:"not null; autoCreateTime" json:"-"`
UpdatedAt time.Time `gorm:"not null; autoUpdateTime" json:"-"`
}
```
つづいて、`main.go`にvalidatorをインポートし処理を追加します。

```diff go:main.go
package main

import (
  "encoding/json"
  "example-golarn/structs"
  "example-golarn/utilities"

+  "github.com/go-playground/validator/v10"
  "github.com/labstack/echo/v4"
  "github.com/labstack/echo/v4/middleware"
)

+// バリデーション用メソッドを追加する
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

  // /usersへPostリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {

    newUser := new(structs.User)

    /*
      型の整合性チェック：送られてきたJSONデータをデコードし、型の整合性チェックする。
      OK： エラーを返す（ステータス 400）
      NG： 送られてきたデータを、newUserにバインドする
    */
    if err := json.NewDecoder(c.Request().Body).Decode(&newUser); err != nil {
      return c.JSON(400, err.Error())
    }

+    // 必須項目（name）の値が空かチェック。空の場合エラーを返す。
+    if err := c.Validate(newUser); err != nil {
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
    d := structs.User{Name: newUser.Name}

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
これで、**nameが空**の場合はエラーを返す処理ができました。
試しにPostmanで、**name: ""** を送信してみて、下記のようなエラーが返って来ればバリデーションがちゃんと動いています。

:::message alert
"Key: 'User.Name' Error:Field validation for 'Name' failed on the 'required' tag"
:::

ここでCreate処理の流れをおさらいしてみましょう。
> 1. /usersへPOSTリクエストが送られてくる。
> 2. データの型をチェックする（nameがstringであるか）
> 3. 型が違っている場合、処理を中断しエラーを返す（ステータス 400）
> 4. バリデーションエラーの場合、処理を中断しエラーを返す（ステータス 400）
> 5. 問題なければデータベースへ接続しデータを登録する
> 6. 登録が完了したら、登録したデータを返す（ステータス 200）

5つのステップが6つになりました。

これで、CRUDの内のCreate（POST）は完了です。
型や必須項目、データ名などが構造体からわかるので、データベースの構造がすぐに把握できると思います。私がGoを選んだ最大の要因はここです。

次は、Update（PUT）を実装していきます。
