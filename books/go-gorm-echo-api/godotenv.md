---
title: "データベース情報を.envから読み込む"
free: false
---

このページではデータベース情報を.envファイルに設定しgodotenvを使って読み込み方法を説明します。

## godotenvとは？
.envファイルから環境変数を読み込むためのGoのパッケージです。
https://github.com/joho/godotenv

## godotenvのインストール

まずは、ターミナルで下記コマンドを実行しgodotenvをインストールします。
```
$ go get github.com/joho/godotenv
```
## .envファイルの作成
次に、.envファイルを作成します。
```
// 現時点でのディレクトリ構成
~/
 └─ golearn/
     ├─ initdb/
         └─ initdb.go
     ├─ migrate/
         └─ migrate.go
     ├─ .env // 追加
     ├─ go.mod
     └─ main.go
```
つづいて.envファイルに下記のようにデータベース情報を、環境変数として記述します。
```:.env
DB_HOST = localhost
DB_PORT = 3306
DB_USER = root
DB_PASS = root
DB_NAME = golearn
```
あとは、データベースのユーザー名などをこの.envファイルから呼び出すだけで、他は特に変わりありません。まずは、`initdb.go`を編集してみましょう。

```diff go:initdb/initdb.go
package main

import (
  "fmt"
+  "os"

+  "github.com/joho/godotenv"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

+  // .envファイルを読み込む（相対パスで指定）
+  err := godotenv.Load("../.env")

+  // 読み込みに失敗時の処理
+  if err != nil {
+    panic("Error loading .env file")
+  }

-  // 今まで使用していたデータベース情報は削除する
-  db_user := "root"
-  db_pass := "root"
-  db_host := "localhost"
-  db_port := "3306"
-  db_name := "golearn"

+  // .envファイルから環境変数を読み込む
+  db_user := os.Getenv("DB_USER")
+  db_pass := os.Getenv("DB_PASS")
+  db_host := os.Getenv("DB_HOST")
+  db_port := os.Getenv("DB_PORT")
+  db_name := os.Getenv("DB_NAME")

+  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

  // データベース：golearn を作成
+  exec := db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)
  fmt.Println(exec)

}
```
確認してみましょう。
まずは、ブラウザで**phpMyAdminを開きデータベース「golearn」を削除します。**
次に`initdb.go`を実行します。
```
# initdb/ ディレクトリに移動し下記を実行。
$ go run initdb.go
```
実行後、再度ブラウザでphpMyAdminを開きデータベース「golearn」があるか確認します。
作成されていれば.envからデータベース情報を正常に読み取りプログラムを実行できたことになります。

同様の処理を`migrate.go`に行います。
```diff go:migrate/migrate.go
package main

import (
  "os"
  "time"

  "github.com/joho/godotenv"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

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

func main() {

-  // 今まで使用していたデータベース情報は削除する
-  db_user := "root"
-  db_pass := "root"
-  db_host := "localhost"
-  db_port := "3306"
-  db_name := "golearn"

+  // .envファイルを読み込む（相対パスで指定）
+  err := godotenv.Load("../.env")

+  // 読み込みに失敗時の処理
+  if err != nil {
+    panic("Error loading .env file")
+  }

+  // .envファイルから環境変数を読み込む
+  user := os.Getenv("DB_USER")
+  pass := os.Getenv("DB_PASS")
+  host := os.Getenv("DB_HOST")
+  port := os.Getenv("DB_PORT")
+  name := os.Getenv("DB_NAME")

+  dsn := user + ":" + pass + "@tcp(" + host + ":" + port + ")/" + name + "?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

  // マイグレーション実行
  db.AutoMigrate(&User{}, &UserTodo{})

}

```
同じように一度削除して、再度作ってみましょう。
ブラウザでphpMyAdminを開き「usersとuser_todos」テーブルを削除します。
次に`migrate.go`を実行します。
```
# migrate/ ディレクトリに移動し下記を実行。
$ go run migrate.go
```
再度、ブラウザでphpMyAdminを開き「usersとuser_todos」テーブルがあるか確認します。
作成されていればこちらも正常にプログラムを実行できたことになります。

これで、データベースの情報を`initdb.go`や`migrate.go`2つのファイルに直書きすることはなくなり、.envファイルから読み取ることができるようになりました。

これで、先程の懸念である **「変更があった場合の手間、直書きの危険性」** に対応できたかと思います。

:::message alert
.envファイルはGithubなどには決してアップしないように気をつけて下さい。
:::

もしも、今後データベースのユーザー名やパスワードが変更になった場合は.envファイルを編集するだけでよいので管理も楽になりましたね。

以上で、データベース関連の基本的な設定は終了です。
次は、Echoを使ってWebサーバーを作ります。