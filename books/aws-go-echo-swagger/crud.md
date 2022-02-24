---
title: "CRUDを実装する"
free: false
---

このページではCRUDの実装を行い、Postmanで動作確認を行います。

## CRUDとは？
**Create, Read, Update, Delete** の頭文字を取ったデータベースを操作する上での基本的な機能のことを指します。
REST APIの設計モデルと照らし合わせると下記の図のようになります。
| ブラウザからのリクエスト | データベースの処理（CRUD） | 具体的な処理 |
| ---- | ---- | ---- |
| Post | Create | データを登録する |
| Get | Read | データを読み込む |
| Put | Update | データを更新する |
| Delete | Delete | データを削除する |

:::message
GETとPOSTリクエストだけでも上記のことはできますが、今回はRESTfull APIを作成なので、「POST、GET、PUT、DELETE」の全てを使います。
:::

## Post （ Create ）の実装
まずは、ユーザーのデータありきなので、ユーザー登録できるように「users」テーブルへのPost（Create）できる処理を作ります。

リクエスト時に「名前」をデータ送信し、「users」テーブルの「name」列にデータを登録します。
:::message
id、created_at、updated_atは自動的に設定されます。
:::
では、`main.go`を編集します。