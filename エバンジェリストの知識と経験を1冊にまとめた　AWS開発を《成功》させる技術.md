<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [クラウドアーテキクチャの資料](#%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89%E3%82%A2%E3%83%BC%E3%83%86%E3%82%AD%E3%82%AF%E3%83%81%E3%83%A3%E3%81%AE%E8%B3%87%E6%96%99)
- [Design for Failure](#design-for-failure)
- [アップデート形式](#%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88%E5%BD%A2%E5%BC%8F)
  - [All at Once](#all-at-once)
    - [メリット](#%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88)
    - [デメリット](#%E3%83%87%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88)
  - [Rolling](#rolling)
    - [メリット](#%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-1)
    - [デメリット](#%E3%83%87%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-1)
  - [Blue/Green](#bluegreen)
    - [メリット](#%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-2)
    - [デメリット](#%E3%83%87%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-2)
  - [Canary](#canary)
    - [メリット](#%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-3)
    - [デメリット](#%E3%83%87%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88-3)
- [アカウント](#%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88)
  - [AWSアカウント](#aws%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88)
  - [コンソールアカウント](#%E3%82%B3%E3%83%B3%E3%82%BD%E3%83%BC%E3%83%AB%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88)
  - [その他アカウント](#%E3%81%9D%E3%81%AE%E4%BB%96%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88)
  - [マルチアカウントアーキテクチャ](#%E3%83%9E%E3%83%AB%E3%83%81%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3)
  - [ルートユーザーの取り扱い](#%E3%83%AB%E3%83%BC%E3%83%88%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%81%AE%E5%8F%96%E3%82%8A%E6%89%B1%E3%81%84)
- [ルーティングプロトコル](#%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB)
  - [EGP](#egp)
  - [IGP](#igp)
- [ウェルノウンポート](#%E3%82%A6%E3%82%A7%E3%83%AB%E3%83%8E%E3%82%A6%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%88)
- [EC2インスタンスにmysqlインストール](#ec2%E3%82%A4%E3%83%B3%E3%82%B9%E3%82%BF%E3%83%B3%E3%82%B9%E3%81%ABmysql%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# クラウドアーテキクチャの資料

- [AWS Well-Architected Labs :: AWS Well-Architected Labs](https://www.wellarchitectedlabs.com/)
- [AWS Well-Architected Framework - AWS Well-Architected Framework](https://docs.aws.amazon.com/ja_jp/wellarchitected/latest/framework/welcome.html)

- [ホワイトペーパー | AWS](https://aws.amazon.com/jp/whitepapers)
- [AWS Trusted Advisor](https://aws.amazon.com/jp/premiumsupport/technology/trusted-advisor/)

# Design for Failure

サービスに異常や障害が発生することを念頭に設計の檀家来から考慮すべき。  
「障害が起こってもサービスが継続できる設計」だけでなくて、「障害が起こったときに短時間で復旧できる仕組みの設計」も大切。

# アップデート形式

## All at Once

すべてのサーバーに一括でアップデートを行う方法。

### メリット

- 追加リソース不要
- 短時間

### デメリット

- 切り戻しが大変
  - バックアップからの復元が必要
- サービス停止が発生

## Rolling

アップデート対象をグループに分けて、グループごとにアップデートを行う方法。

### メリット

- 追加リソース不要
- サービス停止なし

### デメリット

- 切り戻しが大変
  - All at Onceよりはマシ
  - アップデート前のリソースが残っている可能性があるため

## Blue/Green

アップデート対象となるリソースを新規に作成して、あるタイミングでアップデート後、古いリソースを削除する方法。

### メリット

- 切り戻しが簡単
  - 現環環境が残っているため
- サービス停止なし

### デメリット

- 追加リソースが必要
  - 2倍のリソースが必要
- 切り替えまでの準備に時間がかかる
- データベースの切り替えがネック
  - スキーマ変更のような破壊的変更では戻せない可能性があるため

## Canary

現環境のリソースにアップデートをしたりリソースを徐々に追加していって、段階的にアップデートしたリソースへと切り替えていく方法。

### メリット

Blue/Greenとほぼ同じ。違いは一部のユーザーに対してのみアップデートを行うことができる。

### デメリット

Blue/Greenとほぼ同じ。

# アカウント

## AWSアカウント

AWSとの契約単位。12桁の数字で表される。

- 管理アカウント
  - AWS Organizationsの管理者アカウント
  - 請求が行くため請求アカウントとも呼ばれる
- メンバーアカウント
  - 管理アカウント以外のAWSアカウント

## コンソールアカウント

AWSマネジメントコンソールにアクセスするためのアカウント。

- ルートユーザー
  - AWSアカウント作成時に登録したアドレスで認証するアカウント
  - 全AWSサービスリソースに完全なアクセス権限を持つ
- IAMユーザー
  - AWSマネジメントコンソールにアクセスするためのアカウント
  - ポリシーによってAWSサービスリソースへのアクセス権限を制限できる

## その他アカウント

OSやアプリケーションにアクセスするためのアカウント。

- OSアカウント
  - WindowsやLinuxなどOSレイヤへのアクセスに使用するアカウント
- システム利用者アカウント
  - AWS上に構築したシステムを利用するエンドユーザーに割り当てるアカウント

## マルチアカウントアーキテクチャ

「開発用と本番用でAWSアカウントを分ける」ようなAWSアカウントを複数用意してシステムを構築すること

![](./assets/スクリーンショット%202023-10-14%2021.48.38.png)

## ルートユーザーの取り扱い

最高権限を持っているので、取り扱いを考える必要がある。  
https://docs.aws.amazon.com/ja_jp/SetUp/latest/UserGuide/best-practices-root-user.html  
ルートユーザーにしかできないことがあるので、権限を丸投げすると危険なので、ルートユーザーを限定することが大切。

# ルーティングプロトコル

特定のIPアドレスを持つネットワークが追加された場合、自動でルートテーブルを更新する仕組み。  
郵便に例えると、「日本」や「東京都」のような大まかな情報をEGPで情報交換し、それ以降は各地域の郵便局がIGPで情報交換する ようなもの

## EGP

Exterior Gateway Protocol

ネットワークサービスプロバイダーやAWSのような「ある程度大きなネットワーク」はAS番号をもち、この番号をやり取りして「どのネットワークにどのネットワークが接続されているのか」を大まかにやりとりする。

## IGP

Interior Gateway Protocol

EGP内部のルーター同士でルートテーブルの情報をやり取りする。

# ウェルノウンポート

代表的なアプリケーションが使用するポート番号のこと。
例：
SSH = 22
HTTP = 80
HTTPS = 443

# EC2インスタンスにmysqlインストール

https://dev.mysql.com/downloads/repo/yum/  
Red Hat Enterprise Linux 9 / Oracle Linux 9 (Architecture Independent), RPM Package

```bash
sudo yum localinstall https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm

$ yum repolist
repo id                                                                               repo name
amazonlinux                                                                           Amazon Linux 2023 repository
kernel-livepatch                                                                      Amazon Linux 2023 Kernel Livepatch repository
mysql-connectors-community                                                            MySQL Connectors Community
mysql-tools-community                                                                 MySQL Tools Community
mysql80-community                                                                     MySQL 8.0 Community Server

sudo yum install mysql-community-server
```
