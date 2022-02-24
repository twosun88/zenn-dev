---
title: "ディレクトリ構造の整理"
free: false
---

このページでは今後の開発のために、構造体を別ファイルにし他のファイルから利用できるようにしておきます。

## 構造体を別ファイルにする
**「GORMでマイグレーションを実行する」** のページで`migrate.go`に定義した **「User、UserTodo」** の2つの構造体ですが、今後は他のファイルからも利用することになりますので、別ファイルにしておきます。

ここでは、`structs`ディレクトリを作成しその中に`structs.go`を新たに作成します。
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ structs/
         └─ structs.go // 構造体は全てにここにまとめる
     ├─ go.mod
     └─ main.go
```

`structs.go`の中に構造体を移動します。

```go:/structs/structs.go
package structs

import "time"

type User struct {
  ID        int       `gorm:"autoIncrement"`
  Name      string    `gorm:"type:text; not null"`
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

これで、構造体 **「User、UserTodo」** を`structs.go`から呼び出し利用できるようになりました。

同時に`migrate.go`に直書されている構造体は削除して、`structs.go`から構造体を読み込み実行できるようにしておきましょう。

```diff go:main.go
package main

import (
  "os"
- "time"

+  // structs.goを読み込む（go.modに記載されているmodule名からのパスで指定する）
+  "example-golarn/structs"

  "github.com/joho/godotenv"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

- type User struct {
-  ID        int       `gorm:"autoIncrement"`
-  Name      string    `gorm:"type:text; not null"`
-  CreatedAt time.Time `gorm:"not null; autoCreateTime"`
-  UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
- }

- type UserTodo struct {
-  ID         int       `gorm:"autoIncrement"`
-  UserId     int       `gorm:"type:int; not null"`
-  TodoName   string    `gorm:"type:text; not null"`
-  TodoStatus *bool     `gorm:"not null; default: 0"`
-  CreatedAt  time.Time `gorm:"not null; autoCreateTime"`
-  UpdatedAt  time.Time `gorm:"not null; autoUpdateTime"`
- }

func main() {

  // .envファイルを読み込む（相対パスで指定）
  err := godotenv.Load("../.env")

  // 読み込みに失敗時の処理
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
    panic(err.Error())
  }
+  //structs.goの構造体を利用するように変更
+  db.AutoMigrate(&structs.User{}, &structs.UserTodo{})

}
```
これで、`migrate.go`でのマイグレーション時に`structs.go`の構造体を利用できるようになりました。

動作確認する場合は、データベースから「usersとuser_todos」テーブルを削除して、マイグレーションを実行後に再度「usersとuser_todos」テーブルが作られているかを確認して下さい。
:::message alert
■構造体や関数の命名について
Goでは定義した構造体や関数を外部から利用できるようにするには、**頭文字を大文字**にする必要があります。
:::

では、CRUD処理を実装していきます。
