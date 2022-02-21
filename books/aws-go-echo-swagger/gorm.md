---
title: "DB作成とGORMでのマイグレーション"
free: false
---

このページではGORMをインストールして、DBを作成します。
次に、構造体を定義しその構造体を基にマイグレーションを行います。


## GORMのインストール
コマンドラインでGORMをインストールします。
つづいて、今回はデータベースにMySQLを使用しますのでドライバもインストールしておきます。

```
$ go get github.com/jinzhu/gorm
$ go get gorm.io/driver/mysql
```

## 接続確認
インストールが完了しましたらまずは接続チェックを行います。
MampまたはXamppを起動してください。

:::message
この先はMampまたはXamppを起動しっぱなしにしておいて下さい。
:::

【main.go】を下記のように編集します。
ユーザー名はやパスワードなどはご自身の環境に合わせてご変更ください。
```go:main.go
package main

import (
  "fmt"

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

ちなみにエラーの場合は下記のようなメッセージが表示されます。

**※パスワードをわざと間違えた例**
```
[error] failed to initialize database, got error Error 1045: Access denied for user 'root'@'localhost' (using password: YES)
panic: Error 1045: Access denied for user 'root'@'localhost' (using password: YES)
```
```panic: Error ~``` と書かれてる部分が【main.go】に記載した失敗時の処理ですね。

## データベースを作成する
接続の確認ができましたので、【main.go】をにデータベース名の情報を追記し、データベースを作成します。

```go:main.go
package main

import (
  "fmt"

  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {

  // DB接続情報
  db_user := "root"
  db_pass := "root"
  db_host := "localhost"
  db_port := "3306"
  db_name := "golarn" // DB名を追加

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/?charset=utf8mb4&parseTime=True&loc=Local"

  // 接続開始
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  // 接続失敗時はエラーを出力し、成功時は何も出力しない。
  if err != nil {
    panic(err.Error())
  }

  // データベース：golarn を作成
  exec := db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)

}
```
```db.Exec("CREATE DATABASE IF NOT EXISTS " + db_name)``` の部分で、**「もし指定されたDBがなければ新規に作成する」** とゆうプログラムを実行しています。すでにDBがある場合は何も処理をしません。

これでDBへの接続や作成はできました。ですが、何もテーブルがありません。
**構造体を定義し上記のDBにテーブルを作成（マイグレーション）**していきます。

その前に、現在の【main.go】はDBの接続確認と生成を行うだけですので、毎回行う必要はありません。別の場所に移動させて必要な時に実行できるようにしておきます。

ここでは新たに【initdb】とゆうディレクトリを作りそこに【main.go】をコピーしファイル名も【initdb.go】に変更しておきます。

```
~/
 └─ golean/
     ├─ initdb
         └─ initdb.go
     ├─ go.mod
     └─ main.go
```

これで、データベースの接続・作成が必要な場合のみ【initdb】ディレクトリに移動し実行すればよいとゆう情態にできました（下記参照）
```
# ディレクトリを移動
$ cd ~/golarn/initdb/

# 実行する
go run initdb.go
```

## マイグレーションしてテーブル作成する
構造体を定義しDBにテーブルを作成していきます。
今回作成するテーブルは下記のようにシンプルな構成にしたいと思います。

##### テーブル名：users
| id | name | created_at | updated_at |
| ---- | ---- | ---- | ---- |
| int | text | timestamp | timestamp |

##### テーブル名：user_todos
| id | user_id | todo_name | todo_status | created_at | updated_at |
| ---- | ---- | ---- | ---- | ---- | ---- |
| int | int | text | bool | timestamp | timestamp |

**ユーザーがあり、そのユーザーに紐づくToDo（todo_name）があり、そしてToDoの完了・未完了の状態（todo_status）** がある。とゆう状態ですね。

テーブルが定義できたら構造体を作成していきます。
【maing.go】を編集します。

```go:main.go
package main

import (
  "time"

  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)
// User 構造体を定義
type User struct {
  ID        int       `gorm:"autoIncrement"`
  Name      string    `gorm:"type:text; not null"`
  CreatedAt time.Time `gorm:"not null; autoCreateTime"`
  UpdatedAt time.Time `gorm:"not null; autoUpdateTime"`
}

// UserTodo 構造体を定義
type UserTodo struct {
  ID         int       `gorm:"autoIncrement"`
  UserId     int       `gorm:"type:int; not null" validate:"required"`
  TodoName   string    `gorm:"type:text; not null"`
  TodoStatus *bool     `gorm:"not null; default: 0"`
  CreatedAt  time.Time `gorm:"not null; autoCreateTime"`
  UpdatedAt  time.Time `gorm:"not null; autoUpdateTime"`
}

func main() {

  db_user := "root"
  db_pass := "root"
  db_host := "localhost"
  db_port := "3306"
  db_name := "golarn"

  dsn := db_user + ":" + db_pass + "@tcp(" + db_host + ":" + db_port + ")/" + db_name + "?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

  if err != nil {
    panic(err.Error())
  }

  // マイグレーション実行
  db.AutoMigrate(&User{}, &Todo{})

}

```
【main.go】の編集が終わりましたら```go run main.go```を実行して、ブラウザで**MampのphpMyAdminを開きデータベースとテーブルを確認**してみましょう。

**データベース：golarnの中に「users」と「todos」テーブルが作成**されているはずです。
＜ここに画像が入る＞

構造体は下記のような構成で作成されます。
```go
type 構造体名 struct {
  フィールド名 型 `"タグ01" "タグ02"`
}
```
GORMを使い構造体でテーブルを作成する時は下記をご留意ください。

1. **構造体の名前やフィールド名はキャメルケースで書く。**
例：UserTodo、TodoName
2. **構造体名はスネークケースに変換され、テーブル名になる（複数形）**
例：UserTodo → user_todos
3. **複数のタグを設定する場合はスペース区切り**
例：gorm:"type:int"validate:"required" → gorm:"type:int" validate:"required"
4. **intやtextなどのゼロ値（0や""など）を扱う場合はポインタにする**
例：bool → *bool

簡単にDBへテーブルを作ることができました。
以上。GORMでのDB接続とマイグーレションの方法でした。

私はここまでの時点でGoの魅力に惹かれました。
皆さんはどうでしょうか？

今回は構造体をもとにテーブルを作成することをメインとしましたが、後半では構造体のタグを追記しバリデーションやクエリ情報の取得などを行います。

その時にGoの便利さを改めて感じていただければと思います。