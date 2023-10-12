<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents** _generated with [DocToc](https://github.com/thlorenz/doctoc)_

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
