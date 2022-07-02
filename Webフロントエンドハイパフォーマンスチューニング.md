# 1 章 ウェブパフォーマンスとは何か

- インタラクション = ユーザーの行動によってウェブページが何かしらの応答をする行為
- 複雑なインタラクションを持つアプリケーションが今多い
  - ウェブで表現できることが増えたから
- 今までは画面の初期表示スピードさえ気にしていればよかった
- SPA みたいに、初期ロード + インタラクションの描画パフォーマンスが重要になってきた
- ハイブリッドアプリ = HTML,CSS,JS で開発可能な OS ネイティブアプリケーションみたいに利用できるアプリケーション
- ネイティブアプリはコードがコンパイル済み + OS が API を直接用いるから高速
- ハイブリッドアプリはレンダリングエンジンに JS を解釈される必要があるし、HTML と CSS を解釈して描画するため描画パフォーマンス低い

# 2 章 ブラウザのレンダリングの仕組み

- ブラウザを構成する複数のコンポーネントのうちの重要な２つ
  - レンダリングエンジン
    - HTML の描画エンジン
    - HTML、画像、CSS、JS などのリソースをもとに画面上にピクセルで描画
    - Blink、Webkit など
    - ブラウザ以外でも HTML メールの表示に使われていたりする
    - ハイブリッドエンジンはレンダリングエンジンを組み込んでいる
  - JavaScript エンジン
    - JS の実行環境
    - V8、Chakra など
    - DOM ツリー構築、CSSOM ツリー構築、ブラウザ API への JS からのアクセスするバインディング
    - JS エンジンはブラウザ以外にも Node.js に V8 が使われていたり、単独の言語処理系として使われる
- レンダリングの流れ
  - 大きく 4 つ（総称をフレーム）
    - Loading => Scripting => Rendering => Painting
  - 細かくできる
    - Download(Loading) => Parse(Loading) => Scripting => Calculate Style(Rendering) => Layout(Rendering) => Paint(Painting) => Rasterize(Painting) => Composite Layouts(Painting)
  - https://web.dev/rendering-performance/ のピクセルパイプライン

## Loading

- サーバーからリソースのダウンロード
  - HTML ファイル
  - CSS ファイル
  - JS ファイル
  - 画像ファイル
- リソースのパース
  - このときに HTML は DOM ツリー、CSS は CSSOM ツリーへ変換される
- リソース取得順序
  - HTML が最初
    - HTML 内で参照しているリソースがあれば、そのリソースをロードする
    - 取得されたリソースはレンダリングエンジンの実装に沿った内部表現へ変換される
      - HTML は DOM ツリーへ
      - CSS は CSSOM ツリーへ
    - DOM ツリーを構築していく過程で、画像や CSS などのリソースを取得する
    - DOM ツリー構築の流れ
      - 字句解析によるトークンのリスト化
      - 構文解析による構文木の構築
      - 構文木内にある JS を実行しつつ DOM ツリーの構築
  - CSS が同時進行
    - 読み込まれた CSS をもとに CSSOM ツリー構築
    - ルールセットが増えると DOM ツリーのノードに適用されるスタイル計算の時間がかかる

## Scripting

- JS 実行の流れ
  - JS コード => 字句解析（トークン列生成） => 構文解析（AST 生成） => コンパイル（実行可能コード生成） => 実行
- コンパイル
  - 仮想マシンコードへのコンパイル
    - 言語処理を実行できる仮想マシンを用意し、それに向けてコンパイルする
      - 別名　バイトコードインタープリター
      - JIT コンパイル型よりも実装かんたん
      - 仮想マシンを使うので、マルチプラットフォーム対応がかんたん
    - JIT コンパイル
      - マシンの CPU が処理できるコードにコンパイルする
      - 機械語への変換により、実行時のパフォーマンス高い
      - マルチプラットフォーム対応が難しい
        - 実行環境の CPU によってコンパイルを分けないといけないから
- コンパイルが終わると実行し、DOM API の操作で DOM ツリーの操作が可能になる

## Rendering

- Calculate Style
  - DOM ツリーの全 DOM の CSS プロパティが何か計算
    - CSSOM ツリー内の CSS ルールセット（セレクタ＋ルール）を全部見る
    - DOM にどのルールが適用されるか計算
      - セレクタのマッチング処理
        - 右側からマッチング処理が適用
          - button .button
          - button クラスの button element
    - css の詳細度決定
    - 詳細度によって、該当 DOM の CSS ルールの適用を確定
- Layout
  - 要素の「大きさ」「マージン」「パディング」「位置」「z 軸の位置」などのレイアウト情報を計算
- Painting

## Painting

- Paint
  - 2D グラフィックエンジン向けの命令の列を生成する
    - レンダリングエンジンでは、グラフィックエンジンを組み込めるようになっている
    - Skia というグラフィックエンジンが主流
- Rasterize
  - Paint での命令をもとにピクセルへと描画
    - レイヤーに一枚一枚描画
      - レイヤーは Z 軸での上下関係を持つ
      - オーバーラップして表示されるコンテンツが有る場合に生成
        - position: absolute
        - position: fixed
        - transform: translate3d などの GPU で描画合成する
        - opacity を使って背後の何かを表示する必要がある
    - レイヤーを再利用することで、再レンダリングのスピードが上がる
      - スクロールするときは以前描画したレイヤーが再利用される
- Composite Layers
  - レイヤーを合成して最終的なレンダリング結果を生成
    - レイヤーは CPU と GPU で生成される
  - 基本的に CPU で条件満たすと GPU 合成

### 再レンダリング

- 何が変更されたかによってどのフェーズから再実行されるのか決まる

# 3 章 チューニングの基礎

- パフォーマンスの指標

  - RAIL
    - response 100 ミリ秒
      - アクションを起こしてなにかしらのフィードバックを返すまでのタイム
      - 指のタッチ、スクロールなどは 16 ミリ秒以下
        - これらのアクションは 100 ミリ秒以下の高い頻度でおこるため
    - animation 16 ミリ秒
      - 16 ミリ秒だと 60fps が実現できる
        - 1000 / 60 = 16.6666666667
      - アニメーションを実現するために JS を使う場合は、その JS 処理時間は 6 ミリ秒以下
        - システムや OS や GPU ドライバや描画合成の固定的な処理時間などに 4 ミリ秒
        - つまり、16 - 4 = 12
        - 安定的に 60fps を実現するためには JS 処理時間は半分の 12 / 2 = 6 ミリ秒以下が望ましい
      - CSS だと JS が必要ないので、1 フレーム（Scripting, Rendering, Painting）16 ミリ秒以下でいい
    - idle 50 ミリ秒
      - アイドル状態に実行される JS の処理時間
        - 一度 Painting まで終わって、ユーザからのアクションを待ち受けている状態
    - load 1000 ミリ秒
      - ページ読み込み開始からユーザがそのページを操作できるまでの時間

- 残りは計測方法に関しての説明

# 4 章 リソース読み込みのチューニング

- 最初に取得した html が参照しているソースを取得していく
  - `<link>`の css ファイル
    - css ファイルが参照している css ファイル
  - `<script>`の js ファイル
  - src の画像ファイル

## ファイルの圧縮

不要な空白なくしたり、変数名短くしたり

## 適切な画像形式

- jpeg
  - 画像
- gif
  - アニメーション
- png
  - それ以外
- webp
  - 2022/0620 現在 safari だけ特定条件だと使えない

## フォントの最適化

- フォントを使う場合、当然 loading される
- 日本語フォントはでかい
- サブセット化
  - ページ中で使うフォントだけ抜き出して配信する方法

## css import を避ける

- `@import`構文は import を書いた css ファイルが読み込まれるのを待ってから、その import 先 css ファイルを取得する
- つまり直列
- css ファイルの結合は良い手段
  - webpack https://webpack.js.org/plugins/css-minimizer-webpack-plugin/
  - vite https://vitejs.dev/guide/features.html#import-inlining-and-rebasing

## JS の同期読み込みを避ける

- DOM ツリー構築中に DOM を変更するような処理を書かない
  - `document.write("<div>hello</div>")`
  - html のパースを阻害する
- 同期的な処理を行う場合は html の最後の方に書く
  - link タグの読み込みの阻害をさせない

## JS を非同期で読み込む

- defer と async 属性
  - defer
    - 非同期で読み込む
    - ドキュメントのパースをロックしない
    - DOM ツリー構築後に実行
    - 宣言順は担保される
      - `<script defer src="1.js"></script> <script defer src="2.js"></script>`で 1.js => 2.js の順に実行される
    - DOMContentLoaded イベントの発火前に実行される
  - async
    - defer と同じ
    - DOM ツリー構築を待たず、js を読み込んだと同時に実行される
    - 実行順は担保されない
      - `<script async src="1.js"></script> <script async src="2.js"></script>`でも、どっちが最初に実行されるかわからない
- JS から`<script>`を生成して DOM に追加

  - defer async をサポートしていない場合にこの手法を使う
  - 基本的に defer async でいい

- 使う優先順位
  - async,defer,script element
  - async が一番パフォーマンスがいい

## ピクセル比で読み込む画像を変える

- img の srcset を使って、無意味に高画質な画像を読み込ませないように抑えることができる
- picture の source で画面幅による画像の使い分けができる

## css メディアクエリを適切に指定

- 画面に必要ないスタイルはメディアクエリで読み込ませない
  - link 要素の media 属性を指定すると、まるごと css ファイルを読ませないことができる
  - css 内でのメディアクエリではなく、media 属性を使うと効果的

## css スプライト

- HTTP/1.1 以下では 1 リソース 1 リクエストになる
- HTTP リクエストの並列数はブラウザ側で制限があるので、そこを圧迫しないように、複数の画像を１つの画像にまとめてしまう。
  - 接続ブロックを防ぐ
  - HTTP リクエストヘッダーと HTTP レスポンスヘッダーは通常 700~800 バイトであり、「画像数 \* 800 バイト前後」になっていたのを、「800 バイト前後」だけで済ませられる

## クリティカル CSS

- クリティカルレンダリングパス
  - ウェブページの初回の描画に必ず必要な工程
- link 要素の css ファイルの読み込みは初回の描画をブロックする
- CSS をインライン化させて任意のスタイルを読み込ませる = クリティカル CSS
  - css ファイルの読み込みをスキップさせて、CSS ファイルの初回描画を邪魔させない
    - `<link rel="stylesheet" href="style.css" media="none" onload="if (media !== 'all') {media = 'all';}">`
      - `media="none"`といった属性値は存在しないので、ブラウザは読み込みをスキップして描画を始める
      - やがて load されたあとに、media を all にして、ちゃんと使われるようにする
    - style タグにウィンドウに表示する領域のスタイルを直書きする

## リソースの事前読み込み

- DNS プリフェッチ
  - link 要素の rel に`pre-fetch`
    - ドメインの名前入解決をバックグランドで処理する
    - すでに名前解決されている場合、ブラウザがキャッシュしているので意味がない
- リソースの事前読み込み
  - link 要素の rel に`prefetch`
    - `<link rel="prefetch" href="./image.gif">`
  - 事前読込される
  - ブラウザのキャッシュに格納される
- プリレンダリング
  - link 要素の rel に`prerender`
    - `<link rel="prerender" href="//example.com/hoge.html">`
  - バックグラウンドでレンダリングまで行う
  - CPU リソースを使ったり、ネットワークの帯域を圧迫するので、やりすぎ注意
- 接続の投機的開始
  - link 要素の rel に`preconnect`
    - `<link rel="preconnect" href="//example.com">`
  - DNS 解決、TCP ハンドシェイク、TLS コネクション確立を行う

## Gzip 圧縮

- コンテンツを圧縮してレスポンスとする
- すでに圧縮されている画像などのファイルは、意味ないどころか圧縮・展開処理のオーバーヘッドがかかるのでだめ

## CDN

- オリジンサーバーに変わってキャッシュしたコンテンツを返すサーバーを提供するネットワーク
- 物理的距離による遅延を解消

## ドメインシャーディング

- リソースを提供するホドメインを複数に分散して、同時接続数問題を解決して、効率よくリソースの取得を可能にする
  - ブラウザは HTTP でリソースを取得するときに並列実行を試みる
  - 同じホストへの同時接続数は制限されている
    - モダンブラウザは 6
- ドメイン数を増やすと名前解決のために DNS の名前解決に時間がかかる場合もあるので、適度なドメイン分散をする
- CDN が提供している事が多い
- HTTP2 では意味がない
  - ドメインごとの同時接続数が最初からないから

## リダイレクトしない

- リダイレクトするとその分接続の前処理を行うので
- URL は最後に`/`をつける
  - `http"//www.example.com/hoge`は`http"//www.example.com/hoge/`にリダイレクトしていることが多い

## ブラウザキャッシュ

- HTTP ヘッダーの設定
  - 強いキャッシュ
    - 内容が全く変更されないようなファイルに使う
    - Expires
      - 日時指定
      - 期限が切れるまでブラウザにキャッシュさせる
      - 指定するとサーバにリクエストを飛ばさなくなる
      - サーバーとクライアントで時間設定がずれると意図通りに設定できなくなるので注意
    - Cache-Control
      - 経過時間指定
      - 期限が切れるまでブラウザにキャッシュさせる
      - ブラウザは Cache-Control と Expires 両方指定されていると、Cache-Control を優先する。モダンブラウザであれば使えるので、Expires よりこっちをつかう。
      - キャッシュさせたくないファイルがある場合は、URL に毎回異なるパラメータを付与し、違うリクエストとしてサーバーに認識してもらう
        - `<img src="image.png?192378274823" />`
  - 弱いキャッシュ
    - 頻繁に変更されるファイルに使う
    - Last-Modified
      - 日時指定
      - サーバーが`Last-Modified`をレスポンスに含め、ブラウザは次回以降リクエストを送るときに`If-Modified-Since`を含める
        - 条件付き GET リクエストと呼ぶ
      - サーバーが`If-Modified-Since`を見て、リソースの最終更新日がこれよりも後＝レスポンス返す。違う場合＝ 304 Not Modified ステータスを返してブラウザキャッシュ使ってもらう
    - ETag
      - エンティティタグ指定
        - サーバーがリソースをもとに生成したユニークな ID
      - サーバーが ETag を含めてブラウザに送信し、ブラウザはリソースファイルと ETag を対保存する
      - ブラウザが`If-None-Match`ヘッダ（中身は ETag と同じ）を含めてサーバーにリクエスト送信。サーバーは`If-None-Match`ヘッダの値と ETag の値を比較し、違う＝レスポンス返す。同じ＝ 304 Not Modified ステータスを返してブラウザキャッシュ使ってもらう
      - ブラウザは Last-Modified と ETag 両方指定されていると、ETag を優先する
      - 柔軟性が ETag のほうが高いので、ETag を使ったほうがいい

## Service Worker

## HTTP/2

- バイナリ化したプロトコル
  - バイナリを使ってデータのやり取りを行う
  - テキストベースよりも効率がいい
- HTTP/1.1 のセマンティクスの再利用
  - HTTP/1.1 と TCP の間に入って通信効率を高めるような感じ
  - HTTP/1.1 で使うメソッドやステータスを再利用できる
- リクエストとレスポンスの多重化
  - 1TCP 接続で複数のリクエストとレスポンスを並列実行可能
- HTTP ヘッダ圧縮
  - 700~800 バイトぐらいあったヘッダを圧縮

# 5 章 JavaScript 実行のチューニング

- UI スレッド（メインスレッド）
  - １つのタブに付き１つ存在
  - このスレッド上でシングルスレッドで順に動作
  - 各フレームの処理が遅くなれば、シングルスレッドなので後続に影響が出る
- イベントループ
  - UI スレッド上で動く
  - UI スレッドでの JS の並行処理を実現させる
  - 一部 UI スレッドをブロックする処理が存在
    - prompt()
    - alert()
    - confirm()
    - 同期 XMLHttpRequest

## GC 避ける

- JS ではメモリへの直接の処理は完全に隠蔽されている
- オブジェクトを new するときに閾値を超えたら実行される GC によって、描画に影響がある場合がある
  - 殆どの場合そこまで問題にならない
    - requestAnimationFrame のアニメーション実行中にかくつく場合ある
- マークアンドスイープアルゴリズム
  - モダンブラウザでの GC に使われるアルゴリズム
  - root（グローバルオブジェクト）から開始し、root から参照されるオブジェクトをたどっていく
    - 参照されない = ガベージコレクト対象

## メモリリークを避ける

- メモリは使えば使うほど動作が遅延する
  - メモリにキャッシュを仕込んでの速度の向上は見込めない
- console に渡したオブジェクトは GC 対象にならない
  - dev tool に表示するために参照がつく
- DOM リークで 1 つ Node がリークしただけで、それに紐づく DOM ツリーがリークする

## WeakMap, WeakSet

- Map はキーにオブジェクトを指定すると、 Map のキーでそのオブジェクトの参照を持ち続ける
- WeakMap はキーにオブジェクトを指定すると、 そのオブジェクトの参照が消えた時点でキーにあっても WeakMap から参照がなくなり、参照されていたオブジェクトは GC 対象になる
- Set は値にオブジェクトを指定すると、 そのオブジェクトの参照を持ち続ける
- WeakSet は値にオブジェクトを指定すると、 そのオブジェクトの参照が消えた時点で値にあっても WeakSet から参照がなくなり、参照されていたオブジェクトは GC 対象になる

## 高頻度発火イベントの抑制

- scroll イベントや resize イベント
- イベントを X 秒に Y 回みたいに抑制してしまう

## Passive Event Listener

- event listener の option に`{passive: true}`とすることで適用
- listener が`event.preventDefault()`を呼び出さないことを保証する
- これによってブラウザは処理の完了を待つ必要がなく、イベントを実行できる
  - scroll するときに処理の完了を待つことになり、スクロールを開始することができない

## アイドル時処理を行う

- UI スレッド上で長時間かかる処理を行うと、ブラウザは警告を行う
- RAIL の Idle 時の script 実行時間は 50 ミリ秒以内
  - 50 ミリ秒以上かかる場合は次のイベントを実行してもらうようにタスクキューに入れて、再度後ほど実行してもらう
- `requestIdleCallback`で UI スレッドのアイドル状態に処理を実行させることができる
  - https://codesandbox.io/s/jolly-brown-icn6b9?file=/src/index.ts

## ページ表示状態を確認する

- ページを見てもらっているときに表示させたい UI は、見てないときに実行しても意味がない
  - なんかのアニメーション
  - 音楽の再生
- `document.hidden`で表示されているか bool で分かる
- `visibilitychange`イベントもある
  - https://developer.mozilla.org/ja/docs/Web/API/Document/visibilitychange_event

## Forced Synchronous Layout をへらす

- DOM 操作を行うと引き起こされるもの
  - Calculate Style と Layout
  - レイアウトに影響を与える DOM 操作を行ったあとに、関連する DOM 要素のレイアウト情報を参照すると発生する
    - 座標取得
      - clientLeft
      - clientTop
      - offsetLeft
      - offsetTop
    - 大きさ取得
      - clientWidth
      - clientHeight
      - offsetWidth
      - offsetHeight
    - 座標や大きさ取得
      - getClientRects()
      - getBoundingClientRect()
- Scripting フレームの最中に同期的に実行されるので、JS の実行パフォーマンスに影響を与える
- Layout Thrashing を避ける
  - ループ内で Forced Synchronous Layout を起こすと発生
  - ループ前に予めレイアウト情報を取得しておくと防げる
- Forced Synchronous Layout を避ける
  - レイアウト情報を取得したい DOM のレイアウト情報を、DOM 操作を行う前に取得しておく
  - 予め取得できない場合は `requestAnimationFrame()`のコールバック内でレイアウト情報を参照する
    - 新たに追加した DOM のレイアウト情報を参照したい場合とか
    - コールバック実行タイミングは DOM 操作が終わって再レンダリングが終わってからなので、すでに算出済みのレイアウト情報を参照することになり、Forced Synchronous Layout がおこらない

## DocumentFragment を使う

- `document.createDocumentFragment()`で複数の DOM を格納し、fragment に貯めた DOM を`document.body.appendChild(fragment)`のように一気に追加する
- DOM を随時作って随時 appendChild するよりは、一回の DOM ツリー干渉で済むので効率がいい
- https://codesandbox.io/s/reverent-leaf-9je9q3?file=/src/index.ts

## requestAnimationFrame を使う

- setTimeout や setInterval は、特定の時間を引数に入れないといけず、デバイスの負荷による時間調整が容易ではない
- requestAnimationFrame はブラウザのレンダリングの処理に合わせて適切なタイミングで呼び出されるので、ちゃんとすれば 60fps で処理が実行可能

# 6 章 レイアウトツリー構築のチューニング

- 再レンダリングの Calculate Style は、全て計算し直すわけではなくて、新たに計算し直さないといけない DOM 要素のスタイルのみが計算される
- セレクタのチューニング
  - セレクタをシンプルにする
    - 右から左に DOM と突き合わせながらマッチング処理をするので、セレクタが短いほどマッチング処理が減る
  - 子孫セレクタ（`.header .nav .nav-list`）や間接セレクタ（`header > div span`）を避ける
    - 子から見て、親をたどる必要があるので、深い DOM ツリーで親子関係が離れているとその分たどる距離が長い
    - バックトラック
      - 一部の処理を何回もやり直すことになる
        1. 要素名が span
        2. div 要素が親のどこかにあるはずなので上を参照していく
        3. その親に header があるはずなので、親を見る。なければ 2 をやり直す
  - 全称セレクタを避ける
    - 余計なマッチング処理が発生するから
    - 全要素が対象なので、全要素とマッチングした上で、親要素のマッチング処理を行ったりしてしまう
      - `header *`
        1. 要素
        2. その要素の親に header 要素がないか探索する。

## Layout を避ける

- Layout を引き起こす典型的な原因は
  - DOM 要素の座標や大きさの変化
    - ボックスモデルにおける、ボックスの大きさに影響を与える CSS プロパティの変化
      - height, width
      - top, left, right, bottom
      - margin, padding
      - border
  - DOM ツリーの構造の変化
    - DOM の増減
      - removeChild
      - appendChild
      - innerHTML
  - DOM 要素のコンテンツの変化
    - テキストの増減
- 以下のときに DOM 要素は独自のレイアウトを作り、他の DOM に依存することなく負荷を減らせる
  - svg 要素
  - input[type=text or search]
  - inline, inline-block ではない
  - height, width が auto 以外
  - height に%を使っていない
  - overflow scroll or auto or hidden

## img 要素のサイズを固定する

- 無駄な Layout を h ラスには img に width と height 属性を指定する
- 指定しない場合、仮のサイズで Layout が行われ、画像のサイズが確定した時点で再度 Layout とレンダリングが走る

# 7 章 レンダリング結果の描画のチューニング

## 再描画

- Painting フェーズが終わっても、インタラクションのたびに再度 Painting が引き起こされる = Repaint
- Repaint の原因は
  - Rendering の Layout が呼び出される
    - Layout がおこると後続フェーズの Painting のすべて（Paint、Rasterize、Composite Layers）が行われる
  - Painting だけが呼び出される
    - 色、背景など、Layout に影響はないが、再描画はしてもらわないといけないものたちが変更されたとき
  - Composite Layer のみが呼び出される
    - コストは比較的安価
    - 高速なアニメーションを実現することができる
      - opacity の値の変更
      - transform が別の値になったとき

## レイヤーの生成条件

- Paint や Rasterize はレイヤーごとに処理を行う = レイヤーを描画する処理を行う
- Composite Layers ではレイヤーを一枚に合成して最終的な描画結果を生成する
- レイヤーは以下のときに生成される
  - z-index で上下関係を指定
  - transform で 3D 変形
- レイヤーがツリー状に構成される = レイヤーツリー
- GPU で合成されるレイヤーを Graphics Layer
  - GPU による合成
    - Rasterize でビットマップ化されたレイヤーが GPU の VRAM へ送られ、合成されて描画される
    - 要素に施された transform での 3D 変形やエフェクトなどの特定のプロパティにしたがってビットマップを加工・変形して合成
- Graphics Layer が多くなると
  - それを保持するためにメモリを消費する
  - VRAMt 転送に時間がかかる
  - Composite Layers までにかかるオーバーヘッドが大きくなる
- Graphics Layer の性質を有効に扱うと、アニメーション処理を高速化可能

# 8 章 高度なチューニング

## バーチャルレンダリング

- ユーザが目に見える範囲でのみ DOM をツリーに挿入する
  - 大量の要素を DOM ツリーに挿入するよりは効率がいい
    - Calc StyleLayout や Paint などを経ないため

## なめらかなアニメーションの実現

- 60fps を実現させる
  - RAIL モデルだと 1 描画 16ms 以内
    - 再レンダリングの各フェーズをスキップすることが大事
- Calc Style のスキップ
  - DOM ツリーの構造の変化（appendChild など）と DOM 要素の属性の変化（class 属性付与など）を避ける
  - 各 Node の style 属性を直接変更すると、DOM 要素と CSS ルールのマッチング処理を避けられる
- Layout のスキップ
  - 要素の位置や大きさを変えたい場合は、transform プロパティを使う
    - translate()
    - scale()
- Paint をスキップ
  - 以下の CSS プロパティは Paint を引き起こさずに Composite Layers のみを引き起こすので、なるべくこれらをアニメーションに使う
    - opacity
    - transform
- Composite Layers の最適化
  - レイヤー合成には CPU 合成と GPU 合成がある
  - GPU 合成
    - 特定の CSS プロパティを使うと、GPU 合成のレイヤーが生成され、GPU へと転送後に変形、合成される。
      - transform
    - このレイヤーは値が変わっても GPU へと転送済みのレイヤーを再利用する
      - https://codesandbox.io/s/broken-brook-gvkzcg?file=/index.html

## will-change

- 近いうちに値が変化する CSS プロパティを指定する
  - とはいっても、指定できる CSS プロパティは限られていて、translateZ ハックに利用できる opacity と transform の 2 つのみ
- JS での指定が推奨
  - 変化開始のタイミングのみに will-change をつけて、変化終了のタイミングで will-change を外すことを想定されているため
  - CSS で固定でつけることはないが、常に変化する場合は可能。
    - スクロールに応じてアニメーションされる
    - 常にアニメーションされている
- ブラウザではすでに最適化が行われているはずで、will-change に期待しすぎてはいけない。使う場合は必ずパフォーマンスの計測をする。
  - いくつかのブラウザでは最適化が行われない

## CSS Containment https://caniuse.com/mdn-css_properties_contain

- https://developer.mozilla.org/ja/docs/Web/CSS/contain
- 前提として、再レンダリングは一部の CSSOM や DOM に変更があったとしても、レイアウト計算や描画自体を全体でやりなおすため、再レンダリング対象はドキュメント全体
- `contain`はレンダリングを一部の DOM に封じ込められる
  - 見た目のためではなくてパフォーマンスのため
    - will-change と似てる
- 対象の DOM に contain を指定すると、それ以下の子孫 DOM で発生した再レンダリングをその DOM 以下に制限できたり（ドキュメント全体の再レンダリングを防げる）、全体の再レンダリングの対象外にすることもできる
- 値
  - layout
    - レイアウトの計算を子孫以下に閉じ込める
    - レイアウトの関係性が外に漏れない
  - paint
    - ペイントの計算を子孫以下に閉じ込める
    - 対象外にある要素を描画しない
  - style
    - スタイルを閉じ込める
    - 連番スタイルをリセットする
  - size
    - 要素の大きさを閉じ込める
    - 子要素を持たない要素としてレイアウト計算する
      - 縦横がない
    - 一緒に width と height 指定もしないと 0px×0px の要素とされる
  - content = paint, layout, style
  - strict = paint, layout, style, size

# 9 章 認知的チューニング
