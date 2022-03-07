---
title: "はじめに"
free: true
---

本書を手に取っていただきありがとうございます。
この本は「Goを使ってREST APIサーバーを作り、Postmanで動作確認する」までをまとめたものです。

## なぜ、この本を書いたのか？
私自身、主にフロントエンドエンジニアとしてお仕事をさせていただいています。
昨今のフロントエンド周りの技術やフレームワークの移り変わりはとても激しくて日々の学習が欠かせません。

その際に無料のAPIサービス（jsonplaceholderなど）だけではバックエンドとの連携処理（CRUD処理など）が学習しづらいと感じたため、自分でAPIを作りより理解を深めたいと思ったのがきっかけです。

私と同じような思いを感じている方がいるかもしれないと思い、この度、本にまとめさせていただきました。

## なぜ、Goなのか？
APIを作るにあたり、Ruby on Rails、Laravel、Djangoなど様々な言語またはフレームワークを試してみましたがどれも私にはしっくり来ず、Goを試してみたところ、その厳格さとシンプルさに惹かれたのがGo言語を選んだ理由です。

:::message alert
本書では「APIサーバー」を作ることを目的としていますので、型や変数や構造体などのGoの基礎構文の詳細の説明はしておりません。Goの基礎構文の学習を終えてた後に読むことをお勧めします。
:::

## なぜ、有料なの？
学習意欲が高い人だけを対象にしているためです。

## 使用する主なライブラリやツールなど
- シェル：bash
- Go v1.17
- GORM v1.9.16（GoのORMライブラリ）
- Echo v4.6.3（GoのWebフレームワーク）
- Air v1.27（ホットリロード用）
- godotenv v1.4.0（.envから環境変数を読み込む用）
- MySQL v5.7.30~
- MAMPまたはXAMPP（データベースの確認用）
- Postman（APIの動作検証用）
- Visual Studio Code（エディター）
- ターミナル系のソフト（コマンドを使います）

:::message
データベースの確認やAPIの動作検証にはGUIを使いコマンドラインが不慣れな方にも、進めやすい構成にしてあります。
:::

## 本書ではやらないこと
リモートサーバーへのGoのインストールや設定などは、サーバー毎に方法が異なるため、本書では行いません。あくまで、ローカルでの開発に留めております。予めご了承ください。

## 本書の構成
- Goのインストールとセットアップ
- GORMをインストールしてデータベースを作成する
- GORMでマイグレーションを実行する
- データベース情報をgodotenvで.envから読み込む
- Echoのインストールとセットアップ
- 構造体とデータベースに接続する処理を使い回しできるようにする
- Create（POST）の実装
- Read（GET）の実装
- Update（PUT）の実装
- Delete（DELETE）の実装
- コントローラーとモデルに分割する
- バイナリファイルを作る

:::message alert
本書では主にGoのプログラムとコマンドラインを使って勧めていきます。
先頭に "$" が付いているものはコマンドで実行するコードです。
:::