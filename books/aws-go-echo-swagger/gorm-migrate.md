---
title: "GORMでマイグレーションを実行する"
free: false
---

このページでは構造体を定義しGORMのマッピング機能を使ってマイグレーションを行います。
<!-- Step -->
:::details 手順だけ見たい方はこちら
1. 構造体を定義する
```diff go:main.go
package main

import (
  "time"

  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

+ // User 構造体を定義
+ type User struct {
+   ID        int       `gorm:"autoIncrement"`
+   Name      string    `gorm:"type:text;"`
+   Email     string    `gorm:"type:text; not null"`
+   CreatedAt time.Time `gorm:"not null; autoCreateTime"`
+   UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
+ }

+ // UserTodo 構造体を定義
+ type UserTodo struct {
+   ID         int       `gorm:"autoIncrement"`
+   UserId     int       `gorm:"type:int; not null"`
+   TodoName   string    `gorm:"type:text; not null"`
+   TodoStatus *bool     `gorm:"not null; default: 0"`
+   CreatedAt  time.Time `gorm:"not null; autoCreateTime"`
+   UpdatedAt  time.Time `gorm:"not null; autoUpdateTime"`
+ }

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"
  db_name := "golarn"

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/" + db_name + "?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

+  // マイグレーション実行
+  db.AutoMigrate(&User{}, &Todo{})

}

```
2. ```go run main.go```でマイグレーションを実行する
```
$ go run main.go
```
3. MampまたはXamppのphpMyAdminの画面で確認する
4. migrateディレクトリを作りそこにmain.goを複製してmigrate.goにリネームする
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go // main.goを複製してmigrate.goにリネーム
     ├─ go.mod
     └─ main.go
```
:::

## マイグレーションしてテーブルを作成する
マイグレーションをする前に、まずはどのようなテーブルを作るかを決める必要があります。
今回はシンプルに下記のようなテーブルを作成したいと思います。

##### テーブル名：users
| id | name | email | created_at | updated_at |
| ---- | ---- | ---- | ---- | ---- |
| int | text | text | timestamp | timestamp |

##### テーブル名：user_todos
| id | user_id | todo_name | todo_status | created_at | updated_at |
| ---- | ---- | ---- | ---- | ---- | ---- |
| int | int | text | bool | timestamp | timestamp |

**ユーザーがいて、そのユーザーに紐づくToDo（todo_name）があり、そしてToDoの完了・未完了の状態（todo_status）** がある。とゆう状態ですね。

テーブルが定義できましたら、構造体を作成していきます。
`main.go`を下記のように編集します。

```diff go:main.go
package main

import (
  "time"

  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

+ // User 構造体を定義
+ type User struct {
+   ID        int       `gorm:"autoIncrement"`
+   Name      string    `gorm:"type:text;"`
+   Email     string    `gorm:"type:text; not null"`
+   CreatedAt time.Time `gorm:"not null; autoCreateTime"`
+   UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
+ }

+ // UserTodo 構造体を定義
+ type UserTodo struct {
+   ID         int       `gorm:"autoIncrement"`
+   UserId     int       `gorm:"type:int; not null"`
+   TodoName   string    `gorm:"type:text; not null"`
+   TodoStatus *bool     `gorm:"not null; default: 0"`
+   CreatedAt  time.Time `gorm:"not null; autoCreateTime"`
+   UpdatedAt  time.Time `gorm:"not null; autoUpdateTime"`
+ }

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"
  db_name := "golarn"

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/" + db_name + "?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

+  // マイグレーション実行
+  db.AutoMigrate(&User{}, &Todo{})

}

```
`main.go`の編集が終わりましたら```go run main.go```を実行して、ブラウザで**phpMyAdminを開き確認してみましょう。**

**データベース：golarnの中に「users」と「user_todos」テーブル**が作成されています。
![データベース](https://storage.googleapis.com/zenn-user-upload/87180ac99d6d-20220223.png)
このようにGORMを使うと構造体でテーブル情報を定義し、それをもとにマイグレーションを行うことで簡単にテーブルを作ることができます。

ここで少しだけGoの構造体について触れておきます。
構造体は下記のように構成されています。
```go
type 構造体名 struct {
  フィールド名 型 `"タグ01" "タグ02"`
}
```
GORMを使い構造体でテーブルを作成する時は下記をご留意ください。

> 1. **構造体の名前やフィールド名はキャメルケースで書く。**
> 例：UserTodo、TodoName
> 2. **構造体名はマイグレーション時にスネークケースに変換され、テーブル名になる（複数形）**
> 例：UserTodo → user_todos
> 3. **複数のタグを設定する場合はスペース区切り**
> 例：gorm:"type:int"validate:"required" → gorm:"type:int" validate:"required"
> 4. **intやtextなどのゼロ値（0や""など）を扱う場合はポインタにする**
> 例：bool → *bool

:::message
ゼロ値など聴き慣れない言葉があるとは思いますが後ほど説明します。
:::
簡素ではありますが、GORMでの基本的なマイグーレションの方法をご紹介させてただきました。

最後に、先ほどのデータベース作成時と同じようにマイグレーションもテーブルを作った後は、頻繁に行うことはありませんので、別ファイルにしておきます。

ここでは新たに`migrate`ディレクトリを作り、その中に`main.go`ファイルを複製し、`migrate.go`にリネームしておきましょう。

```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go // main.goを複製してmigrate.goにリネーム
     ├─ go.mod
     └─ main.go
```

これで、マイグーレションが必要な時（テーブル作成時）は`migrate`ディレクトリに移動し実行するだけでよい環境ができました。（下記参照）
```
# ディレクトリを移動
$ cd ~/golarn/migrate/

# 実行する
go run migrate.go
```

以上でGORMでデータベースの作成とマイグレーションは完了です。

ここまでの作業で下記の懸念が生まれた方もおられると思います。

:::message alert
- データベース情報が変わったらinitdb.go、migrate.goの両方を変更するのは手間だ！
- そもそもデータベース情報をファイルに直書きするのは、危険！
:::

これはごもっともです。
特に、実際の現場ではデータベース情報をファイルに直書きする、なんてことはまずありません（Githubなどに上げてしまった場合は大惨事です。Yahooニュースに流れます）

これらを解決するために次に.envでからデータベース情報を読み込む方法を紹介します。
