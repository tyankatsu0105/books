<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [How JavaScript works: an overview of the engine, the runtime, and the call stack](#how-javascript-works-an-overview-of-the-engine-the-runtime-and-the-call-stack)
- [How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](#how-javascript-works-inside-the-v8-engine--5-tips-on-how-to-write-optimized-code)
- [How JavaScript works: memory management + how to handle 4 common memory leaks](#how-javascript-works-memory-management--how-to-handle-4-common-memory-leaks)
- [How JavaScript works: Event loop and the rise of Async programming + 5 ways to better coding with async/await](#how-javascript-works-event-loop-and-the-rise-of-async-programming--5-ways-to-better-coding-with-asyncawait)
- [How JavaScript works: Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](#how-javascript-works-deep-dive-into-websockets-and-http2-with-sse--how-to-pick-the-right-path)
- [How JavaScript works: A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](#how-javascript-works-a-comparison-with-webassembly--why-in-certain-cases-its-better-to-use-it-over-javascript)
- [How JavaScript works: The building blocks of Web Workers + 5 cases when you should use them](#how-javascript-works-the-building-blocks-of-web-workers--5-cases-when-you-should-use-them)
- [How JavaScript works: Service Workers, their lifecycle and use cases](#how-javascript-works-service-workers-their-lifecycle-and-use-cases)
- [How JavaScript works: the mechanics of Web Push Notifications](#how-javascript-works-the-mechanics-of-web-push-notifications)
- [How JavaScript works: tracking changes in the DOM using MutationObserver](#how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver)
- [How JavaScript works: the rendering engine and tips to optimize its performance](#how-javascript-works-the-rendering-engine-and-tips-to-optimize-its-performance)
- [How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security](#how-javascript-works-inside-the-networking-layer--how-to-optimize-its-performance-and-security)
- [How JavaScript works: Under the hood of CSS and JS animations + how to optimize their performance](#how-javascript-works-under-the-hood-of-css-and-js-animations--how-to-optimize-their-performance)
- [How JavaScript works: Storage engines + how to choose the proper storage API](#how-javascript-works-storage-engines--how-to-choose-the-proper-storage-api)
- [How JavaScript works: the internals of Shadow DOM + how to build self-contained components](#how-javascript-works-the-internals-of-shadow-dom--how-to-build-self-contained-components)
- [How JavaScript works: WebRTC and the mechanics of peer to peer networking](#how-javascript-works-webrtc-and-the-mechanics-of-peer-to-peer-networking)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## [How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

- JS エンジンは Google の V8
- メモリヒープとコールスタックが主要なコンポーネント
- API（WebAPI 含む）は V8 によって提供
- JS はシングルスレッド
- 処理はコールスタック管理
  - LIFO
  - よって後に入った処理が最初に実行
- Blowing the stack
  - スタックに処理が貯まって the maximum Call Stack size に達すると発生
  - ブラウザはエラーをスローして停止する
- シングルスレッド故、複雑なシナリオを考慮する必要がない
  - [デッドロック - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%87%E3%83%83%E3%83%89%E3%83%AD%E3%83%83%E3%82%AF)とか
- JS は単一のコールスタックなので、遅いはず
  - でも回避策がある
  - それはなにか
  - 非同期コールバック

## [How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

- JavaScript エンジンは、 JavaScript コードを実行するプログラムまたはインタプリタ
  - [インタプリタとコンパイラ](https://nyumon-info.com/program/bunrui.html)
- V8 は複数ある JS エンジンの一つ
  - C++
  - Chrome 内部
  - Node.js のランタイムにも
- V8 初期

  - インタプリタの代わりにマシンコードに変換
    - > コンパイラってこと？
  - web ブラウザの実行パフォーマンス向上のため設計
  - JIT コンパイラ実装
    - [実行時コンパイラ - Wikipedia](https://ja.wikipedia.org/wiki/%E5%AE%9F%E8%A1%8C%E6%99%82%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%A9)
    - JIT 搭載エンジンは SpiderMonkey や Rhino などにも

- V8 5.9 以前
  - ２つのコンパイラ搭載
    - full-codegen
    - Crankshaft
    - 現在これらは使われてない
      - 現在はコレ　[Launching Ignition and TurboFan · V8](https://v8.dev/blog/launching-ignition-and-turbofan)
  - いくつかのスレッド
  - メインスレッド
    - コードを実行
  - コンパイルスレッド
    - メインスレッドに渡すコンパイル済みコードを生成
  - プロファイラースレッド
    - Crankshaft で最適化をする際に、どのメソッドが時間かかるかランタイムに伝える
    - [プロファイラ](http://e-words.jp/w/%E3%83%97%E3%83%AD%E3%83%95%E3%82%A1%E3%82%A4%E3%83%A9.html)
  - GC をスイープするスレッド
    - [ガベージコレクション](https://ja.javascript.info/garbage-collection)
    - [スウィープ - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%A6%E3%82%A3%E3%83%BC%E3%83%97)
  - full-codegen で一般的なバイナリデータに変換して、プロファイラースレッドから指摘された箇所を Crankshaft で効率的なコードに変換する
- Crankshaft による最適化
  - Hydrogen という SSA に変換し、Hydrogen グラフを最適化しようとする
    - [V8 の JIT コンパイラ、Crankshaft について — V8 Crankshaft Overview 1.3 documentation](http://nothingcosmos.github.io/V8Crankshaft/src/blog.html)
  - inline
    - 別関数を main な関数に取り込む
  - hidden class
    - JS はクラスのインスタンス生成してもプロパティメソッドを変更できてしまう
    - JS インタプリタはハッシュ関数的なものでメモリ割り当てを行うのでコストがかかる。
      - プロパティ値の取得に時間がかかる
    - Java のような静的プログラミング言語はその心配がない
      - プロパティメソッドを変更できない
      - プロパティタイプが固定で、固定オフセットでメモリ内に連続バッファで保存可能
      - 効率的
      - continuous buffer in the memory with a fixed-offset between each - [バッファ 【 buffer 】 緩衝記憶装置
        ](http://e-words.jp/w/%E3%83%90%E3%83%83%E3%83%95%E3%82%A1.html)
    - [V8 の Hidden Class の話 - LINE ENGINEERING](https://engineering.linecorp.com/ja/blog/v8-hidden-class/)
    - オブジェクトにプロパティが追加されるたびに HC が追加
    - 元の HC は transition として新たに生成された HC を参照する
    - HC は再利用できる
      - a,b プロパティを追加した流れを他の箇所でも流用可能
    - プロパティに追加した順番が違うと異なる HC が生成され、再利用されない
      - a,b プロパティを追加、b,a プロパティを追加は違う
      - a,b プロパティを追加、a,b プロパティを追加は同じで再利用可能
  - インラインキャッシング
    - [Optimizing dynamic JavaScript with inline caches · sq/JSIL Wiki · GitHub](https://github.com/sq/JSIL/wiki/Optimizing-dynamic-JavaScript-with-inline-caches)
    - 同じ HC に二回 transition すると、直接メモリのオフセットをセットして直接参照するようになる
      - 結果早くなる
  - Hydrogen の最適化後は Lithium という下位レベルの表現にする
  - Lithium はマシンコードになる
  - on-stack replacement を行う
    - [On-Stack Replacement — OpenJDK Internals 1.0 documentation](https://nothingcosmos.github.io/OpenJDKOverview/src/osr.html)
- GC

  - マークアンドスイープでクリーンアップ
    - [マーク・アンド・スイープ - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%BC%E3%82%AF%E3%83%BB%E3%82%A2%E3%83%B3%E3%83%89%E3%83%BB%E3%82%B9%E3%82%A4%E3%83%BC%E3%83%97)
    - インクリメンタルマーキングを使用

- [V8 の Ignition Interpreter について - Speaker Deck](https://speakerdeck.com/brn/v8falseignition-interpreternituite)

## [How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

[JavaScript の仕組み：メモリ管理+ 4 つの共通のメモリリーク処理方法 - Qiita](https://qiita.com/tkdn/items/ea4f034e0d661def244a#3-%E3%82%AF%E3%83%AD%E3%83%BC%E3%82%B8%E3%83%A3)
[JavaScript で起こるメモリリークのパターン - EagleLand](https://1000ch.net/posts/2017/javascript-memory-leak.html)

- JS は参照されなくなったオブジェクトのメモリを自動で開放する
  (https://developer.mozilla.org/ja/docs/Web/JavaScript/Memory_Management)
- 自動とはいえ、メモリ管理を知らなくても良いわけではない
  - 自動メモリ管理の問題がある場合があるので
- メモリライフサイクル
  - Allocate memory
  - Use memory
  - Release memory
  - [メモリとスタックとヒープとプログラミング言語 | κeen の Happy Hacκing Blog](https://keens.github.io/blog/2017/04/30/memoritosutakkutohi_puto/)
- メモリとは
  - 1 ビットを格納するトランジスタはフリップフロップに含まれている
  - フリップフロップはアドレス指定できる
  - コードをコンパイルすると、コンパイラはプリミティブデータ型を調べて、必要なメモリ量を事前に計算する
  - stack space のプログラムに必要な量が割り当てられる
- dunamic allocation
  - ユーザーによって値が変わる場合、メモリを決定できない
  - OS にメモリ量を聞く
  - メモリはヒープ領域からもらう
    - ヒープ領域は OS が持ってる
- JS での allocation と GC
  - [メモリ管理 - JavaScript | MDN]
- プログラムの書き方によって GC では開放されないメモリが残ってしまう
  - メモリリーク
  - いくつかパターンが有る
    - Global variables
      - 変数宣言しない変数はグローバルに作られてしまう
      - `bar = 'bar'` = `window.bar = 'bar'`
      - null にしたら参照なくなって GC で開放される
    - Timers or callbacks
      - 変数を null にして参照しなくなっても、timer は動き続ける
    - Closures
      - クロージャーが同じスコープレベルに存在すると、スコープが共有される
        - つまり、同階層のスコープであれば変数を共有できる
      - [JavaScript のクロージャはメモリーリークをちゃんと理解して使おう - Qiita](https://qiita.com/102Design/items/be66ae7ba7d160d7e419)
    - Out of DOM references
      - コードで DOM を参照したら、参照を消さないとメモリに残り続ける
      - 子 node は親 node を参照するので、子を消したからと言って親は残る
  - [JavaScript's Memory Management Explained](https://felixgerschau.com/javascript-memory-management/)

## [How JavaScript works: Event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

- single thread
  - スタックが溜まって処理が多くなるとブラウザがページ終了の confirm を出す
    - これは Bad UX
- 同期的じゃない処理は、まだ値がないので処理がおかしくなる
  - コールバックを使う
- イベントループ
  - [Node.js でのイベントループの仕組みとタイマーについて - 技術探し](https://blog.hiroppy.me/entry/nodejs-event-loop)
- Job Queue
  - [Task Queue and Job Queue - Deep dive into Javascript Event Loop Model - Hashnode](https://hashnode.com/post/task-queue-and-job-queue-deep-dive-into-javascript-event-loop-model-cjui19qqa005wdgs1742fa4wz)

## [How JavaScript works: Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7)

[Deep dive into WebSockets and HTTP/2 with SSE を読んだメモ - castaneai のブログ](https://castaneai.hatenablog.com/entry/2018/11/02/160707)

- インターネットはもともと単方向
- その後 Ajax が出現し、双方向での通信方法を模索

  - 定期的なポーリング

- ポーリング
  - 接続を常に解放
  - 通信したら、その都度接続を確立
  - よって常に接続されているように見える
  - [ロングポーリング](https://ja.javascript.info/long-polling)
- ウェブソケット
  - クライアントとサーバーで永続的に確立された接続を実現
    - クライアントがサーバーに接続することを伝える
    - サーバーが受理する
    - WS 確立
  - [WebSocket](https://ja.javascript.info/websocket)
  - [The WebSocket API (WebSockets) - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/WebSockets_API)
  - [インターリーブ(いんたーりーぶ)とは - コトバンク](https://kotobank.jp/word/%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%AA%E3%83%BC%E3%83%96-895)
  - 任意の時点でクライアントから ping、サーバーから pong で接続確認できる - できるだけ早く pong を返さないといけない - Heartbeating - [ハートビート 【 heartbeat 】
    ](http://e-words.jp/w/%E3%83%8F%E3%83%BC%E3%83%88%E3%83%93%E3%83%BC%E3%83%88.html)
- WebSocket と HTTP / 2
  - [WebSocket のはなし｜ Wireless・のおと｜サイレックス・テクノロジー株式会社](https://www.silex.jp/blog/wireless/2016/10/websocket.html)
  - [EventSource - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/EventSource)
- 双方向通信方法２つ
  - WebSocket
    - 十分使われている技術
    - 低遅延でほぼリアルタイムの接続が必要な場合
  - HTTP/2 + SSE
    - SSE
      - サーバーからクライアントに非同期でデータを送れる
      - [`yml](:note:a9e7062a-7019-431d-9401-275d9ecc5171)
    - 今の所対応しているブラウザが少ない
  - [WebSocket と SSE による HTTP サーバー Push](https://www.ibm.com/developerworks/jp/web/library/wa-http-server-push-with-websocket-sse/index.html)

## [How JavaScript works: A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)

- wasm
  - バイナリを配信するので読み込みが高速
- v8 の動き
  - コードベース
    - 構文解析し AST へ
    - ツリーを走査してマシンコードへ
    - TurboFan で最適化
    - ここまでのプロセスに CPU を消費する
  - wasm
    - AST スキップ
    - マシンコードへ変換スキップ
    - 最適化スキップ
    - wasm をデコードするだけで終了
    - CPU が削減される
- GC
  - wasm をコンパイルする言語は GC がないので、wasm に GC は作用しない
  - dom 変更のような複雑処理は不可能
  - 重い計算が絡むような処理に有効
    - アニメーションとか
- プラットフォーム API
  - DOM、CSSOM、WebGL、IndexedDB、Web Audio API など
  - wasm はアクセス不可能
    - JS を通してアクセスする
  - 将来 wasm に API を提供する予定

## [How JavaScript works: The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

Web Workers

- [Web Worker の使用 - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API/Using_web_workers)

- 非同期プログラミングによって UI のレンダリングに支障が出なくなる
  - リクエストを待ってる間に他のコードの実行が可能だったり
- 非同期処理の成功コールバック内のコードが重要な場合は？
  - リクエストは WEB API によって処理されるので問題ない
  - 例えばでかい for 文の結果を非同期にしたい場合は？
    - UI をブロックしないで実行することはできない
  - 非同期関数は JS のシングルスレッド制限のごく一部しか解決しない
  - settimeout 使えばできなくもない
    - for 文だったら settimeout と再帰使う
- Web workers はイベントループをブロックせずに JavaScript コードを実行するために使用できるブラウザー内スレッド
  - JS を介してアクセスできるブラウザの機能
  - そのため Node には存在しない
- Web workers は 3 種類ある
  - Dedicated Workers 　この記事ではこれについて解説
  - Shared Workers
  - Service workers
- Web workers は非同期 http リクエストに含まれる js ファイルで実装される
  - ブラウザ内の分離されたスレッドで実行されるので、js ファイルも分離する必要がある
- worker ファイルを指定したインスタンスを生成し、postMessage で処理を開始する
  - それ以外では BroadcastChannel で通信する
    - [BroadcastChannel - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/BroadcastChannel)
  - post されたメッセージを受け取ると、イベントループをブロックせずに処理
  - WebWorkers 内ではグローバルは self
    - [WorkerGlobalScope.self - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/WorkerGlobalScope/self)
- web worker にはアクセスできるものの制限がある
  - [Web Workers が使用できる関数とクラス - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)
- Web workers には良い用法がある
  - Ray tracing
  - Encryption
  - Prefetching data
  - Progressive Web Apps
  - Spell checking

## [How JavaScript works: Service Workers, their lifecycle and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)

- ネイティブアプリのような Experience 目的の PWA

  - [プログレッシブウェブアプリ | MDN](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps)

- SW の特徴
  - 独自のグローバルコンテキスト
    - self
  - 特定の Web ページに関連付けられない
  - DOM にアクセス不可能
- SW のファイルの場所はドメインルート
  - スコープが origin 全体
- register は load イベント後に行うようにする
  - SW があるかどうかを最終的に判断したほうがメモリや GPU を使わないのでエコ

> コレ見たほうが理解早そう　[Service worker の使用 - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Service_Worker_API/Using_Service_Workers)

## [How JavaScript works: the mechanics of Web Push Notifications](https://blog.sessionstack.com/how-javascript-works-the-mechanics-of-web-push-notifications-290176c5c55d)

- プッシュ通知
  - モバイルだと一般的
  - ウェブへの参入は比較的遅め
  - Service Workers によって実現
- なぜ SW で？
  - バックグラウンドで動いてくれる
  - ユーザーは通知としか対話をしない
  - 通知のコードしかいらない
  - SW が最適
- push & notification
  - push
    - サーバーから SW に情報が提供されたときに呼ばれる
  - notification
    - スクリプト
    - SW の実際のアクション
- push の手順

  - UI
    - プッシュ許可を許可するか否かユーザーに委ねる
  - server からデバイスへメッセージプッシュ
  - ブラウザに到着したメッセージの処理

- より詳細の手順

  1. ブラウザーサポートの検出
     - if 文のやつ
     - navigator.serviceWorker
     - window.PushManager
  2. SW 登録
  3. SW の regist
     - permission allow or block
       - granted 付与する
       - default
       - denied
       - 以上３つ
       - block で denied だと再度許可を求めることは不可能
  4. registration.pushManager.subscribe(options)のオプション
     - userVisibleOnly
       - true にしないとエラー出る
       - ユーザーに見える形でプッシュ通知をするかどうか
     - applicationServerKey
       - 公開鍵
       - サーバーの秘密鍵とペア
       - ここで作るといいらしい
       - [Push Companion](https://web-push-codelab.glitch.me/)
     - つまりこう
       - pushSubscription => subscribe
       - ウェブアプリケーションロード後、subscribe()が呼ばれ、サーバーキーが渡る
       - エンドポイントを生成し、キーとエンドポイントを関連付けて、プッシュサービス（サードパーティーのサービス）にネットワーク要求を行う
       - pushSubscription オブジェクトにエンドポイントを足して、subscription 関数を返す
  5. サーバーに先程の pushSubscription オブジェクトを送る

- push service
  - サービス
  - ユーザーが管理するものではない
    - コレが有名　[Firebase Cloud Messaging  |  Firebase](https://firebase.google.com/docs/cloud-messaging/)
  - ブラウザがオフラインの時、キューに入れて、オンラインになったら実行
- push servise API
  - プッシュメッセージを送信する方法を提供
  - push servise API = push servise protocol
  - 取り決め
    - [draft-ietf-webpush-protocol-12 - Generic Event Delivery Using HTTP Push](https://tools.ietf.org/html/draft-ietf-webpush-protocol-12)
  - プッシュメッセージで扱うデータは暗号化の必要がある
    - プッシュサービスが安全でないやつ使ってる可能性を考慮
  - メッセージに対していくつかオプションの指定が可能
- ブラウザでイベントをプッシュ
  - 以下の流れ
    - サーバーからメッセージを push service に送る
      - メッセージは保留され、トリガーが起動するまで保留のままになる
    - push service がメッセージを配信
    - ブラウザはそれを受信
    - SW へ push
    - SW はそれを処理
  - 該当ページを開いてなくても、ブラウザーが SW を実行するので、通知を受け取れる

## [How JavaScript works: tracking changes in the DOM using MutationObserver](https://blog.sessionstack.com/how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver-86adc7446401)

- MutationObserver は DOM の変更を検知できる
  - [MutationObserver - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/MutationObserver)
- 便利なケースがいくつかある
  - サイト訪問者にページの変更知らせる
  - DOM 変更検知して JS を動的にロード
  - undo redo
- MutationObserver が出るまではどうやって変更検知していたのか
  - ポーリング
    - setInterval WebAPI 使って
  - MutationEvents
    - DOM すべての変更を検知する
    - パフォーマンスがとんでもないことになる
    - 今存在しない
  - css animation
    - 要素が追加されると animationstart イベントが発生するので、それを応用

## [How JavaScript works: the rendering engine and tips to optimize its performance](https://blog.sessionstack.com/how-javascript-works-the-rendering-engine-and-tips-to-optimize-its-performance-7b95553baeda)

[ブラウザのしくみ: 最新ウェブブラウザの内部構造 - HTML5 Rocks](https://www.html5rocks.com/ja/tutorials/internals/howbrowserswork/)

- JS が実行される環境
  - ![1*lMBu87MtEsVFqqbfMum-kA.png](https://miro.medium.com/max/1748/1*lMBu87MtEsVFqqbfMum-kA.png)
  - User interface
    - ブラウザの window 以外の箇所
  - Browser engine
    - User interface と Rendering engine の仲介
  - Rendering engine
    - html と css の解析、結果を画面に表示
  - Networking
    - ネットワーク層、XHR など、ネットワークを呼び出す
  - UI backend
    - window やチェックボックスなどのコアウィジェット
    - プラットフォーム固有ではない汎用インターフェースを公開
  - JavaScript engine
    - 今まで見てきたやつ
  - Data persistence
    - データの保存先
    - localstorage
    - indexDB
    - WebSQL
    - FileSystem
- Rendering engine とは
  - request されたページを画面に描画する
  - ブラウザで異なる
    - Gecko
      - Firefox
    - WebKit
      - safari
    - Blink
      - Chrome, opera
- Rendering process
  - ![1*9b1uEMcZLWuGPuYcIn7ZXQ.png](https://miro.medium.com/max/1001/1*9b1uEMcZLWuGPuYcIn7ZXQ.png)
  - html を parse して DOM tree を構築
    - [オブジェクト モデルの構築  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model?hl=ja)
  - css を parse して CSSOM 作成
  - DOM と CSSOM で render tree 構築
    - DOM tree のルート `html`からたどっていく
    - css に応じて DOM の node が消えたりする
      - ex) disolay: none
    - head の link や scripts などのタグは解析はされるもののページに表示されないため省略
    - 表示することに決定された node に対して CSSOM の適用
    - 最終結果を render tree として出力
  - レイアウトの決定
    - 位置とサイズの決定
  - ペイント
    - render tree を操作して画面にコンテンツを描画
    - global
      - tree 全体の描画
    - incremental
      - ネットワークでコンテンツを取得後に描画するような場合、render tree に増加が合ったときに描画
    - すべての HTML が解析されて render tree の構築とレイアウトが開始されるまでペイントは開始しない
- script stylesheet の処理
  - `<script>`に達すると読み込みが開始
  - フェッチが終わるまで解析が停止
  - html5 では async が追加されたので、その場合は別スレッドで解析実行される。
- 最適化の 5 ポイント
  - 開発者がコントロールできる領域
  1. JS
     - settimeout みたいなのを視覚的な更新のために使用するのを避ける
     - Web Worker を使用する
     - micro task を利用する
       - [JavaScript とイベントループ | TOAST Meetup](https://meetup-jp.toast.com/896)
  2. style calculations
  - 要素の変更で style は計算を余儀なくされる
  - セレクタの複雑さの軽減
  - 計算が必要な要素の軽減
  3. layout
     - body の幅みたいな、他の要素がレイアウトの計算に影響を与えるのを考える
     - with,height,left,top みたいなものはレイアウトの再計算が必要なので、変更を抑える
     - js でアクセスする要素のスタイルが変更されると、そのスタイルを見ていた処理が遅れることになるので避ける
  4. paint
     - opacity、transform 以外のプロパティはペイントが走るので避ける
     - レイアウトの変更はペイントの変更も伴うので避ける
  5. compositing

## [How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security](https://blog.sessionstack.com/how-javascript-works-inside-the-networking-layer-how-to-optimize-its-performance-and-security-f71b7414d34c)

- 最近のブラウザは効率的かつ安全な配信をするために設計されている
- ブラウザの全体的なパフォーマンスはネットワークスタックも関わる
  - TCP,TLS みたいなやつ
- 基本的にアプリケーションの開発でそれらを意識する必要はないが、知っているとより高速で安全なアプリケーション作成の助けになる
- ユーザとブラウザの対話の流れ
- ![1*rjBdCBwOx5Gp_A6b6FQgfw.png](https://miro.medium.com/max/1117/1*rjBdCBwOx5Gp_A6b6FQgfw.png)

  - アドレスバーに URL を入力
  - ローカルキャッシュと app キャッシュのチェック
  - なければ URL からドメイン名を取得、DNS から IP アドレスを要求
    - ドメインがキャッシュ済みの場合、DNS クエリは不要
  - ブラウザはリモートサーバーにある web ページを要求する http パケットを作成
  - パケットは TCP レイヤーに送信される
    - ここで http パケットに独自の情報を追加する
  - パケットを IP 層に送る
    - リモートサーバーにパケットを送信する方法を見つける
  - パケットがリモートサーバーに送信される
  - パケットがリモートサーバーに送信されると、同様の方法で response が返ってくる

- ソケット
  - [ソケット
    ](http://research.nii.ac.jp/~ichiro/syspro98/socket.html)
  - js,wasm ではここのネットワークソケットのライフサイクルを管理できない
  - ブラウザがソケット管理を自動で行い、最適化してくれている
    - ソケットはプール管理され、保留中の要求はプールされているソケットに割り当てられる
    - ![1*0e8X3UTBpsiBSZKa3l1hXA.png](https://miro.medium.com/max/913/1*0e8X3UTBpsiBSZKa3l1hXA.png)
    - 新しい TCP 接続の開放のコストを抑えられる
  - ブラウザによって自動化されるとはいっても、開発者が適切なアプリケーション構成を選ぶことで更にパフォーマンスを高められる
  - ブラウザがソケットを管理することで、信頼できないアプリケーションソースに制約を課せられる
    - TLS
    - same-origin policy
      - [同一オリジンポリシー - ウェブセキュリティ | MDN](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy)
- 最速のリクエストはキャッシュ
  - [HTTP: Optimizing Application Delivery - High Performance Browser Networking (O'Reilly)](https://hpbn.co/optimizing-application-delivery/#cache-resources-on-the-client)
  - cookie jar でブラウザ cookie を管理している
    - オリジンごとに jar は違う
- Web アプリのパフォーマンスとセキュリティを改善するためにできること
  - リクエストでは常に「Connection：Keep-Alive」ヘッダーを使用
  - 適切な Cache-Control、Etag、および Last-Modified ヘッダーを使用
  - 常に TLS を使用

## [How JavaScript works: Under the hood of CSS and JS animations + how to optimize their performance](https://blog.sessionstack.com/how-javascript-works-under-the-hood-of-css-and-js-animations-how-to-optimize-their-performance-db0e79586216)

## [How JavaScript works: Storage engines + how to choose the proper storage API](https://blog.sessionstack.com/how-javascript-works-storage-engines-how-to-choose-the-proper-storage-api-da50879ef576)

## [How JavaScript works: the internals of Shadow DOM + how to build self-contained components](https://blog.sessionstack.com/how-javascript-works-the-internals-of-shadow-dom-how-to-build-self-contained-components-244331c4de6e)

## [How JavaScript works: WebRTC and the mechanics of peer to peer networking](https://blog.sessionstack.com/how-javascript-works-webrtc-and-the-mechanics-of-peer-to-peer-connectivity-87cc56c1d0ab)
