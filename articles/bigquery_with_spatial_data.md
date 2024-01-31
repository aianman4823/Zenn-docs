---
title: "Bigqueryで始める地理空間データ処理"
emoji: "🗺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Bigquery", "Spatial Data", "GCP"]
published: true
---

# TL;DR
- Bigqueryを使用して地理空間データを処理する方法についての紹介
- Bigqueryでできることやできないこと、地理空間データの概念、GISソフトウェアの利用方法などについての説明
- 公式のチュートリアルを通じて、Bigqueryを使用して地理空間データを処理し、可視化する方法の紹介
- Bigqueryを利用することで、クラウドのパワーを活用しながら地理空間データの分析や可視化が可能となるのではと期待している


# はじめに
## 目的
- Bigqueryで基本的な地理空間データを行えるということを知りましたので、どのようなことができるのか検証し、利用ケースを考えていきたいと思います。


## 前提事項
- 有料契約をしているGCPプロジェクトがある
- BigQueryにアクセスできる権限がある
- Bigquery Geo Vizの認証が完了している
- 地理空間データについて基本的な理解がある


## 対象読者
- 地理空間データをCloudでも扱いたい方
- BigQueryを使って地理空間データを分析したい方
- 地理情報システム(GIS)に関連するプロジェクトを扱っている方
- 地理空間データをビジュアライズする方法を探している方


# 地理空間データとは
[IBMさんの記事](https://www.ibm.com/jp-ja/topics/geospatial-data)によると、地理空間データは、地表上または地表近くの場所にあるオブジェクト、イベント、またはその他の特徴を記述する情報で、通常、位置情報（通常、地球上の座標）と属性情報（関係する物体、事象、現象の特徴）に時間情報（位置と属性が存在する時間または寿命）を組み合わせたものだそうです。

自分の理解では、緯度や経度のような位置の情報と、その位置に付随する何かしらのデータを持っている情報のことを地理空間データと考えています。

代表的な地理空間データには、GPS（全地球測位システム）や空中写真データ、人工衛星等による観測データ、道路や河川などの台帳データ、人口や農業などの統計データなどがあります。

これらの地理空間データを実際分析するときには、**Geographic** **Information** **System** **(GIS)** という分析用のソフトウェアを一般的に利用します。

よく利用されるのが[QGIS](https://www.qgis.org/ja/site/about/index.html)や[ArcGIS](https://resources.arcgis.com/ja/help/getting-started/articles/026n00000014000000.htm)といった無料のものや有料のものが利用されます。

使い方としては、地理空間データをレイヤーと呼ばれる層として扱い、それらを重ね合わせることで空間的な解析を行います。

具体的なイメージとしては下記のようなものになります。

![image](/images/bigquery_with_spatial_data/001026409.gif)
*[国土交通省 地理空間情報ページより](https://www.mlit.go.jp/tochi_fudousan_kensetsugyo/chirikukannjoho/tochi_fudousan_kensetsugyo_tk1_000041.html)*


複数の地理空間データの層を重ねることで1つの情報だけでは発見できなかったインサイトを見つけることができるようになったりします。
複雑なものでは2桁のレイヤーを重ね合わせ分析したりします。

また、単純に重ね合わせるだけでなく、地理空間データの**投影変換**も必要になるケースが多いです。（今回は投影変換（空間座標変換）には触れません。興味がある方は[こちら](https://gis-oer.github.io/gitbook/book/materials/08/08.html)）

そして、上記のような地理空間データをなんとBigqueryでも行えるということを最近知りました（[資料](https://cloud.google.com/bigquery/docs/geospatial-data?hl=ja)）。

無料であるQGISといったGISソフトウェアは、自分のPCスペックに依存するものも多く、レイヤーが増えたり、重たい処理を実行した際、時間がかかったりフリーズしたりするので、重たい処理の実行などでCloudのパワーを利用できるのではと期待しています。

一方、調べを進めると、Bigqueryでは現状基本的な地理空間処理のみ行えるらしく、できるといってもできないこともあるといった状況のようです。

# Bigqueryでできること・できないこと 
次にBiguqeryでできること・できないことを表形式でまとめていきたいと思います。

下記表のような状況となっているようです。

| できること | できないこと |
| --- | --- |
| 地理空間データのストレージとクエリ | 複雑な地理空間分析の制限 |
| 標準的な地理空間関数のサポート | 地理空間データタイプの制限 |
| 大規模な地理空間データセットの分析 | 測地系の変更 |
| 他GCPサービスとの連携で地理空間データの可視化 (Looker Studio, BigQuery Geo Viz, Google Earth Engine) |  |

[こちら](https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions)がBigqueryでサポートされている地理空間関数一覧のドキュメントになりますが、逆に言えば、ここにない処理はBigqueryでは実行できないということになります。

例えば、地理空間データからメッシュ化した別の地理空間データの作成や複雑な複数の地理空間データ間関係処理を行うことができません。

複雑な処理などBigqueryにできないことは他のGISツールに頼り、できることはBigqueryに強力なコンピュートリソースを利用し処理していくというスタイルがいいかもしれないですね。

では次に実際にBigqueryでどのように地理空間データを扱うのか公式のチュートリアルを再現していきたいと思います。

# 地理空間データの処理・可視化（チュートリアル編）
今回は[こちら](https://cloud.google.com/bigquery/docs/geospatial-get-started
)のGoogle Cloud公式ドキュメントのチュートリアルに沿って地理空間分析を行っていきたいと思います。

## 利用データ
利用するデータは、Bigquery一般公開データセットに含まれる、NYCシティバイクの移動に関するデータセットである**citibike_stations**というテーブルを利用します。

biguqery-public-dataというプロジェクトの中の、new_york_citibikeというデータセットの中にあります。

![image](/images/bigquery_with_spatial_data/bigquery_console.png)
*一般公開データセットであるcitibike_stationsテーブル*

今回このテーブルのうち、
- longitude: ステーションの経度情報。-73.993934のように10進数表記で格納されている。
- latitude: ステーションの緯度情報。40.751551のように10進数表記で格納されている。
- num_bikes_available: レンタル可能な自転車の数情報。
の3つを利用し、利用可能な自転車の数を基準にステーションを可視化していきます。

また、今回緯度軽度の情報が10進数表記で表されていることから、特に変換処理などは必要ないが、よくあるケースで度分秒形式で緯度経度が格納されている場合は、10進数表記に変換する処理が必要となります。（今回はいらない）

では、次にBigqueryコンソールからクエリしましょう

## クエリ
今回、チュートリアルでは30台を超える自転車が利用可能な自転車ステーションのクエリを行います。

まずはBigqueryコンソール画面を開きます。
下記画像の赤枠のプラスボタンを押し、無題のページを作成します。

![image](/images/bigquery_with_spatial_data/bigquery_console1.png)
*Bigqueryコンソールでクエリ入力ができる状態*


上記画像のようになったのち、下記コードを上記ページにコピー&ペーストします。

```SQL
SELECT
  ST_GeogPoint(longitude, latitude)  AS WKT,
  num_bikes_available
FROM
  `bigquery-public-data.new_york.citibike_stations`
WHERE num_bikes_available > 30
```

このクエリのポイントは**ST_GeogPoint**関数で、この関数により、経度と緯度を引数に、**GEOGRAPHY型**呼ばれる形式に変換しています。
この変換ではGEOGRAPHY型のうち、**ポイント**（Point）と呼ばれるものに変換しています。

上記のような地理空間データに対して処理を行う関数を**地理関数** **(Geography function)** と呼びます。
[こちらのドキュメント](https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions)にBigqueryで利用可能な地理関数の一覧がまとまっています。

GEOGRAPHY型やポイントの説明は今回割愛しますが、気になる方は[こちら](http://cse.naro.affrc.go.jp/yellow/pgisman/3.0.0/using_postgis_dbmanagement.html#:~:text=%E3%82%B8%E3%82%AA%E3%82%B0%E3%83%A9%E3%83%95%E3%82%A3%E5%9E%8B%E3%81%AF%E3%80%81%E3%80%8C%E5%9C%B0%E7%90%86,%E3%81%95%E3%82%8C%E3%82%8B%E7%90%83%E9%9D%A2%E5%BA%A7%E6%A8%99%E3%81%A7%E3%81%99%E3%80%82)の資料を参考してください。


先ほどのクエリをBigqueryコンソールから実行した結果が下記画像になります。

![image](/images/bigquery_with_spatial_data/bigquery_console2.png)
*クエリを実行したBigquery画面*

こちらの画像のようにST_GeogPoint関数によって緯度経度の情報がPOINTという情報になっていることがわかると思います。

この形式になれば地図上にマッピングするようなレイヤーとして扱えます。

次にこちらのクエリを利用してマッピングしてみましょう

## マッピング
利用するツールは**Bigquery** **Geo** **Viz**というツールになります。

利用するには認証を行い、Bigqueryへのアクセス権を付与する必要があります。

また、このツールを開いたのち、
1. 利用しているGCPプロジェクトの選択
2. 実行したいクエリのコピー&ペースト
を行います。

行った後の画面が下記になります。

![image](/images/bigquery_with_spatial_data/bigquery_geo_viz.png)
*Bigquery geo vizに入力した後の画面*

このような状態になった後、RUNボタンを押すと下記のようにポイントが表示されます。

![image](/images/bigquery_with_spatial_data/bigquery_geo_viz1.png)
*Bigquery geo vizでRUNした後の画像*

このように赤い点が表示されている箇所が利用可能な自転車が30台以上あるステーションになります。

また、下記画像のStyleという箇所からポイントの色や大きさをデータによって変えたりすることができます。
今回は色はそのままに、ポイントの大きさを利用可能な自転車台数によって変えるようにしたいと思います。

![image](/images/bigquery_with_spatial_data/bigquery_geo_viz2.png)
*Bigquery geo vizのStyle*

利用するのはStyleの中の**circleRadius**という要素です。
手順としては
1. **Data-driven**をONにする
2. Functionに**linear**を選択
3. Fieldに**num_bikes_available**を選択
4. Domainの左枠に31、右枠に101を入力（maxというのが右枠の右に出るのでその値を参考に入れる数値を決める）
5. Rangeの左枠に20、右枠に100を入力
で、上記を実施したのち、**Apply** **Style**ボタンをクリックします。


実行した結果が下記の画像になります。

![image](/images/bigquery_with_spatial_data/bigquery_geo_viz3.png)
*Bigquery geo vizでStyleをApplyした後の結果*

このように、データに応じて色や円の大きさなど変えることができます。

# まとめ
- Bigqueryでも基本的な地理空間データを扱うことはできる。
- 測地系を変えられないなどの制約があるため、他のGISツールと組み合わせて利用するなどの工夫が必要そう。
- BigQuery Geo Vizを使用すると、クエリだけで地理データを可視化することができて便利。

次回は、少し応用で日本の都道府県別、地震の発生回数をBigqueryを利用して分析したいと思います！