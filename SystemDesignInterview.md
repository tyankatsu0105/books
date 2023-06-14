<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Database の種類](#database-%E3%81%AE%E7%A8%AE%E9%A1%9E)
- [垂直と水平のスケーリング](#%E5%9E%82%E7%9B%B4%E3%81%A8%E6%B0%B4%E5%B9%B3%E3%81%AE%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%AA%E3%83%B3%E3%82%B0)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Database の種類

- リレーショナルデータベース
  - RDBMS または SQL データベースとも呼ばれる
    - MySQL
    - Oracle Database
    - PostgreSQL
  - データを表と行で表して保存
  - 異なるデータベースのテーブル間で結合操作可能
- ノンリレーショナルデータベース
  - NoSQL データベースとも呼ばれる
    - CouchDB
    - Neo4j
    - Cassandra
    - HBase
    - Amazon DynamoDB
  - 4 つのカテゴリに分類
    - key value store
    - Graph store
    - Column store
    - Document store
  - ジョイン操作がサポートされてない
  - 以下のときに適している
    - アプリケーションに超低レイテンシーが必要な場合
    - データが非構造化であるか、リレーショナルデータがない場合
    - データのシリアライズとデシリアライズだけが必要な場合
    - 大量のデータを保存する必要がある場合

## 垂直と水平のスケーリング

- スケールアップ
  - 垂直
  - サーバーのパワーをアップ
  - シンプル
  - トラフィックが少ないときは良い
  - 制限事項がある
    - 無制限にパワーアップできない
      - CPU とメモリの上限がある
    - フェイルオーバーや冗長性がない
      - フェイルオーバー
        - 稼働中のシステムで問題が生じてシステムやサーバーが停止してしまった際に、自動的に待機システムに切り替える仕組み
      - 1 つのサーバーダウンで死ぬ
- スケールアウト
  - 水平
  - サーバの数増やす
  - 大規模アプリケーションに適している
