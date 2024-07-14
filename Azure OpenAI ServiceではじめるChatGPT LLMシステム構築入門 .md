<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents** _generated with [DocToc](https://github.com/thlorenz/doctoc)_

- [生成AI](#%E7%94%9F%E6%88%90ai)
- [出力形式](#%E5%87%BA%E5%8A%9B%E5%BD%A2%E5%BC%8F)
- [Code Interpreter](#code-interpreter)
  - [Open Interpreter](#open-interpreter)
- [ハルシネーションの回避に注意](#%E3%83%8F%E3%83%AB%E3%82%B7%E3%83%8D%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E5%9B%9E%E9%81%BF%E3%81%AB%E6%B3%A8%E6%84%8F)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 生成AI

大量のデータを学習することで、新たなコンテンツが生成できるようになったAIのこと

# 出力形式

ChatGPTが出力をjsonで返すことができるように、決まった形式で出力できると、そのままその結果をアプリケーションで組み込むことが簡単になる

# Code Interpreter

- ChatGPTに指示与える
- ChatGPTがその指示を解釈し、コードを生成
- 生成されたコードを実行
- 実行結果の解釈
- 追加で必要なコードを生成
- 生成されたコードを実行 ...
- 繰り返し

## Open Interpreter

Code Interpreterのオープンソース版。ローカルPC上でLLMと対話しながらLLMにコードが自校できる

# ハルシネーションの回避に注意

GPTは「確率的に確からしい返答」を行うので、実際に存在しない返答を行うことがある。これをハルシネーション（幻想）と呼ぶ。

- ユーザーがGPTの返答の妥当性を検証する
- 外部情報を参照して返答させるようなプロンプトエンジニアリングを行う

これらで回避を検討する
