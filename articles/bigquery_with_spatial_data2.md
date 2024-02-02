---
title: "Bigqueryで始める地理空間データ処理②〜応用編〜"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Bigquery", "Spatial Data", "GCP"]
published: false
---


こちらは、 **「** **Biguqeryで始める地理空間データ処理** **」** というタイトルの記事の続きです。
まだ前の記事を見ていない方はぜひ、前の記事も参考にしてください！

# TL;DR
- 実際に自分で集め、用意した地理空間データを利用し、Bigqueryで分析・可視化する
- 日本の都道府県別、震度5弱以上の地震の発生回数の可視化を行う（地震の発生回数といっても震源のみのデータを利用）
- Bigqueryで扱えない形式の地理空間データを扱える形に変換
- Bigquery Geo Vizを利用し、可視化


# はじめに
## 目的
- Biguqeryを利用し、都道府県別で震度5弱い嬢の地震が何回起きたかを可視化する

## 前提事項
- 有料契約をしているGCPプロジェクトがある
- BigQueryにアクセスできる権限がある
- Bigquery Geo Vizの認証が完了している
- 地理空間データについて基本的な理解がある
- [こちら](https://zenn.dev/aian1001/articles/bigquery_with_spatial_data)の記事に目を通している（推奨事項）


## 対象読者
- 地理空間データをCloudでも扱いたい方
- BigQueryを使って地理空間データを分析したい方
- 地理情報システム(GIS)に関連するプロジェクトを扱っている方
- 地理空間データをビジュアライズする方法を探している方

:::message
本記事では地理空間データがどのようなものかは説明しません。知りたい方は[こちら](https://zenn.dev/aian1001/articles/bigquery_with_spatial_data1)を参考してください。
:::

# 利用データ

# データ変換
## 震度データ
## 都道府県別のポリゴンデータ

# Bigqueryのテーブル作成

# クエリ作成・呼び出し

# Bigquery Geo Vizで可視化

# まとめ
