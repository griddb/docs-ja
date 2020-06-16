## メタテーブルとは
# <a id="label_metatables"></a> メタテーブル


GridDBの管理用のメタデータを参照することができるテーブル群です。

<!--[!NOTE]-->
>#### メモ
>- メタテーブルは、参照のみ可能です。データの登録や削除はできません。
>- [SELECT](#select)で参照する際、メタテーブル名は引用符"で囲んでください。

<!--[!WARNING]-->
>#### 注意
>- メタテーブルのスキーマは将来のバージョンで変更される可能性があります。


## テーブル情報

テーブルに関する情報を取得できます。

**テーブル名**

\#tables

**スキーマ**

| 列名                        | 内容                                                | 型      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME              | データベース名                                      | STRING  |
| TABLE_NAME                 | テーブル名                                          | STRING  |
| TABLE_OPTIONAL_TYPE         | テーブル種別 <br>COLLECTION / TIMESERIES           | STRING  |
| DATA_AFFINITY              | データアフィニティ                                  | STRING  |
| EXPIRATION_TIME            | 期限解放経過時間                                    | INTEGER |
| EXPIRATION_TIME_UNIT        | 期限解放経過単位                                    | STRING  |
| EXPIRATION_DIVISION_COUNT   | 期限解放分割数                                      | STRING  |
| COMPRESSION_METHOD         | 時系列圧縮方式                                      | STRING  |
| COMPRESSION_WINDOW_SIZE     | 時系列圧縮間引き最大期間                            | INTEGER |
| COMPRESSION_WINDOW_SIZE_UNIT | 時系列圧縮間引き最大期間単位                        | STRING  |
| PARTITION_TYPE             | パーティショニング種別                              | STRING  |
| PARTITION_COLUMN           | パーティショニングキー                              | STRING  |
| PARTITION_INTERVAL_VALUE    | 分割値（インターバル/インターバルハッシュの場合）   | INTEGER |
| PARTITION_INTERVAL_UNIT     | 分割単位（インターバル/インターバルハッシュの場合） | STRING  |
| PARTITION_DIVISION_COUNT    | 分割数（ハッシュの場合）                            | INTEGER |
| SUBPARTITION_TYPE          | パーティショニング種別<br>（インターバルハッシュの場合にHASH）| STRING  |
| SUBPARTITION_COLUMN        | パーティショニングキー<br>（インターバルハッシュの場合）| STRING  |
| SUBPARTITION_INTERVAL_VALUE | 分割値                                              | INTEGER |
| SUBPARTITION_INTERVAL_UNIT  | 分割単位                                            | STRING  |
| SUBPARTITION_DIVISION_COUNT | 分割数 <br>（インターバルハッシュの場合）             | INTEGER |
| EXPIRATION_TYPE            | 期限解放種別  <br>ROW / PARTITION                  | STRING  |

## 索引情報

索引に関する情報を取得できます。

**テーブル名**

\#index_info

**スキーマ**

| 列名             | 内容                         | 型     |
|------------------|------------------------------|--------|
| DATABASE_NAME    | データベース名                | STRING |
| TABLE_NAME       | テーブル名                    | STRING |
| INDEX_NAME       | 索引名                        | STRING |
| INDEX_TYPE       | 索引種別 <br>TREE / HASH / SPATIAL     | STRING |
| ORDINAL_POSITION | 索引内のカラム列順序(1からの連番)         | SHORT  |
| COLUMN_NAME      | 列名                         | STRING |

## パーティショニング情報

パーティショニングされたテーブルの内部コンテナ(データパーティション)に関する情報を取得することができます。

**テーブル名**

\#table_partitions

**スキーマ**

| 列名                    | 内容                                   | 型     |
|-------------------------|---------------------------------------|--------|
| DATABASE_NAME           | データベース名                         | STRING |
| TABLE_NAME              | パーティショニングされたテーブルの名前   | STRING |
| PARTITION_BOUNDARY_VALUE | データパーティションの下限値            | STRING |


#### 仕様

- ロウ1件がひとつのデータパーティションの情報を表します。
  - 例えば、分割数128のハッシュパーティショニングテーブルの場合、TABLE_NAMEにテーブル名を指定して検索するとロウが128個表示されます。
- 上記の列以外にも複数の列が表示されます。
- 下限値でソートする場合、対象のパーティショニングキーの型に合わせてキャストする必要があります。

#### 例

- データパーティションの数を確認する

  ``` example
  SELECT COUNT(*) FROM "#table_partitions" WHERE TABLE_NAME='myIntervalPartition';

  COUNT(*)
  -----------------------------------
   8703
  ```

- データパーティションの下限値を確認する

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions" WHERE TABLE_NAME='myIntervalPartition'
  ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-10-30T10:00:00Z
  2017-01-29T10:00:00Z
          :
  ```

- インターバルパーティショニングのテーブル「myIntervalPartition2」(パーティショニングキーの型：INTEGER、分割基準値 20000)のデータパーティションの下限値一覧を確認する

  ``` example
  SELECT CAST(PARTITION_BOUNDARY_VALUE AS INTEGER) V FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalPartition2' ORDER BY V;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  -5000
  15000
  35000
  55000
    :
  ```

## ビュー情報

ビューに関する情報を取得できます。

**テーブル名**

\#views

**スキーマ**

| 列名                       | 内容                                                | 型      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME              | データベース名                                      | STRING  |
| VIEW_NAME                  | ビュー名                                            | STRING  |
| VIEW_DEFINITION            | ビュー定義文字列                                    | STRING  |

## 実行中SQL情報

実行中のSQL(クエリまたはジョブ)に関する情報を取得できます。

**テーブル名**

\#sqls

**スキーマ**

| 列名                       | 内容                                                | 型       |
|----------------------------|-----------------------------------------------------|----------|
| DATABASE_NAME              | データベース名                                      | STRING   |
| NODE_ADDRESS               | 処理実行中のノードのアドレス(system)                | STRING   |
| NODE_PORT                  | 処理実行中のノードのポート(system)                  | INTEGER  |
| START_TIME                 | 処理開始時刻                                        | TIMESTAMP|
| APPLICATION_NAME           | アプリケーション名                                  | STRING   |
| SQL                        | クエリ文字列                                        | STRING   |
| QUERY_ID                   | クエリID                                            | STRING   |
| JOB_ID                     | ジョブID                                            | STRING   |

## 実行中イベント情報

実行中のイベントに関する情報を取得できます。

**テーブル名**

\#events

**スキーマ**

| 列名                       | 内容                                                | 型       |
|----------------------------|-----------------------------------------------------|----------|
| NODE_ADDRESS               | 処理実行中のノードのアドレス(system)                | STRING   |
| NODE_PORT                  | 処理実行中のノードのポート(system)                  | INTEGER  |
| START_TIME                 | 処理開始時刻                                        | TIMESTAMP|
| APPLICATION_NAME           | アプリケーション名                                  | STRING   |
| SERVICE_TYPE               | サービス種別(SQL/TRANSACTION/CHECKPOINT/SYNC)       | STRING   |
| EVENT_TYPE                 | イベント種別(PUT/CP_START/SYNC_START など)          | STRING   |
| WORKER_INDEX               | ワーカーのスレッド番号                              | INTEGER  |
| CLUSTER_PARTITION_INDEX    | クラスタパーティション番号                          | INTEGER  |

## コネクション情報

接続中のコネクションに関する情報を取得できます。

**テーブル名**

\#sockets

**スキーマ**

| 列名                       | 内容                                                | 型       |
|----------------------------|-----------------------------------------------------|----------|
| SERVICE_TYPE               | サービス種別(SQL/TRANSACTION)                       | STRING   |
| SOCKET_TYPE                | ソケット種別                                        | STRING   |
| NODE_ADDRESS               | 接続元ノードのアドレス(ノード視点)                  | STRING   |
| NODE_PORT                  | 接続元ノードのポート(ノード視点)                    | INTEGER  |
| REMOTE_ADDRESS             | 接続先ノードのアドレス(ノード視点)                  | STRING   |
| REMOTE_PORT                | 接続先ノードのポート(ノード視点)                    | INTEGER  |
| APPLICATION_NAME           | アプリケーション名                                  | STRING   |
| CREATION_TIME              | ソケット生成時刻                                    | TIMESTAMP|
| DISPATCHING_EVENT_COUNT    | イベントハンドリングの要求を開始した総回数          | LONG     |
| SENDING_EVENT_COUNT        | イベント送信を開始した総回数                        | LONG     |

ソケット種別は次の通り。

|値        |説明|
|-|-|
|SERVER    |サーバ間同士のTCP接続       |
|CLIENT    |クライアントとのTCP接続     |
|MULTICAST |マルチキャストソケット      |
|NULL      |接続中など現時点で不明の場合|

#### 例

クライアントとのTCP接続(ソケット種別:CLIENT)の場合に限り、そのコネクションが
実行待ちかどうかを判別することができます。

具体的には、DISPATCHING_EVENT_COUNTの方がSENDING_EVENT_COUNTより大きい場合、
実行待ち状態のタイミングが存在した可能性が比較的高いと判定できます。

``` example
SELECT CREATION_TIME, NODE_ADDRESS, NODE_PORT, APPLICATION_NAME FROM "#sockets"
WHERE SOCKET_TYPE='CLIENT' AND DISPATCHING_EVENT_COUNT > SENDING_EVENT_COUNT;

CREATION_TIME             NODE_ADDRESS   NODE_PORT  APPLICATION_NAME
--------------------------------------------------------------------
2019-03-27T11:30:57.147Z  192.168.56.71  20001      myapp
2019-03-27T11:36:37.352Z  192.168.56.71  20001      myapp
          :
```
