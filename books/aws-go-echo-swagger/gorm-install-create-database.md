---
title: "GORMをインストールしてデータベースを作成する"
free: false
---

このページではGORMをインストールして、データベースを作成します。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. ##### GORMとMySQL用のドライバをインストール
```
$ go get github.com/jinzhu/gorm
$ go get gorm.io/driver/mysql
```
2. ##### MySQLへの接続確認。`main.go`を下記のように編集し`go run main.go`で実行
（事前にMampまたはXamppを起動）
```go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  _, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

}
```
3. ##### データベースを作成
```diff go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"
+ db_name := "golarn" // DB名を追加

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

+  // データベース：golarn を作成
+  exec := db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)
+  fmt.Println(exec)

}
```
4. ##### MampまたはXamppのphpMyAdminの画面で確認する
5. ##### initdbディレクトリを作りそこにmain.goを複製してinitdb.goにリネームする
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go // main.goを複製してinitdb.goにリネーム
     ├─ go.mod
     └─ main.go
```

:::
<!-- Step -->

## GORMとは？
GoのORMです。
ORMとは、（Object-Relational Mapping）の頭文字をとったもので、その名前からわかる通り、オブジェクトと関係（関係データベース、RDB）とのマッピングを行うものです。
https://gorm.io/ja_JP/docs/index.html

## GORMのインストール
まずは、コマンドラインでGORMをインストールします。
今回はデータベースにMySQLを使用しますのでいっしょにドライバもインストールしておきます。

```
$ go get github.com/jinzhu/gorm
$ go get gorm.io/driver/mysql
```

## 接続確認
インストールが完了しましたらまずはローカルのMySQLに接続してみましょう。
事前にMampまたはXamppを起動しておいてください。

:::message
この先はMampまたはXamppを起動しっぱなしにしておいて下さい。
:::

```main.go```を下記のように編集します。
ユーザー名やパスワードなどはご自身の環境に合わせてご変更ください。
```go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  _, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

}
```

```main.go```を編集したらコマンドラインに```go run main.go``` と入力し実行します。
接続成功時の処理は何も書いていないので実行後に何も返って来なければ接続OKです。
```
$ go run main.go
#何も返って来なければOK
```

ちなみにエラーの場合は下記のようなメッセージが表示されます（パスワードを間違えた例）
```
[error] failed to initialize database, got error Error 1045: Access denied for user 'root'@'localhost' (using password: YES)
panic: Error 1045: Access denied for user 'root'@'localhost' (using password: YES)
```
**panic: Error ~** と書かれてる部分が```main.go```に記載した失敗時の処理ですね。

## データベースを作成する
接続の確認ができましたので、次はデータベースを作成します。

```diff go:main.go
package main

import (
  "fmt"

  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // データベース情報
  db_user := "root"
  db_pass := "1234"
  db_host := "localhost"
  db_port := "3306"
+ db_name := "golarn" // DB名を追加

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

+  // データベース：golarn を作成
+  exec := db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)
+  fmt.Println(exec)

}
```
`main.go`の編集ができましたら実行します。
```
# main.goを実行する
$ go run main.go
```

```db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)``` の部分で、**「もしデータベースがなければ新規に作成する」** のプログラムを実行しています。すでにある場合は何も処理をしません。

:::message
ブラウザでphpMyAdminを開いてデータベース **「golarn」** が作られているかご確認ください。
:::

データベースの作成が成功すると下記のような文字列がコマンドラインに表示されていると思いますが、これは無視しておいて下さい。
```
&{0xc0000d42d0 <nil> 1 0xc0001ae1c0 0}
```

これでデータベースへの接続や作成はできました。ですが、まだ何もテーブルがありません。
次のページで**構造体を定義しテーブルを作成（マイグレーション）** していきます。

その前に、現在の`main.go`はデータベースの接続確認と作成を行うだけの処理ですので、作成後はデータベースが削除されない限りは実行する必要はありません。

別の場所に移動させて必要な時に実行できるようにしておきましょう。

ここでは新たに`initdb`ディレクトリを作り、その中に`main.go`ファイルを複製し、`initdbe.go`にリネームしておきます。
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go // main.goを複製してinitdb.goにリネーム
     ├─ go.mod
     └─ main.go
```

これで、データベースの接続・作成が必要な時は`initdb`ディレクトリに移動し、実行するだけでよい環境ができました。（下記参照）
```
# ディレクトリを移動
$ cd ~/golarn/initdb/

# 実行する
go run initdb.go
```

では、、構造体を定義しテーブルを作成（マイグレーション）する作業に進みましょう。

