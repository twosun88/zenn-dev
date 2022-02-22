---
title: "GORMをインストールしてデータベースを作成する"
free: false
---

このページではGORMをインストールして、DBを作成します。

<!-- Step -->
:::details 手順だけ見たい方はこちら
1. GORMとMySQL用のドライバをインストール
```
$ go get github.com/jinzhu/gorm
$ go get gorm.io/driver/mysql
```
2. MySQLへの接続確認を行います。main.goを下記のように編集し```go run main.goで```実行
※事前にMampまたはXamppを起動しておくこと。
```go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // DB接続情報
  db_user := "root"
  db_pass := "root"
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
3. データベースを作成する
```diff go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // DB接続情報
  db_user := "root"
  db_pass := "root"
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

}
```
4. MampまたはXamppのphpMyAdminの画面で確認する
5. initdbディレクトリを作りそこにmain.goを複製してinitdb.goにリネームする
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
つづいて、今回はデータベースにMySQLを使用しますのでドライバもインストールしておきます。

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

【main.go】を下記のように編集します。
ユーザー名やパスワードなどはご自身の環境に合わせてご変更ください。
```go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // DB接続情報
  db_user := "root"
  db_pass := "root"
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

【main.go】を編集したら ```go run main.go``` で実行します。
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
```panic: Error ~``` と書かれてる部分が【main.go】に記載した失敗時の処理ですね。

## データベースを作成する
接続の確認ができましたので、【main.go】をにデータベース名の情報を追記し、データベースを作成します。

```diff go:main.go
package main

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // DB接続情報
  db_user := "root"
  db_pass := "root"
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

}
```
```db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)``` の部分で、**「もし指定されたDBがなければ新規に作成する」** とゆうプログラムを実行しています。すでにDBがある場合は何も処理をしません。

:::message
ブラウザでMampのphpMyAdminを開いてデータベース【golarn】が作られているかご確認ください。
:::

これでDBへの接続や作成はできました。ですが、何もテーブルがありません。
**構造体を定義しテーブルを作成（マイグレーション）** していきます。

その前に、現在の【main.go】はデータベースの接続確認と作成を行うだけの処理ですので、作成後は対象のデータベースが削除されない限りは実行する必要はありません。

別の場所に移動させて必要な時に実行できるようにしておきましょう。

ここでは新たに【initdb】ディレクトリを作り、その中に【main.go】ファイルを複製し、【initdbe.go】にリネームしておきます。
```
// 現時点でのディレクトリ構成
~/
 └─ golean/
     ├─ initdb/
         └─ initdb.go // main.goを複製してinitdb.goにリネーム
     ├─ go.mod
     └─ main.go
```

これで、データベースの接続・作成が必要な時は【initdb】ディレクトリに移動し実行するだけでよい環境ができました。（下記参照）
```
# ディレクトリを移動
$ cd ~/golarn/initdb/

# 実行する
go run initdb.go
```

では、、次のページで構造体を定義しマイグレーションを行なっていきます。

