---
title: "Delete（DELETE）の実装"
free: false
---

このページではCRUDのDeleteの実装を行い、Postmanで動作確認を行います。

## Delete（DELETE）の実装
データの削除（Delete）は、GETの時と同じようにDELETEリクエストで送られてきたJSONデータを受け取り、メールアドレスでデータベースを検索し、まず、データがあるか確認します。

その後、データがあれば、データを物理削除します。（データがない場合は404を返す）

:::message alert
**論理削除と物理削除とは？**
データベースからデータを削除する場合、主に**論理削除と物理削除**の2種類あります。

- **論理削除**
実際のデータは削除せず、データベース内のステータスを「無効」に変更しデータを利用できなくする。データは残っているのでステータスを「有効」に戻せば、またデータを利用することができる。

- **物理削除**
実際にデータを削除する（基本的に復元不可）
:::

`main.go`にDELETE用のエンドポイントを追加します。
なお今までのPOST、GET、PUT時の処理は変更ありませんので省略してあります。

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

  e.Validator = &CustomValidator{validator: validator.New()}

  // /usersへPOSTリクエストが来た時の処理
  e.POST("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへGETリクエストが来た時の処理
  e.GET("/users", func(c echo.Context) error {
    〜 省略 〜
  })

  // /usersへPUTリクエストが来た時の処理
  e.PUT("/users", func(c echo.Context) error {
    〜 省略 〜
  })

+ // /usersへDELETEリクエストが来た時の処理
+ e.DELETE("/users", func(c echo.Context) error {
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
+   // データ取得エラー時の処理
+   if r.Error != nil {
+     return c.JSON(400, r.Error)
+   }
+
+   // u.IDでレコードを指定しデータを削除する
+   r = db.Delete(&structs.User{}, "id = ?", u.ID)
+
+   // 削除エラー時の処理
+   if r.Error != nil {
+     return r.Error
+   }
+
+   // 削除完了後にステータス200とメッセージを返す
+   return c.JSON(200, "deleted")
+
+ })

  e.Logger.Fatal(e.Start("localhost:1323"))
}
```
**`db.Delete(&structs.User{}, "id = ?", u.ID)`** でデータを削除しています。
削除時の処理は、ほぼ、PUTの時と同じになっています。

## Postmanで動作確認する
では、Postmanで動作確認してみましょう。
メールアドレスを含めたJSONを記述しDELETEリクエストを送ってみます。

設定方法は下記の画像をご参考ください。
<！！！POSTMANの画像！！！>

送信したらPostmanの画面下部にレスポンスが表示されます。
下記のようなレスポンスが返ってきてると思います。

```json
"deleted"
```

物理削除を行っているので、当然、返すデータはありません。
なので、ここではただのテキスト（**"deleted"**）を返しています。

今度はデータベースにデータがない場合の動きをチェックするため、メールアドレスを変更しPUTリクエストを送ってみます。（ここでは例として「xyz@example.com」に変更）

下記のようなレスポンスが返ってくると思います。
```json
"Not Found"
```
データが存在しないので意図した通り、**Not Found** が返ってきました。

最後に、PUT時とほぼ変わりありませんが、Delete処理の流れをおさらいしてみましょう。
> 1. /usersへDELETEリクエストが送られてくる。
> 2. バリデーションチェック（emailが空、またはメールアドレスの形式が正しいか？）
> 3. バリデーションエラーの場合、処理を中断しエラーを返す（ステータス 400）
> 4. バリデーションがOKならデータベースへ接続しデータを検索する
> 5. データがなければ404を返す。データがあればデータを削除
> 6. 削除後にテキストを返す（ステータス 200）

これで、Delete処理ができましたので、CRUDが全て完了しました。

ただ、今のままだと型チェックや必須パラメータのチェックを行う処理と、データベース関連の処理が全て`main.go`に書かれているので、ちょっと管理がしづらいかと思います。

そこで、次のページでは、処理を分割し管理しやすいようにしたいと思います。
