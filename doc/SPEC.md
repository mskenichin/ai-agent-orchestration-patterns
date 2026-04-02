配送管理システム（Delivery Tracker）仕様書
1. プロジェクト概要
項目	内容
アプリ名	Delivery Tracker
概要	荷物の配送状況を管理するシンプルなWebアプリ
目的	GitHub Copilot ハンズオン用サンプルアプリケーション
2. 技術スタック
技術	バージョン	用途
Java	17	メイン言語
Spring Boot	3.x	Webフレームワーク
Thymeleaf	Spring Boot同梱	サーバーサイドテンプレートエンジン（HTMLの画面描画）
Spring Data JPA	Spring Boot同梱	データアクセス層
H2 Database	Spring Boot同梱	インメモリDB（設定不要・起動即使用可能）
Bootstrap	5.x（CDN読み込み）	UIスタイリング
Maven	3.x	ビルドツール
3. ドメインモデル
Delivery（配送）エンティティ
フィールド名	型	必須	説明
id	Long	✅	主キー（自動採番）
trackingNumber	String	✅	送り状番号（例：TRK-0001）
recipientName	String	✅	受取人名
recipientAddress	String	✅	届け先住所
status	DeliveryStatus	✅	配送ステータス
createdAt	LocalDateTime	✅	登録日時（自動設定）
DeliveryStatus（Enum）
値	表示名
PENDING	集荷待ち
DELIVERED	配達完了
4. 画面仕様（1画面のみ）
画面イメージ
┌──────────────────────────────────────────────────┐
│ 🚚 配送管理                                       │
├──────────┬───────────┬───────────┬────────────────┤
│ 送り状番号 │  受取人名  │ ステータス │      操作      │
├──────────┼───────────┼───────────┼────────────────┤
│ TRK-0001 │ 山田 太郎  │ 集荷待ち   │ [配達完了にする] │
│ TRK-0002 │ 鈴木 花子  │ 配達完了   │                │
└──────────┴───────────┴───────────┴────────────────┘

  ─── 新規登録フォーム ─────────────────────────────
  受取人名   [                    ]
  届け先住所  [                    ]
                               [ 登録する ]
  ──────────────────────────────────────────────────
URL・エンドポイント一覧
HTTPメソッド	URL	説明
GET	/	配送一覧表示（トップページ）
POST	/deliveries	新規配送を登録
POST	/deliveries/{id}/complete	ステータスを配達完了に更新
操作ボタン
現在のステータス	表示するボタン
PENDING	[配達完了にする]
DELIVERED	（なし）
5. サービス層仕様
DeliveryService メソッド一覧
メソッド名	引数	戻り値	説明
findAll()	-	List<Delivery>	全件取得（登録日時降順）
create(recipientName, recipientAddress)	String, String	Delivery	新規登録・送り状番号を自動採番
markAsDelivered(id)	Long	Delivery	ステータスを配達完了に更新
送り状番号の採番ルール
フォーマット：TRK-XXXX（4桁ゼロ埋め）
採番方法：現在の最大ID + 1 をゼロ埋めして生成
例：TRK-0001、TRK-0002、TRK-0003
6. 初期データ
アプリ起動時に以下の2件のデータが自動投入されること（data.sql）。

送り状番号	受取人名	届け先住所	ステータス
TRK-0001	山田 太郎	東京都新宿区西新宿1-1-1	PENDING
TRK-0002	鈴木 花子	東京都渋谷区道玄坂2-2-2	DELIVERED
7. application.properties 設定値
spring.application.name=delivery-tracker
server.port=8080

# H2 インメモリDB
spring.datasource.url=jdbc:h2:mem:deliverydb;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# 初期データ投入
spring.sql.init.mode=always
spring.sql.init.data-locations=classpath:data.sql

# H2コンソール（開発確認用）
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Thymeleaf
spring.thymeleaf.cache=false
