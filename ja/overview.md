<!-- pre-align:aligned sig=154b1ec7fbd6 -->

<a id="easyqueue-overview"></a>
## Data & Analytics > EasyQueue > 概要 { #easyqueue-overview }

NHN Cloud EasyQueue は、別途のインフラ構築や複雑なクラスター管理の負担なく、NHN Cloud が提供する完全マネージド型のパブリック Kafka クラスターを通じて、すぐにトピックを作成して活用できるメッセージキューサービスです。
ユーザーは Kafka トピックを通じてアプリケーション間のデータを非同期的に発行およびサブスクライブし、柔軟なデータパイプラインを手軽に構成できます。
また、メッセージはクラスター内に分散保存および多重レプリカされるため、障害発生時もデータ損失を防ぎます。受信アプリケーションが一時停止した場合でも、キュー (queue) に保管されたメッセージを通じて安定した処理を保証します。

<a id="service-access-path"></a>
## サービスへのアクセス経路 { #service-access-path }

NHN Cloudコンソールで**Data & Analytics > EasyQueue**を選択して、サービスにアクセスできます。

<a id="main-features"></a>
## 主な機能 { #main-features }

<a id="topic-creation-and-lifecycle-management"></a>
### トピックの作成とライフサイクル管理 { #topic-creation-and-lifecycle-management }

Webコンソールを通じて、複雑なコマンドや設定ファイルなしで、クリックのみで簡単にトピックを作成及び削除できます。
データの保管周期や最大メッセージサイズなど、トピックごとの詳細なポリシーをサービス要件に合わせて柔軟に設定できます。
パーティション数を調整することで、トラフィック規模に応じた処理パフォーマンスを最適化できます。

<a id="monitoring-dashboard"></a>
### モニタリングダッシュボード { #monitoring-dashboard }

トピック別のデータの流入量、メッセージ数などの指標を確認できます。

<a id="message-sendreceive-test"></a>
### メッセージ送受信テスト { #message-sendreceive-test }

別途のクライアントアプリケーションやコードを作成しなくても、コンソール内で直接テストメッセージを発行できます。
特定のトピックに積載されたメッセージを照会して確認できます。
初期連携段階の通信状態を点検したり、データフォーマットを検証するデバッグツールとして活用できます。

<a id="consumer-group-monitoring"></a>
### コンシューマーグループのモニタリング { #consumer-group-monitoring }

コンシューマーグループとグループ別のコンシューマー一覧を確認できます。
コンシューマーグループの処理状態を一目で確認し、Lagの数値を確認することで、処理パフォーマンスを迅速に把握できます。

<a id="how-easyqueue-works"></a>
## EasyQueueの動作方式 { #how-easyqueue-works }

![[図1] EasyQueueの動作方式](http://static.toastoven.net/prod_easyqueue/15_data&analytics_easyqueue_img_jp.png) 

1. メッセージ発行：プロデューサーがEasyQueueの特定のトピックへデータを送信します。
➋ メッセージキューイング：受信したメッセージはEasyQueueのクラスター内に分散保存され、大量のトラフィック流入時にも損失なく保管されます。
3. メッセージ購読：コンシューマーがキューに保存されたメッセージを取得し、ビジネスロジックに合わせてデータを処理します。

<a id="service-terms"></a>
## サービス用語 { #service-terms }

| 用語 | 説明 |
| --- | --- |
| メッセージキュー | 分散された環境で、プロセスやプログラムのシステム間でデータを交換するために使用される通信手法 |
| ブローカー | 生産者からメッセージを受け取ってメッセージを保存し、消費者に提供するサーバー |
| トピック | 関連するテーマに対するメッセージのグループ化単位 |
| パーティション | トピックに設定されたパーティション数に応じて、複数のパーティションにデータが分割されて保存される |
| プロデューサー | トピックへメッセージを送信する主体 |
| コンシューマー | 特定のトピックをサブスクライブし、メッセージを受信して消費する主体 |
| コンシューマーグループ | 同じトピックをサブスクライブする複数の消費者で構成されるグループ |

<a id="table-of-contents"></a>
## 主な目次 { #table-of-contents }

* [コンソール使用ガイド](./console-guide/)
* [APIガイド](./public-api/)
* [APIエラーコード](./api-error-codes/)
* [リリースノート](./release-notes/)
