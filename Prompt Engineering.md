<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Output length](#output-length)
- [Sampling controls](#sampling-controls)
  - [Temprature](#temprature)
  - [Top-p](#top-p)
  - [Top-k](#top-k)
- [Prompting](#prompting)
  - [sytem prompt](#sytem-prompt)
  - [contextual prompt](#contextual-prompt)
  - [role prompt](#role-prompt)
  - [Prompt techniques](#prompt-techniques)
- [Best practices](#best-practices)
  - [例を与える (Give examples)](#%E4%BE%8B%E3%82%92%E4%B8%8E%E3%81%88%E3%82%8B-give-examples)
  - [アウトプットを具体的にする (Be specific about the output)](#%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%E3%82%92%E5%85%B7%E4%BD%93%E7%9A%84%E3%81%AB%E3%81%99%E3%82%8B-be-specific-about-the-output)
  - [成約よりも指示を優先する (Prefer instructions over constraints)](#%E6%88%90%E7%B4%84%E3%82%88%E3%82%8A%E3%82%82%E6%8C%87%E7%A4%BA%E3%82%92%E5%84%AA%E5%85%88%E3%81%99%E3%82%8B-prefer-instructions-over-constraints)
  - [シンプルさで設計する (Design with simplicity)](#%E3%82%B7%E3%83%B3%E3%83%97%E3%83%AB%E3%81%95%E3%81%A7%E8%A8%AD%E8%A8%88%E3%81%99%E3%82%8B-design-with-simplicity)
  - [出力について具体的にする (Be specific about the output)](#%E5%87%BA%E5%8A%9B%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E5%85%B7%E4%BD%93%E7%9A%84%E3%81%AB%E3%81%99%E3%82%8B-be-specific-about-the-output)
  - [最大トークン長を制御する (Control the max token length)](#%E6%9C%80%E5%A4%A7%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3%E9%95%B7%E3%82%92%E5%88%B6%E5%BE%A1%E3%81%99%E3%82%8B-control-the-max-token-length)
  - [プロンプトで変数を使用する (Use variables in prompts)](#%E3%83%97%E3%83%AD%E3%83%B3%E3%83%97%E3%83%88%E3%81%A7%E5%A4%89%E6%95%B0%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B-use-variables-in-prompts)
  - [入力形式と記述スタイルを実験する (Experiment with input formats and writing styles)](#%E5%85%A5%E5%8A%9B%E5%BD%A2%E5%BC%8F%E3%81%A8%E8%A8%98%E8%BF%B0%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%82%92%E5%AE%9F%E9%A8%93%E3%81%99%E3%82%8B-experiment-with-input-formats-and-writing-styles)
  - [構造化された出力が必要な場合 (When structured output is required)](#%E6%A7%8B%E9%80%A0%E5%8C%96%E3%81%95%E3%82%8C%E3%81%9F%E5%87%BA%E5%8A%9B%E3%81%8C%E5%BF%85%E8%A6%81%E3%81%AA%E5%A0%B4%E5%90%88-when-structured-output-is-required)
  - [JSON修復 (JSON Repair)](#json%E4%BF%AE%E5%BE%A9-json-repair)
  - [他のプロンプトエンジニアと協力して実験する (Experiment together with other prompt engineers)](#%E4%BB%96%E3%81%AE%E3%83%97%E3%83%AD%E3%83%B3%E3%83%97%E3%83%88%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2%E3%81%A8%E5%8D%94%E5%8A%9B%E3%81%97%E3%81%A6%E5%AE%9F%E9%A8%93%E3%81%99%E3%82%8B-experiment-together-with-other-prompt-engineers)
  - [CoTのベストプラクティス (CoT Best practices)](#cot%E3%81%AE%E3%83%99%E3%82%B9%E3%83%88%E3%83%97%E3%83%A9%E3%82%AF%E3%83%86%E3%82%A3%E3%82%B9-cot-best-practices)
  - [様々なプロンプトの試行を文書化する (Document the various prompt attempts)](#%E6%A7%98%E3%80%85%E3%81%AA%E3%83%97%E3%83%AD%E3%83%B3%E3%83%97%E3%83%88%E3%81%AE%E8%A9%A6%E8%A1%8C%E3%82%92%E6%96%87%E6%9B%B8%E5%8C%96%E3%81%99%E3%82%8B-document-the-various-prompt-attempts)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Output length

より多くのトークンを生成するには、より多くの計算リソースが必要となり、エネルギー消費が増え、応答時間が遅くなり、コストが高くなる可能性がある。

## Sampling controls

一般的な開始点として、温度0.2、Top-P 0.95、Top-K 30は比較的整合性のある結果をもたらし、創造的すぎないようにできる。

### Temprature

温度は、モデルの出力のランダム性を制御するパラメータである。温度が高いほど、モデルはより多様な応答を生成し、低いほど決定論的な応答を生成する。

### Top-p

Top-pは、モデルが次のトークンを生成する際に考慮するトークンの確率分布の累積確率を制御するパラメータである。top-pが高いほど、モデルはより多くのトークンを考慮し、低いほど少ないトークンを考慮する。

### Top-k

Top-kは、モデルが次のトークンを生成する際に考慮するトークンの数を制御するパラメータである。top-kが高いほど、モデルはより多くのトークンを考慮し、低いほど少ないトークンを考慮する。
Kの値が高いほど多様性が増し、低いほど事実に忠実な出力になりやすい。

## Prompting

### sytem prompt

言語モデル全体のコンテキストと目的を設定します。モデルが何をすべきかの「大きな絵」を定義します。出力形式（例：大文字のみ、JSON形式）を指定するのにも有用です。安全や毒性のある出力を制御するためにも利用できます。

### contextual prompt

現在の会話やタスクに関連する特定の詳細または背景情報を提供します。モデルが質問のニュアンスを理解し、それに応じて応答を調整するのに役立ちます。タスク固有であり、動的です。

### role prompt

言語モデルに特定のキャラクターまたはアイデンティティを割り当てます。これにより、モデルは割り当てられた役割とその関連知識および行動と一貫性のある応答を生成できます（例：旅行ガイド、教師）。モデルの出力スタイルとトーンをフレーム化します。

### Prompt techniques

- **Few-shot prompting**: いくつかの例を与えることで、モデルに特定のタスクを学習させる。
- **Zero-shot prompting**: モデルにタスクを説明するだけで、例を与えない。
- **Chain-of-thought prompting**: モデルに思考過程を示すことで、より複雑なタスクを解決させる。
  - モデルに中間的な推論ステップを生成させることで、数学的なタスクなど、単に最終的な答えを返すだけでなく、問題解決のプロセスを「考える」ように促します。これにより、LLMはより正確な答えを提供できるようになります。「Let's think step by step.」のようなフレーズがトリガーとして使用されます。Zero-shot CoTと、例を含むFew-shot CoTがあります。CoTは、コード生成や合成データ生成など、問題を「話し合う」ことで解決できるあらゆるタスクに適しています。
- **Self-consistency**: モデルに複数の応答を生成させ、その中から最も適切なものを選択する。
  - Chain of Thoughtの考え方を拡張し、単一の線形的な推論パスではなく、複数の多様な推論パスを生成させ、最も一般的な答えを選択することで、応答の精度と一貫性を向上させます。高い温度設定で同じプロンプトを複数回実行し、各応答から答えを抽出し、最も一般的な答えを選択します。これにより、答えが正しい可能性の「疑似確率」が得られますが、コストが高くなります。
- **Tree of Thoughts**: モデルに思考の木を生成させ、各ノードで選択肢を評価させる。
  - Chain of Thoughtの概念を一般化し、LLMが複数の異なる推論パスを同時に探索できるようにします。複雑なタスクに特に適しており、各「思考」は問題解決への中間ステップとなる首尾一貫した言語シーケンスを表します。モデルはツリー内の異なるノードから分岐することで、異なる推論パスを探索できます。
- **Re Act**: モデルに特定のアクションを実行させる。reason & act
  - LLMが外部ツール（検索、コードインタープリターなど）を使用して複雑なタスクを解決できるようにするパラダイムです。自然言語の推論と外部ツールとの相互作用を組み合わせ、LLMが特定の行動を実行できるようにします。人間が推論し行動を起こして情報を得る方法を模倣し、思考-行動ループを通じて機能します。LangChainなどのフレームワークを使用して実装できます。

## Best practices

### 例を与える (Give examples)

- 例を与えることで、モデルが期待される出力の形式やスタイルを理解しやすくなります。

### アウトプットを具体的にする (Be specific about the output)

望ましいアウトプットを具体的に示すことで、モデルが期待される結果を理解しやすくなります。

### 成約よりも指示を優先する (Prefer instructions over constraints)

- インストラクションは、望ましい応答の形式、スタイル、または内容に関する明確な指示を提供する。これは、モ
  デルが何をすべきか、何を生成すべきかについてモデルを導きます。
- 制約とは、応答に対する一連の制限または境界のことである。モデルが何をすべきでないか、何を避けるべきかを
  制限します。

制約に大きく依存するよりも、肯定的な指示に焦点を当てた方が効果的。
可能であれば、積極的な指示を使用する。モデルに「何をしてはいけないか」を指示する代わりに、「何をすべきか」を指示する。こうすることで、混乱を避け、アウトプットの精度を向上させることができる

### シンプルさで設計する (Design with simplicity)

プロンプトは簡潔、明確で、モデルにとって理解しやすいものであるべきです。複雑な言語や不必要な情報は避けるべきです。具体的な行動を表す動詞（Act, Analyze, Classifyなど）を使用することが推奨されています。
ユーザーがわかりにくいものは、モデルもわかりにくいです。プロンプトは、モデルが理解できるように設計する必要があります。
複雑な言葉を使わず、不必要な情報を提供しないようにしましょう。

### 出力について具体的にする (Be specific about the output)

必要な出力の形式、スタイル、トーン、長さなどを明確に指定することが重要です。

### 最大トークン長を制御する (Control the max token length)

モデルの構成設定またはプロンプト内で明示的に指定することで、生成される応答の長さを制御できます。

### プロンプトで変数を使用する (Use variables in prompts)

プロンプトを再利用可能で動的にするために、変数を使用できます。これにより、異なる入力に対して同じプロンプトの構造を利用できます。アプリケーションへの統合時に特に有用です。

### 入力形式と記述スタイルを実験する (Experiment with input formats and writing styles)

モデルの種類、構成、プロンプトの形式、単語の選択、実行ごとに結果が異なる可能性があるため、様々な属性を実験することが重要です。質問、ステートメント、指示など、異なる形式で同じ目標を表現することで、異なる出力が得られます。

### 構造化された出力が必要な場合 (When structured output is required)

JSONなどの形式で構造化された出力を要求することで、データの扱いが容易になり、幻覚（hallucination）の可能性を減らすことができます。JSONスキーマを使用して出力の属性を定義することも可能です。

### JSON修復 (JSON Repair)

JSON出力はトークンを多く消費し、出力が途中で途切れると無効になる可能性があります。json-repairのようなライブラリは、不完全または不正なJSONオブジェクトを自動的に修復しようと試みることができます。

### 他のプロンプトエンジニアと協力して実験する (Experiment together with other prompt engineers)

複数の人が異なるプロンプトを試すことで、パフォーマンスのばらつきを確認し、学習することができます。

### CoTのベストプラクティス (CoT Best practices)

CoTプロンプトでは、推論の後に最終的な答えを置くことが必要です。推論と答えを分離して抽出できるようにする必要があります。CoTプロンプトでは、温度は0に設定する必要があります。これは、推論は通常単一の正しい答えを持つため、モデルが最も確率の高い次のトークンを選択するgreedy decodingに基づいているからです。

### 様々なプロンプトの試行を文書化する (Document the various prompt attempts)

プロンプトの試行とその詳細を文書化することは非常に重要です。これにより、後で再検討したり、異なるモデルバージョンでテストしたり、将来のエラーをデバッグしたりする際に役立ちます。プロンプトの名前、目標、使用モデル、設定（温度、トークン制限、Top-K、Top-P）、プロンプトテキスト、出力、評価結果（OK/NOT OK/SOMETIMES OK）、フィードバックなどを記録することが推奨されています。バージョン管理も重要です。
