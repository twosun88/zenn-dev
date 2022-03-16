---
title: "構造体とデータベースに接続する処理を使い回しできるようにする"
free: false
---

このページでは今後の作業のために、構造体とデータベースに接続する処理を、別ファイルに移動し使い回せるようにしておきます。

## 構造体を別ファイルに移す
**「GORMでマイグレーションを実行する」** のページで`migrate.go`に定義した **「User、UserTodo」** の2つの構造体ですが、今後は他のファイルからも利用することになりますので、別ファイルにしておきます。

ここでは、`structs`ディレクトリを作成しその中に`structs.go`を新たに作成します。
```
// 現時点でのディレクトリ構成
~/
 └─ golearn/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ structs/
         └─ structs.go // 構造体は全てこのファイルにまとめる
     ├─ go.mod
     └─ main.go
```

`structs.go`の中に構造体を移動します。コードに特に変更はありません。

```go:/structs/structs.go
package structs

import "time"

type User struct {
  ID        int       `gorm:"autoIncrement"`
  Name      string    `gorm:"type:text;"`
  Email     string    `gorm:"type:text; not null"`
  CreatedAt time.Time `gorm:"not null; autoCreateTime"`
  UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
}

type UserTodo struct {
  ID         int       `gorm:"autoIncrement"`
  UserId     int       `gorm:"type:int; not null"`
  TodoName   string    `gorm:"type:text; not null"`
  TodoStatus *bool     `gorm:"not null; default: 0"`
  CreatedAt  time.Time `gorm:"not null; autoCreateTime"`
  UpdatedAt  time.Time `gorm:"not null; autoUpdateTime"`
}
```

## データベースへ接続する処理を関数化しておく
つづいて、データベースへ接続する処理はCRUD処理の時に多用していくことになるので、こちらも別のファイルへ移動して使いやすくしておきます。
ここでは、`utilities`ディレクトリを作成しその中に`utilities.go`を新たに作成します。
```
// 現時点でのディレクトリ構成
~/
 └─ golearn/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ structs/
         └─ structs.go
     ├─ utilities/
         └─ utilities.go // よく使用する関数などをこのファイルに記述
     ├─ go.mod
     └─ main.go
```

データベースの接続処理を外部関数にして他のファイルからも利用できるように、`utilities.go`に下記のように記述します。
```go: utilities/utilities.go
package utilities

import (
  "os"

  "github.com/joho/godotenv"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

// データベースへの接続処理の関数
func DB() (*gorm.DB, error) {

  err := godotenv.Load("../.env")

  if err != nil {
    panic("Error loading .env file")
  }

  user := os.Getenv("DB_USER")
  pass := os.Getenv("DB_PASS")
  host := os.Getenv("DB_HOST")
  port := os.Getenv("DB_PORT")
  name := os.Getenv("DB_NAME")

  dsn := user + ":" + pass + "@tcp(" + host + ":" + port + ")/" + name + "?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  if err != nil {
    return db, err
  }

  return db, err
}
```
:::message alert
■構造体や関数の命名について
Goでは定義した構造体や関数を外部から利用できるようにするには、**頭文字を大文字**にする必要があります。
:::

では、今作成した`structs.go`と`utilities.go`を読み込み、動作するように`migrate.go`を編集します。
```diff go:migrate/migrate.go
package main

import (
+  "example-golearn/structs"
+  "example-golearn/utilities"

-  "os"
-  "github.com/joho/godotenv"
-  "gorm.io/driver/mysql"
-  "gorm.io/gorm"
)

func main() {

-  err := godotenv.Load("../.env")
-  if err != nil {
-    panic("Error loading .env file")
-  }

-  user := os.Getenv("DB_USER")
-  pass := os.Getenv("DB_PASS")
-  host := os.Getenv("DB_HOST")
-  port := os.Getenv("DB_PORT")
-  name := os.Getenv("DB_NAME")

-  dsn := user + ":" + pass + "@tcp(" + host + ":" + port + ")/" + name + "?charset=utf8mb4&parseTime=True&loc=Local"

+  // utilitis.goの中の関数を使用
+  db, err := utilities.DB()

  if err != nil {
    panic(err.Error())
  }

+  // structs.goの中の構造体を使用してマイグレーションを行う
+  db.AutoMigrate(&structs.User{}, &structs.UserTodo{})

}

```
これで、構造体とデータベースへの接続処理を使い回しできるようになりました。

念のため、動作確認してみましょう。
データベースから「usersとuser_todos」テーブルを削除します。そして、マイグレーションを実行後に再度この2つのテーブルが作られているかを確認して下さい。

```
#usersとuser_todosテーブルを削除した後に`migrate.go`を実行する
$ go run migrate.go
```

これで、環境は整えましたのでCRUD処理を実装していきます。
