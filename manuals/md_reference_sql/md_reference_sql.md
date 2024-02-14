
# SQL記述形式

本章では、NewSQLインターフェースで使用できるSQLの記述形式について示します。



## 使用できる操作

SELECT文の他、CREATE TABLE等のDDL(Data Definition Language、データ定義言語)やINSERT/DELETE文などをサポートしています。詳細は[GridDBでサポートされるSQL文](#sql_commands_supported_by_griddb)を参照して下さい。

　

## データ型

<a id="data_types_used_in_data_storage"></a>
### データ格納に使用する型

NewSQLインターフェースでデータの格納に使用する型は次の通りです。この型名はテーブル作成時にカラム型として記述できます。

| データ型     | 内容詳細                                               |
|-------------|--------------------------------------------------------|
| BOOL型      | true/false                                             |
| BYTE型      | -2<sup>7</sup>から2<sup>7</sup>-1 (8ビット)の整数値     |
| SHORT型     | -2<sup>15</sup>から2<sup>15</sup>-1 (16ビット)の整数値  |
| INTEGER型   | -2<sup>31</sup>から2<sup>31</sup>-1 (32ビット)の整数値  |
| LONG型      | -2<sup>63</sup>から2<sup>63</sup>-1 (64ビット)の整数値  |
| FLOAT型     | 単精度型(32ビット) IEEE754で定められた浮動小数点数        |
| DOUBLE型    | 倍精度型(64ビット) IEEE754で定められた浮動小数点数        |
| TIMESTAMP型 | 日付と時刻の組。ミリ秒、マイクロ秒、ナノ秒のいずれか精度を用いるか指定が可能。指定がない場合はミリ秒精度。  |
| STRING型    | Unicodeコードポイントを文字とする、任意個数の文字の列    |
| BLOB型      | 画像や音声などのバイナリデータのためのデータ型<br>入力したままの形式で保存されるラージオブジェクト<br>文字xあるいはXをつけて、X'23AB'のような16進表現もできる   |

また、テーブルにNULL値を格納することができます。NULL値に対して“IS NULL”などの演算子を使用すると、SQL仕様に沿った結果を返却します。

### TIMESTAMP型の精度指定の方法

TIMESTAMP型は、日付と時刻の組を表現する型です。
時刻についてミリ秒、マイクロ秒、ナノ秒のいずれかの精度を指定できます。
TIMESTAMP(p)の形式で精度を指定します。精度指定値pには3,6,9のいずれかを使用できます。
これらの指定値付きのTIMESTAMP型を精度指定付きTIMESTAMP型と呼びます。
具体的には、以下の型名でミリ秒、マイクロ秒、ナノ秒のいずれかの精度を定義できます。

| 型名     | 内容詳細                                               |
|-------------|--------------------------------------------------------|
| TIMESTAMP | ミリ秒精度まで表現（デフォルトの精度）  |
| TIMESTAMP(3) | ミリ秒精度まで表現  |
| TIMESTAMP(6) | マイクロ秒精度まで表現 |
| TIMESTAMP(9) | ナノ秒精度まで表現  |

【メモ】
 - 時系列コンテナのロウキーはミリ秒精度のTIMESTAMP型に固定されていて、他の精度のTIMESTAMP型には変更はできません。

### テーブル作成時にカラム型として記述可能な表現

NewSQLインターフェースでは、テーブル作成時にカラム型として記述された型名について、[データ格納に使用する型](#data_types_used_in_data_storage)で列挙した型名と一致しなくても、ルールに従って解釈しデータの格納に使用する型を決定します。

以下のルールを上から順にチェックし、合致したルールによってデータ格納に使用する型を決定します。 ルールのチェック時には記述した型名およびルールでチェックする文字列の大文字小文字は区別しません。
複数のルールに合致した場合はより上にあるルールが優先されます。
どのルールにも当てはまらない場合はエラーとなりテーブル作成に失敗します。

| ルールNo. | テーブル作成時にカラム型として記述した識別子           | 作成するテーブルのカラム型         |
|-----------|----------------------------------------------------|------------------------------------|
| 1         | [データ格納に使用する型](#data_types_used_in_data_storage)に列挙した型名 | テーブル作成時に指定された型に従う |
| 2         | REAL                                             | DOUBLE型                           |
| 3         | TINYINT                                          | BYTE型                             |
| 4         | SMALLINT                                         | SHORT型                            |
| 5         | BIGINT                                           | LONG型                             |
| 6         | INTを含む型名                                     | INTEGER型                          |
| 7         | CHAR, CLOB, TEXTのいずれかを含む型名               | STRING型                           |
| 8         | BLOBを含む型名                                    | BLOB型                             |
| 9         | REAL, DOUBのいずれかを含む型名                     | DOUBLE型                           |
| 10        | FLOAを含む型名                                    | FLOAT型                            |

上記ルールによるデータ型決定の例を示します。
-   記述した型名が"BIGINTEGER"→INTEGER型(ルール6)
-   記述した型名が"LONG"→LONG型(ルール1)
-   記述した型名が"TINYINT"→BYTE型(ルール3)
-   記述した型名が"FLOAT"→FLOAT型(ルール1)
-   記述した型名が"VARCHAR"→STRING型(ルール7)
-   記述した型名が"CHARINT"→INTEGER型(ルール6)
-   記述した型名が"BIGBLOB"→BLOB型(ルール8)
-   記述した型名が"FLOATDOUB"→DOUBLE型(ルール9)
-   記述した型名が"INTREAL"→INTEGER型(ルール6)
-   記述した型名が"FLOATINGPOINT"→INTEGER型(ルール6)
-   記述した型名が"DECIMAL"→エラー

NoSQLインターフェースのクライアントにおけるデータ型と同等の型をNewSQLインターフェイスで使用する場合は、以下のように記述してください。ただし、一部同等の型が存在せず、使用できないものがあります。

| NoSQLインターフェースのデータ型        | 同等の型となるNewSQLインターフェースのカラム型記述 |
|--------------------------------------|----------------------------------------------------|
| STRING(文字列型)                     | STRING または STRING型となる表現               |
| BOOL(ブール型)                       | BOOL                                         |
| BYTE(8ビット整数型)                  | BYTE または BYTE型となる表現                   |
| SHORT(16ビット整数型)                | SHORT または SHORT型となる表現                 |
| INTEGER(32ビット整数型)              | INTEGER または INTEGER型となる表現             |
| LONG(64ビット整数型)                 | LONG または LONG型となる表現                   |
| FLOAT(32ビット単精度浮動小数点数型)   | FLOAT または FLOAT型となる表現                  |
| DOUBLE(64ビット倍精度浮動小数点数型)  | DOUBLE または DOUBLE型となる表現                |
| TIMESTAMP(時刻型)                    | TIMESTAMP                                    |
| GEOMETRY(空間型)                     | テーブル作成時のカラム型には指定できません       |
| BLOB型                               | BLOB または BLOB型となる表現                  |
| 配列型                               | テーブル作成時のカラム型には指定できません       |

【メモ】
   - TIMESTAMP型については、NoSQLインターフェースを用いて精度情報を参照したうえで、NewSQLの精度指定付きTIMESTAMP型の精度指定値(TIMESTAMP(p)のp)を定義する必要があります。精度情報の参照方法については『[GridDB Java APIリファレンス](../md_reference_java_api/md_reference_java_api.html)』、もしくは『[GridDB C APIリファレンス](../md_reference_c_api/md_reference_c_api.html)』を参照してください。  

### コンテナをテーブルとしてアクセスするときのデータ型と値の扱い

NoSQLインターフェースのクライアントで作成したコンテナを、NewSQLインターフェースでアクセスする場合のコンテナのカラム型および値の扱いを以下に示します。

| コンテナのカラム型  | NewSQLにマッピングされるデータ型    | 値             |
|--------------------|----------------------------------|----------------|
| STRING型           | STRING型                         | 元の値と同一   |
| BOOL型             | BOOL型                           | 元の値と同一   |
| BYTE型             | BYTE型                           | 元の値と同一   |
| SHORT型            | SHORT型                          | 元の値と同一   |
| INTEGER型          | INTEGER型                        | 元の値と同一   |
| LONG型             | LONG型                           | 元の値と同一   |
| FLOAT型            | FLOAT型                          | 元の値と同一   |
| DOUBLE型           | DOUBLE型                         | 元の値と同一   |
| TIMESTAMP型        | TIMESTAMP型                      | 元の値と同一   |
| GEOMETRY型         | NULL定数と同等の型(Types.UNKNOWN) | 全ての値がNULL |
| BLOB型             | BLOB型                           | 元の値と同一   |
| 配列型             | NULL定数と同等の型(Types.UNKNOWN) | 全ての値がNULL |


【メモ】
   - TIMESTAMP型については、SQLの精度指定付きTIMESTAMP型の精度指定値(TIMESTAMP(p)のp)を確認したうえで、NoSQLのTIMESTAMP型の精度指定情報を設定する必要があります。精度情報の設定方法については『[GridDB Java APIリファレンス](../md_reference_java_api/md_reference_java_api.html)』
、もしくは『[GridDB C APIリファレンス](../md_reference_c_api/md_reference_c_api.html)』を参照してください。  
   
### SQLでサポートしていないデータ型の扱い

NoSQLインタフェースでサポートしているが、NewSQLインタフェースではサポートしていないデータ型は次の通りです。

- GEOMETRY型
- 配列型

これらのデータ型のデータに対して、NewSQLインタフェースでアクセスした場合の扱いについて説明します。

- テーブル作成 CREATE TABLE
  - テーブル作成時のカラムのデータ型として、これらのデータ型は指定できません。エラーになります。

- テーブル削除 DROP TABLE
  - 削除対象テーブルがこれらのデータ型のカラムを持っていても、テーブル削除はできます。

- 登録/更新/削除 INSERT/UPDATE/DELETE
  - これらのデータ型のカラムを持つテーブルに対しては、INSERT/UPDATE/DELETEはエラーになります。
  - これらのデータ型のカラムの値は指定せずに、サポート範囲のデータ型のカラムの値だけ指定しても、ロウを登録・更新することはできません。

    ```
    // NoSQLインタフェースを用いて作成したテーブル
    　名前  ： sample1
    　カラム： id INTEGER型
              value DOUBLE型
              geometry GEOMETRY型

    // INTEGER型とDOUBLE型のカラムのみ指定してロウの登録　→テーブルにGEOMETRY型のカラムがあるので、エラーが発生する
    INSERT INTO sample1 (id, value) VALUES (1, 192.3)
    ```

- 検索 SELECT
  - これらのデータ型のカラムを持つテーブルを検索すると、そのカラムの値は常にNULLが返ります。


- 索引作成/削除 CREATE INDEX/DROP INDEX
  - GEOMETRY型カラムへの索引作成・削除は可能です。
  - 配列型カラムへの索引作成・削除はできません。エラーになります。（NoSQLインタフェースでも、配列型カラムへの索引作成・削除は未サポートです）

　

## ユーザとデータベース

GridDBのユーザには、管理ユーザと一般ユーザの2種類があり、利用できる機能に違いがあります。 また、データベースを作成することで、利用ユーザ単位にアクセスを分離することができます。
ユーザ、データベースの詳細は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。



## ネーミングの規則

ネーミングの規則は次の通りです。

-   データベース名・テーブル名・ビュー名・列名・索引名および一般ユーザ名は、1文字以上のASCII英数字ならびにアンダースコア「\_」、ハイフン「-」、ドット「.」、スラッシュ「/」、イコール「=」の列で構成されます。
-   テーブル名にはノードアフィニティ機能向けに「@」の文字も指定できます。

ノードアフィニティ機能、ネーミングの規則・制限についての詳細は、『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。


[メモ]
- 名前にASCII英数字とアンダースコア以外の文字を含む、または、先頭文字が数字のテーブルやカラムなどをSQL文に記述する場合は、引用符"で囲んでください。

  ```
  SELECT "column.a1" FROM "Table-5"
  ```


<a id="sql_commands_supported_by_griddb"></a>
# GridDBでサポートされるSQL文

サポートされるSQL文は、次の通りです。


| コマンド                             | 概要                                                |
|-------------------------------------|----------------------------------------------------|
| [CREATE DATABASE](#create-database) | データベースを作成する。                             |
| [CREATE TABLE](#create-table)       | テーブルを作成する。                                 |
| [CREATE INDEX](#create-index)       | 索引を作成する。                                     |
| [CREATE VIEW](#create-view)         | ビューを作成する。                                   |
| [CREATE USER](#create-user)         | 一般ユーザを作成する。                               |
| [CREATE ROLE](#create-role)         | ロールを作成する。                               |
| [DROP DATABASE](#drop-database)     | データベースを削除する。                             |
| [DROP TABLE](#drop-table)           | テーブルを削除する。                                 |
| [DROP INDEX](#drop-index)           | 索引を削除する。                                     |
| [DROP VIEW](#drop-view)             | ビューを削除する。                                   |
| [DROP USER](#drop-user)             | 一般ユーザを削除する。                               |
| [DROP ROLE](#drop-role)             | ロールを削除する。                               |
| [ALTER TABLE](#alter-table)         | テーブルの構造を変更します。                         |
| [GRANT](#grant)                     | 一般ユーザにデータベースへのアクセス権を設定する。     |
| [REVOKE](#revoke)                   | 一般ユーザからデータベースへのアクセス権を削除する。   |
| [SET PASSWORD](#set-password)       | 一般ユーザのパスワードを変更する。                   |
| [SELECT](#select)                   | データを取得する。                                   |
| [INSERT](#insert)                   | テーブルに行を挿入する。                             |
| [DELETE](#delete)                   | テーブルから行を削除する。                           |
| [UPDATE](#update)                   | テーブルにある行を更新する。                         |
| [コメント](#comment)                | コメントを表記する。                                 |
| [ヒント](#hint)                     | 実行計画を制御する。                                 |

本章では、SQL文の分類ごとに説明を行います。


## データ定義言語(DDL)

### CREATE DATABASE

データベースを作成します。

**構文**

|                                   |
|-----------------------------------|
| CREATE DATABASE *データベース名* ; |

**仕様**

-   管理ユーザのみが実行可能です。
-   「public」、「information_schema」と同一の名前のデータベースは、GridDBの内部用に予約済みのため作成できません。
-   既に同一の名前のデータベースが存在した場合は何も変更しません。
-   データベース名の規則は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。

### CREATE TABLE

#### テーブルの作成

テーブルを作成します。

**構文**

- テーブル(コレクション)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] \[, PRIMARY KEY(列名 \[, ...\])\] ) <br> \[WITH (プロパティキー=プロパティ値)\]; |

- 時系列テーブル(時系列コンテナ)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( *列名* TIMESTAMP PRIMARY KEY \[, **列定義** ...\] ) <br>USING TIMESERIES \[WITH (プロパティキー=プロパティ値 \[, プロパティキー=プロパティ値 ...\])\] ; |

- **列定義**
  - *列名* *型名* \[ **列制約** \]

- **列制約**
  - PRIMARY KEY (先頭の列のみ指定可)
  - NULL
  - NOT NULL

**仕様**

- テーブル名、列名の規則は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- “IF NOT EXISTS”が指定された場合、指定したテーブル名と同じ名前のテーブルが存在しないときのみ作成します。
- **列定義** では、列名と型名の指定が必須です。指定可能な型名は[データ格納に使用する型](#data_types_used_in_data_storage)を参照してください。
- テーブル(コレクション)では列定義の記述後、PRIMARY KEYによる複合主キーの設定が可能です。複合主キーは先頭カラムより順に設定することが必要で、上限数は16です。列制約におけるPRIMARY KEYと同時に設定することはできません。また、時系列テーブル(時系列コンテナ)では設定できません。
- 時系列テーブル（時系列コンテナ）については『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- データアフィニティに関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。


<a id="label_data_affinity_property"></a>

  | 機能                | 内容       | プロパティキー           | プロパティ値の型                             |
  |---------------------|------------|--------------------------|----------------------------------------------|
  | データアフィニティ  | ヒント情報 <br>(コンテナ間の類似性を示す文字列) | data_affinity            | STRING型                                     |

- 各項目の内容については『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。

**例**

- テーブルの作成

  ``` example
  CREATE TABLE myTable (
    key INTEGER PRIMARY KEY,
    value1 DOUBLE NOT NULL,
    value2 DOUBLE NOT NULL
  );
  ```

#### パーティショニングテーブルの作成

パーティショニングテーブルを作成します。

各パーティショニングの機能については、『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。

**(1) ハッシュパーティショニングテーブルの作成**

**構文**

- テーブル(コレクション)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] [, PRIMARY KEY(列名 \[, ...\])\] ) <br> \[WITH (プロパティキー=プロパティ値)\] <br>PARTITION BY HASH (*パーティショニングキーの列名*) PARTITIONS *分割数* ; |

- 時系列テーブル(時系列コンテナ)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] ) <br>USING TIMESERIES \[WITH (プロパティキー=プロパティ値, ...)\]\] <br>PARTITION BY HASH (*パーティショニングキーの列名*) PARTITIONS *分割数* ; |

**仕様**

- 指定されたパーティショニングキーの列名と分割数の値により、ハッシュパーティショニングテーブルを作成します。
- 「分割数」は、1以上かつ1024以下の値を指定してください。
- パーティショニングキーには主キーを設定する必要があります。主キー以外を設定する場合は、コンフィグファイルで制限を解除する必要があります。詳細は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』のクラスタ定義ファイルの設定を参照してください。
- パーティショニングキーに指定した列の値は、更新できません。

***オプション指定***

****データアフィニティ****
- データアフィニティに関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。指定可能なオプションは[通常のテーブル](#label_data_affinity_property)と同様です。

**例**

- ハッシュパーティショニングテーブルの作成

  ``` example
  CREATE TABLE myHashPartition (
    id INTEGER PRIMARY KEY,
    value STRING
  ) PARTITION BY HASH (id) PARTITIONS 128;
  ```

**(2) インターバルパーティショニングテーブルの作成**

**構文**

- テーブル(コレクション)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] [, PRIMARY KEY(列名 \[, ...\])\]) <br>\[WITH (プロパティキー=プロパティ値, ...)\] <br>PARTITION BY RANGE(*パーティショニングキーの列名*) EVERY(*分割幅値* \[, *単位* \]) ; |

- 時系列テーブル(時系列コンテナ)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] ) <br>USING TIMESERIES \[WITH (プロパティキー=プロパティ値, ...)\] <br>PARTITION BY RANGE(*パーティショニングキーの列名*) EVERY(*分割幅値* \[, *単位* \]) ; |

**仕様**

- 「パーティショニングキーの列名」には、BYTE型、SHORT型、INTEGER型、LONG型、TIMESTAMP型のいずれかの列を指定してください。
- パーティショニングキーには主キーを設定する必要があります。主キー以外を設定する場合は、コンフィグファイルで制限を解除する必要があります。詳細は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』のクラスタ定義ファイルの設定を参照してください。
- パーティショニングキーに指定した列の値は、更新できません。
- 分割幅値には次の値が指定できます。

  | パーティショニングキーの型   | 分割幅値に指定できる値    |
  |----------------------------|---------------------------|
  | BYTE型                     | 1～2<sup>7</sup>-1                    |
  | SHORT型                    | 1～2<sup>15</sup>-1                  |
  | INTEGER型                  | 1～2<sup>31</sup>-1             |
  | LONG型                     | 1000～2<sup>63</sup>-1 |
  | TIMESTAMP型                | 1以上                     |

- TIMESTAMP型(精度指定付きTIMESTAMP型を含む)の列を指定した場合は、単位を指定する必要があります。単位に指定できる値は、DAYです。
- 上記以外の型の列を指定した場合は、単位は指定できません。

***オプション指定***

****データアフィニティ****
- データアフィニティに関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。
  指定可能なオプションは[通常のテーブル](#label_data_affinity_property)と同様です。

****期限開放****
- 期限解放に関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。

  | 機能          | 内容     | プロパティキー           | プロパティ値の型                                  | 期限解放設定時の必須項目 |
  |--------------|----------|-------------------------|---------------------------------------------------|-------------------------|
  | 期限解放機能  | 種別     | expiration_type          | STRING型 <br>(指定可能な値は次の1種類。<br> PARTITION: パーティション期限解放) | 省略可（デフォルト：PARTITION） |
  |              | 経過時間 | expiration_time          | INTEGER型                                                | 必須|
  |              | 経過単位 | expiration_time_unit      | STRING型 <br>(指定可能な値は次の5種類。<br> DAY / HOUR / MINUTE / SECOND / MILLISECOND )  | 省略可(デフォルト:DAY) |
  |              | 分割数   | expiration_division_count | INTEGER型                                    | 省略可(デフォルト:8) |

- パーティション期限解放は次の場合に指定できます。
  - 時系列テーブル(時系列コンテナ)
  - パーティショニングキーがTIMESTAMP型(精度指定付きTIMESTAMP型を含む)のテーブル(コレクション)
- 各項目の内容については『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。

【メモ】
 - 時系列コンテナの場合、ロウキーはミリ秒精度のTIMESTAMP型に固定されていて、他の精度のTIMESTAMP型には変更はできません。

****データパーティション配置***
- 各日付に対応するデータパーティションの配置先を決定するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で以下の値で指定することができます。

  | 機能                | 内容                                         | プロパティキー        | プロパティ値の型   | 設定時の必須項目 |
  |---------------------|----------------------------------------------|---------------------|-------------------|------------|
  |   区間グループ番号  | データパーティションの配置先を識別する番号。異なる区間グループ番号指定で生成されたテーブル同士は同一日時における処理スレッド競合が発生しない割付となる | interval_worker_group            | INTEGER型  | 必須 |
  | 区間グループノード補正値  | 区間グループ番号で決定される処理スレッドに対して、どのノードで実行するかを補正する値。ユーザが指定する以外にサーバに決定させることも可能である | interval_worker_group_position | INTEGER型  | 省略可(デフォルト:0) |

【メモ】
 - インターバルパーティショニングでパーティショニングキーがTIMESTAMP型のケースのみ適用可能となります。
 - 本機能を適用するためにには機能面、運用面から幾つかの条件がありますが、それらは、『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。

**例**

- インターバルパーティショニングテーブルの作成

  ``` example
  CREATE TABLE myIntervalPartition (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) PARTITION BY RANGE (date) EVERY (30, DAY);
  ```

- パーティション期限解放を使用するインターバルパーティショニングテーブル(時系列テーブル)の作成

  ``` example
  CREATE TABLE myIntervalPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) USING TIMESERIES WITH (
    expiration_type='PARTITION',
    expiration_time=90,
    expiration_time_unit='DAY'
  ) PARTITION BY RANGE (date) EVERY (30, DAY);
  ```

- データパーティションの配置先を指定したインターバルパーティショニングテーブルの作成

  ``` example
  CREATE TABLE myIntervalPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) WITH (
    interval_worker_group=1
  ) PARTITION BY RANGE (date) EVERY (1, DAY);
  ```

**(3) インターバル-ハッシュパーティショニングテーブルの作成**

**構文**

- テーブル(コレクション)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] [, PRIMARY KEY(列名 \[, ...\])\] ) <br>\[WITH (プロパティキー=プロパティ値, ...) \] <br>PARTITION BY RANGE(*インターバルパーティショニングキーの列名*) EVERY(*分割幅値* \[, *単位* \]) <br>SUBPARTITION BY HASH(*ハッシュパーティショニングキーの列名*) SUBPARTITIONS *分割数* ; |

- 時系列テーブル(時系列コンテナ)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] *テーブル名* ( **列定義** \[, **列定義** ...\] ) <br>USING TIMESERIES \[WITH (プロパティキー=プロパティ値, ...)\] <br>PARTITION BY RANGE(*インターバルパーティショニングキーの列名*) EVERY(*分割幅値* \[, *単位* \]) <br>SUBPARTITION BY HASH(*ハッシュパーティショニングキーの列名*) SUBPARTITIONS *分割数* ; |

**仕様**

- 「インターバルパーティショニングキーの列名」には、BYTE型、SHORT型、INTEGER型、LONG型、TIMESTAMP型のいずれかの列を指定してください。
- インターバルパーティショニングの分割幅値には次の値が指定できます。

  | パーティショニングキーの型   | 分割幅値に指定できる値                     |
  |----------------------------|--------------------------------------------|
  | BYTE型                     | 1～2<sup>7</sup>-1                                     |
  | SHORT型                    | 1～2<sup>15</sup>-1                                   |
  | INTEGER型                  | 1～2<sup>31</sup>-1                              |
  | LONG型                     | 1000×ハッシュの分割数～2<sup>63</sup>-1 |
  | TIMESTAMP型                | 1以上                                      |

  - TIMESTAMP型の列を指定した場合は、単位を指定する必要があります。単位に指定できる値は、DAYです。
  - TIMESTAMP型以外の列を指定した場合は、単位は指定できません。

- ハッシュパーティショニングの「分割数」は、1以上かつ1024以下の値を指定してください。
- パーティショニングキーには主キーを設定する必要があります。主キー以外を設定する場合は、コンフィグファイルで制限を解除する必要があります。詳細は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』のクラスタ定義ファイルの設定を参照してください。
- パーティショニングキーに指定した列の値は、更新できません。

***オプション指定***

****データアフィニティ****
- データアフィニティに関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。
  指定可能なオプションは[通常のテーブル](#label_data_affinity_property)と同様です。

****期限解放****
- 期限解放に関するオプションを" WITH (プロパティキー=プロパティ値, ...)"の形式で指定することができます。


  | 機能          | 内容     | プロパティキー           | プロパティ値の型                                  | 期限解放設定時の必須項目 |
  |--------------|----------|-------------------------|---------------------------------------------------|-------------------------|
  | 期限解放機能  | 種別     | expiration_type          | STRING型 <br>(指定可能な値は次の1種類。<br> PARTITION: パーティション期限解放) | 省略可（デフォルト：PARTITION） |
  |              | 経過時間 | expiration_time          | INTEGER型                                                | 必須|
  |              | 経過単位 | expiration_time_unit      | STRING型 <br>(指定可能な値は次の5種類。<br> DAY / HOUR / MINUTE / SECOND / MILLISECOND )  | 省略可(デフォルト:DAY) |
  |              | 分割数   | expiration_division_count | INTEGER型                                    | 省略可(デフォルト:8) |


**例**

- インターバルハッシュパーティショニングテーブルの作成

  ``` example
  CREATE TABLE myIntervalHashPartition (
    date TIMESTAMP,
    value STRING,
    PRIMARY KEY (date, value)
  ) PARTITION BY RANGE (date) EVERY (60, DAY)
  SUBPARTITION BY HASH (value) SUBPARTITIONS 64;
  ```

- パーティション期限解放を使用するインターバルハッシュパーティショニングテーブル(時系列テーブル)の作成

  ``` example
  CREATE TABLE myIntervalHashPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) USING TIMESERIES WITH (
    expiration_type='PARTITION',
    expiration_time=90,
    expiration_time_unit='DAY'
  ) PARTITION BY RANGE (date) EVERY (60, DAY)
  SUBPARTITION BY HASH (date) SUBPARTITIONS 64;
  ```

### CREATE INDEX

索引を作成します。

**構文**

|                                   |
|-----------------------------------|
| CREATE INDEX \[IF NOT EXISTS\] *索引名* ON *テーブル名* ( *索引をつける列名* \[, ...\] ) ; |

**仕様**

- 索引名の規則は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- 同じテーブルに対して、既に存在する索引と同じ名前の索引は作成できません。
- 処理対象のテーブルにおいて実行中のトランザクションが存在する場合、それらの終了を待機してから作成を行います。
- BLOB型と配列型の列には索引を作成できません。
- 複数のカラムを指定した索引を作成できます。これを複合索引と呼びます。
- 1つの複合索引に指定できるカラム数の上限は16個で、同じカラムを複数回指定することはできません。
- 時系列テーブルでは、複合索引に主キーを含むことはできません。

### CREATE VIEW

ビューを作成します。

**構文**

|                                   |
|-----------------------------------|
| CREATE \[FORCE\] VIEW *ビュー名* AS *SELECT文* ; |

**仕様**

- ビュー名の規則は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- SELECT文の参照可否のチェックを行います。参照できない場合は作成できません。
- FORCE指定時はSELECT文の参照可否のチェックを行いません。ただし、構文チェックは行われます。
- SELECT文中に他のビュー名を含むことができます。ただし、循環参照となる場合はFORCEを指定しても作成できません。

### CREATE USER

一般ユーザを作成します。

**構文**

|                                   |
|-----------------------------------|
| CREATE USER *ユーザ名* IDENTIFIED BY *‘パスワード文字列’* ; |

**仕様**

- ユーザ名の規則は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- 管理ユーザのみが実行可能です。
- インストール時に登録済の管理ユーザ(adminおよびsystem)と同一の名前のユーザは作成できません。
- パスワード文字列に使用できる文字は、ASCII文字のみです。大文字と小文字は区別します。

### CREATE ROLE

LDAP認証で必要なロールを作成します。

**構文**

|                                   |
|-----------------------------------|
| CREATE ROLE *ロール名* ; |

**仕様**

- ロール名には、LDAPユーザ名、もしくはLDAPグループ名を指定してください。詳細は『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。
- 管理ユーザのみが実行可能です。
- インストール時に登録済の管理ユーザ(adminおよびsystem)と同一の名前のユーザは作成できません。
- パスワード文字列に使用できる文字は、ASCII文字のみです。大文字と小文字は区別します。


### DROP DATABASE

データベースを削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP DATABASE *データベース名* ;   |

**仕様**

- 管理ユーザのみが実行可能です。
- 「public」、「information_schema」という名前のデータベース、および「gs\#」で始まる名前のデータベースは、GridDBの内部用に予約済みのため削除できません。
- ユーザが作成したテーブルが存在するデータベースは削除できません。

### DROP TABLE

テーブルを削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP TABLE \[IF EXISTS\] *テーブル名* ; |

**仕様**

- “IF EXISTS”が指定された場合、指定した名前のテーブルが存在しない場合は何も変更しません。
- 処理対象のテーブルにおいて実行中のトランザクションが存在する場合、それらの終了を待機してから削除を行います。

### DROP INDEX

指定された索引を削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP INDEX \[IF EXISTS\] *索引名* ON *テーブル名* ; |

**仕様**

- “IF EXISTS”が指定された場合、指定した名前の索引が存在しない場合は何も変更しません。
- 処理対象のテーブルにおいて実行中のトランザクションが存在する場合、それらの終了を待機してから削除を行います。
- NoSQLインターフェースから名前無しで付与した索引をDROP INDEXで削除することはできません。

### DROP VIEW

ビューを削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP VIEW \[IF EXISTS\] *ビュー名* ; |

**仕様**

- “IF EXISTS”が指定された場合、指定した名前のビューが存在しない場合は何も変更しません。

### DROP USER

一般ユーザを削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP USER *ユーザ名* ;             |

**仕様**

- 管理ユーザのみが実行可能です。

### DROP ROLE

LDAP認証で必要なロールを削除します。

**構文**

|                                   |
|-----------------------------------|
| DROP ROLE *ロール名* ; |

**仕様**

- 管理ユーザのみが実行可能です。



### ALTER TABLE

テーブルの構造を変更します。

#### カラムを追加する

テーブルの末尾にカラムを追加します。

**構文**

|                                   |
|-----------------------------------|
| ALTER TABLE *テーブル名* ADD \[COLUMN\] **列定義** \[,ADD \[COLUMN\] **列定義** ...\] ; |

- **列定義**
  - *列名* *型名* \[ **列制約** \]

- **列制約**
  - NULL
  - NOT NULL

**仕様**

- 追加するカラムは、テーブルの末尾に配置します。複数のカラムを指定した場合は指定した順序で配置します。
- 列制約に"PRIMARY KEY"は指定できません。
- 既に同じ名前のカラムが存在した場合はエラーになります。

**例**

- 複数のカラムの追加

  ``` example
  ALTER TABLE myTable1
    ADD COLUMN col111 STRING NOT NULL,
    ADD COLUMN col112 INTEGER;
  ```

#### データパーティションを削除する

テーブルパーティショニングで作成されたデータパーティションを削除します。

**構文**

|                                   |
|-----------------------------------|
| ALTER TABLE *テーブル名* DROP PARTITION FOR ( *削除するデータパーティションの区間(下限値から上限値)に含まれる値* ); |

**仕様**

- インターバルとインターバル-ハッシュパーティショニングの場合のみ、データパーティションを削除できます。
- 削除するデータパーティションの区間(下限値から上限値)に含まれる値を指定してください。
- 一度削除したデータパーティションの区間(下限値から上限値)に該当するデータの新規登録はできません。
- データパーティションの区間の下限値は、メタテーブルで確認できます。区間の上限値は多くの場合、下限値＋分割幅値の値です。
- インターバル-ハッシュパーティショニングの場合は、同じ下限値のデータパーティションが、ハッシュの分割数分(最大)存在します。
  データパーティションを削除する場合は、それらの同じ下限値をもつデータパーティションは同時に削除されます。削除の確認はメタテーブルで行います。

メタテーブルの詳細は[メタテーブル](#label_metatables)を参照してください。

**例**

**インターバルパーティショニングテーブル**

- インターバルパーティショニングのテーブル「myIntervalPartition1」(パーティショニングキーの型：TIMESTAMP、分割幅値30日)のデータパーティションの下限値一覧を確認する

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalPartition1' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
   2017-01-10T13:00:00.000Z
   2017-02-09T13:00:00.000Z
   2017-03-11T13:00:00.000Z
         :
  ```

- 不要なデータパーティションを削除する

  ``` example
  ALTER TABLE myIntervalPartition1 DROP PARTITION FOR ('2017-01-10T13:00:00Z');
  ```

**インターバル-ハッシュパーティショニングテーブル**

- インターバル-ハッシュパーティショニングのテーブル「myIntervalHashPartition」(インターバルパーティショニングキーの型：TIMESTAMP、分割幅値90DAY、ハッシュパーティショニングキーの分割数3)のデータパーティションの下限値一覧を確認する

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalHashPartition' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-08-01T10:00:00.000Z    同じ下限値のデータがハッシュされて3つの
  2016-08-01T10:00:00.000Z    データパーティションに分割されている
  2016-08-01T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2017-01-29T10:00:00.000Z
         :
  ```

- 不要なデータパーティションを削除する

  ``` example
  ALTER TABLE myIntervalHashPartition DROP PARTITION FOR ('2016-09-15T10:00:00Z');
  ```

- 同じ下限値のデータパーティションが削除される

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalHashPartition' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-10-30T10:00:00.000Z    '2016-09-15T10:00:00Z'を含む区間(下限値'2016-08-01T10:00:00Z')の
  2016-10-30T10:00:00.000Z    3つのデータパーティションが削除される
  2016-10-30T10:00:00.000Z
  2017-01-29T10:00:00.000Z
         :
  ```

#### カラム名を変更する

指定した既存のカラム名を変更します。

**構文**

|                                   |
|-----------------------------------|
| ALTER TABLE *テーブル名* RENAME COLUMN *変更前カラム名* TO *変更後カラム名*;|


**仕様**

- 変更前カラム名が指定したテーブルに存在しない場合は、エラーになります。
- 変更後カラム名が指定したテーブルに既に存在する場合は、エラーになります。

**例**

- カラム名の変更

  ``` example
  ALTER TABLE myTable1 RENAME COLUMN col112 TO col121;
  ```

　

## データ制御言語(DCL)

### GRANT

一般ユーザ、もしくはロールにデータベースへのアクセス権を付与します。

**構文**

|                                   |
|-----------------------------------|
| GRANT {SELECT\|ALL} ON *データベース名* TO {*ユーザ名*\|*ロール名*}; |

**仕様**

- 管理ユーザのみが実行可能です。
- SELECTは参照権限、ALLは参照権限と更新権限を表します。

### REVOKE

一般ユーザ、もしくはロールからデータベースへのアクセス権を剥奪します。

**構文**

|                                   |
|-----------------------------------|
| REVOKE {SELECT\|ALL} ON *データベース名* FROM {*ユーザ名*\|*ロール名*} ; |

**仕様**

- 管理ユーザのみが実行可能です。
- SELECTは参照権限、ALLは参照権限と更新権限を表します。

### SET PASSWORD

一般ユーザのパスワードを変更します。

**構文**

|                                   |
|-----------------------------------|
| SET PASSWORD \[FOR *ユーザ名* \] = *‘パスワード文字列’* ; |

**仕様**

- 管理ユーザは全ての一般ユーザのパスワードを変更可能です。
- 一般ユーザは自身のパスワードのみ変更可能です。

　　

## データ操作言語(DML)

### SELECT

データを取得します。FROM句、WHERE句など様々な[句](#label_clauses)から構成されます。

**構文**

|                                   |
|-----------------------------------|
| SELECT \[{ALL\|DISTINCT}\] * \| *列名1* \[, *列名2* ...\] <br>\[FROM句\] <br>\[WHERE句\] <br>\[GROUP BY句 \[HAVING句\]\] <br> \[{UNION \[ALL\] \|INTERSECT\|EXCEPT} SELECT文\]<br>\[ORDER BY句\] <br>\[LIMIT句 \[OFFSET句\]\] ; |


### INSERT

テーブルに行を登録します。INSERT句は単に行を登録し、INSERT OR REPLACE句とREPLACE句は、既に同一主キーが存在するデータを与えた場合、既存のデータを上書きします。REPLACE句はINSERT OR REPLACE句の別名で、機能の違いはありません。

**構文**

|                                   |
|-----------------------------------|
| {INSERT\|INSERT OR REPLACE\|REPLACE} INTO *テーブル名* <br>{VALUES ( { *数値1* \| *文字列1* } \[, { *数値2* \| *文字列2* } ...\] )\|SELECT文} ; |

**仕様**

- VALUESではなくSELECT文を指定した場合は、その実行結果が登録データになります。ただし、UNION/INTERSECT/EXCEPTを含む問い合わせ結果は登録できません。

``` example
INSERT INTO myTable1 VALUES(1, 100);

REPLACE INTO myTable1 VALUES(1, 200);

INSERT INTO myTable1 SELECT * FROM myTable2;
```

### DELETE

テーブルから行を削除します。

**構文**

|                                   |
|-----------------------------------|
| DELETE FROM *テーブル名* \[WHERE句\] ; |

### UPDATE

テーブルに存在する行を更新します。

**構文**

|                                   |
|-----------------------------------|
| UPDATE *テーブル名* SET *列名1* = *式1* \[, *列名2* = *式2* ...\] \[WHERE句\] ; |

**仕様**

- PRIMARY KEY制約のあるカラムの値は更新できません。
- パーティショニングを設定した場合、UPDATE句を使ってパーティショニングキーになっている項目を別の値に更新することはできません。このような場合は、DELETE文を実行後にINSERT文を実行してください。
  - 例)
    ``` example
    CREATE TABLE tab (a INTEGER, b STRING) PARTITION BY HASH a PARTITIONS 5;

    -- NG
    UPDATE tab SET a = a * 2;
    [240016:SQL_COMPILE_PARTITIONING_KEY_NOT_UPDATABLE] Partitioning column='a' is not updatable

    -- OK
    UPDATE tab SET b = 'XXX';
    ```

- SET句で指定する列名は、テーブル名で修飾することはできません。
  - 例)
    ``` example
    CREATE TABLE myTable1 (key INTEGER, value INTEGER);

    -- NG
    UPDATE myTable1 SET myTable1.value = 999 WHERE myTable1.key = 8;

    -- OK
    UPDATE myTable1 SET value = 999 WHERE myTable1.key = 8;
    ```

- サブクエリを更新値に利用することはできません。ただし、WHERE句などの条件文には利用可能です。
  - 例)
    ``` example
    CREATE TABLE myTable1 (key INTEGER, value INTEGER);
    
    -- NG
    UPDATE myTable1 SET value = (SELECT 999) WHERE key = 8;

    -- OK
    UPDATE myTable1 SET value = 999 WHERE key = (SELECT 8);
    ```



<a id="label_clauses"></a>
## 句

### FROM

データ操作を行うテーブル名またはビュー名、サブクエリを指定します。

**構文**

|                                   |
|-----------------------------------|
| FROM *テーブル名1* \[, *テーブル名2* ... \] |
| FROM ( *sub_query* ) [AS] 別名 \[, ... \] |

**仕様**

- サブクエリを指定する場合は、()で括り、別名を指定する必要があります。

例)
``` example
SELECT a.ID, b.ID FROM mytable a, (SELECT ID FROM mytable2) b;

ID     ID
---+-----
 1    100
 1    200
 2    100
 2    200
   :
```

### GROUP BY

前に指定された句の結果の中で、指定された列で同じ値を持った行をグループ化します。

**構文**

|                                   |
|-----------------------------------|
| GROUP BY *列名1* \[, *列名2* ...\] |

### HAVING

GROUP BY句によりグループ化された情報に対して探索条件で絞り込みを行います。GROUP BY句は省略できません。

**構文**

|                                   |
|-----------------------------------|
| HAVING 探索条件                    |

### ORDER BY

検索結果の並べ替え（ソート）を行います。

|                                   |
|-----------------------------------|
| ORDER BY *列名1* \[{ASC\|DESC}\] \[, *列名2* \[{ASC\|DESC}\] ...\] |

### WHERE

先行するFROM句の結果に、探索条件を適用します。

**構文**

|                                   |
|-----------------------------------|
| WHERE 探索条件                     |

**仕様**

- 探索条件は式や関数、サブクエリなどを用いて記述できます。

### LIMIT/OFFSET

指定した位置から指定した件数分のデータを取り出します。

**構文**

|                                   |
|-----------------------------------|
| LIMIT *値1* \[OFFSET *値2* \]      |

**仕様**

- 値1は取り出すデータ件数を表し、値2は取り出すデータ位置を表します。

　

### JOIN

テーブルを結合します。

**構文**

| 結合の種類   |  構文                 |
|-------------|-----------------------|
| 内部結合     | *テーブル1* [INNER] JOIN *テーブル2* [ ON *式* \| USING(*列名* [,*列名* ...]) ]  |
| 左外部結合   | *テーブル1* LEFT [OUTER] JOIN *テーブル2* [ ON *式* \| USING(*列名* [,*列名* ...]) ]   |
| クロス結合   | *テーブル1* CROSS JOIN *テーブル2* [ ON *式* \| USING(*列名* [,*列名* ...]) ] |

- 内部結合は、両方のテーブルの指定した列の値が一致する結果を返します。
- 左外部結合は、両方のテーブルの指定した列の値が一致する結果と、*テーブル1*にしか存在しない結果を返します。
- クロス結合は、内部結合(INNER JOIN)と等価です。

結合条件は、ON句またはUSING句を用いて指定します。

例)
```example
名前: employees

 id   first_name   department_id
----+------------+----------------
  0   John         0
  1   William      1
  2   Richard      0
  3   Mary         4
  4   Lisa         3
  5   James        1

名前: departments

 department_id   department   
---------------+------------
  0              Sales
  1              Development
  2              Research
  3              Marketing

○内部結合
SELECT * FROM employees e INNER JOIN departments d ON e.department_id=d.department_id;

 id    first_name  department_id  department_id  department
------+-----------+--------------+--------------+-----------
  0    John         0              0             Sales
  1    William      1              1             Development
  2    Richard      0              0             Sales
  4    Lisa         3              3             Marketing
  5    James        1              1             Development


○左外部結合
SELECT * FROM employees e LEFT JOIN departments d  ON e.department_id=d.department_id;

 id    first_name  department_id  department_id  department
------+-----------+--------------+--------------+-----------
  0    John         0              0             Sales
  1    William      1              1             Development
  2    Richard      0              0             Sales
  3    Mary         4              (NULL)        (NULL)
  4    Lisa         3              3             Marketing
  5    James        1              1             Development
```

　

自然結合(NATURAL JOIN)を用いると、指定されたテーブルの同じ名前のカラムの値が一致するかを結合条件として結合を行います。

| 結合の種類   |  構文                                              |
|-------------|----------------------------------------------------|
| 内部結合     | *テーブル1* NATURAL [INNER] JOIN *テーブル2*        |
| 左外部結合   | *テーブル1* NATURAL LEFT [OUTER] JOIN *テーブル2*   |
| クロス結合   | *テーブル1* NATURAL CROSS JOIN *テーブル2*          |

``` example
SELECT * FROM employees NATURAL INNER JOIN departments;

 department_id   id    first_name     department
---------------+-----+--------------+--------------
  0              0     John           Sales
  1              1     William        Development
  0              2     Richard        Sales
  3              4     Lisa           Marketing
  1              5     James          Development
```

### UNION/INTERSECT/EXCEPT

2つの問い合わせ結果の集合に対して演算を行います。

**構文**

|                              |                                                   |
|------------------------------|---------------------------------------------------|
| *問合せ1* UNION *問合せ2*     | 2つの問合せのすべての結果を返します (重複は含まない)  |
| *問合せ1* UNION ALL *問合せ2* | 2つの問合せのすべての結果を返します (重複を含む)      |
| *問合せ1* INTERSECT *問合せ2* | 2つの問合せの共通の結果を返します                    |
| *問合せ1* EXCEPT *問合せ2*    | 2つの問合せの差分(*問合せ1*に含まれていて*問合せ2*に含まれない結果)の結果を返します |


### OVER

問い合わせ結果の分割や、並び替えを行います。WINDOW関数と共に利用します。

**構文**

|                                   |
|-----------------------------------|
| *関数*　OVER ( \[PARTITION BY *式1* \] \[ORDER BY *式2* \] )     |

**仕様**


- SELECT句で利用できます。
- 対応する関数は以下です。
  - ROW_NUMBER()
  - LAG()
  - LEAD()
  - AVG()
  - COUNT()
  - MAX()
  - MIN()
  - SUM()/TOTAL()
  - STDDEV_SAMP()
  - STDDEV()/STDDEV0()
  - STDDEV_POP()
  - VAR_SAMP()
  - VARIANCE()/VARIANCE0()
  - VAR_POP()
- 関数の第一引数にDISTINCTを指定することはできません。
- PARTITION BY句で問い合わせ結果を分割します。ORDER BY句でロウの並び替えを行います。
- 同一SELECT句内でのWINDOW関数/OVER句の複数利用や、WINDOW関数/OVER句とMEDIAN関数の同時利用はできません。
- PARTITION BY句に、以下の式は指定できません。
  - OVER句を含む式
  - 集計関数を含む式
  - 列の別名を含む式
  - サブクエリ
- ORDER BY句に、以下の式は指定できません。
  - OVER句を含む式
  - 集計関数を含む式
  - 列の別名を含む式
  - サブクエリ


### GROUP BY RANGE

一定の時間幅毎に結果を分割した結果集合を作成し、各結果集合の値について集計演算を行います。また、補間演算の指定がある場合、結果に値が含まれない集合について値の補間演算を行います。

**構文**

|                                   |
|-----------------------------------|
| GROUP BY RANGE(*日時カラム名*) EVERY( *時間間隔*, *単位* \[ ,*オフセット* \] ) \[ FILL(*補間方法*) \]    |

**仕様**

GROUP BY RANGE句を含むSELECT文は、以下の制約があります。

- SELECT句の*式リスト*にはカラム式や関数などを指定できますが、分析関数やサブクエリを用いることはできません。
- WHERE句で指定する探索条件には、必ず*日時カラム名*に関する日時範囲の条件を指定する必要があります。具体的には以下のいずれかの条件が必須です。いずれも定数値との比較でなければなりません。
  + *日時カラム名*との大小比較条件による範囲条件
  + *日時カラム名*を用いたBETWEEN式による範囲条件

GROUP BY RANGE句の仕様は以下となります。

- 集約や補間を行う結果集合の始点時刻は*日時カラム名*のカラムの値として計算されます。このカラムに指定できるのはTIMESTAMP型(精度指定付きTIMESTAMP型を含む)のカラムのみです。
- 結果集合の始点時刻は、「先頭時刻」と「時間の間隔」を基準に求められます。具体的には、「「先頭時刻」+「時間の間隔」のN倍」の日時が各結果集合の始点時刻に用いられます。
- 「先頭時刻」と「時間の間隔」はそれぞれ以下のように設定されます。
  - 「先頭時刻」は*オフセット*で指定します。*オフセット*を明示的に指定する方法として、以下の２種類の方法があります。  
    * 整数値(定数値)を指定する方法：整数値(LONG)で指定します。値の単位には*単位*を用います。
    * 文字列でタイムゾーンを指定する方法：タイムゾーン(Z|±HH:MM|±HHMM)の書式で指定します。
  - *オフセット*の指定がない場合、DB接続時に決定されたタイムゾーンのオフセットが*オフセット*値として用いられます。DB接続時のタイムゾーンの決定については、『[GridDB JDBCドライバ説明書](../md_reference_jdbc/md_reference_jdbc.md)』等を参照してください。 
  - 結果集合の始点時刻を計算するときに用いる「時間の間隔」は、*時間間隔*と*単位*で指定されます。以下のように設定できます。
    + *時間間隔*は、正の整数値を指定できます。定数値のみ指定可能です。
    + *単位*には、以下のいずれかを指定できます。
      * DAY | HOUR | MINUTE | SECOND | MILLISECOND
- 得られた結果集合内にひとつも値が含まれない場合、その区間の値を補間演算により求めることができます。値の求め方(補間演算の方法)は*補間方法*で指定します。
  + *補間方法*には、以下のいずれかを指定できます。
    * LINEAR　：直近の前後の時刻のカラム値を用いて線形補間して出力します。
      + ただし、数値演算できない型のカラムの場合は、以下のPREVIOUS演算と同じ補間を行います。
    * NONE　　：ロウ全体を出力しません。
    * NULL　　：カラム値をNULLとします。
    * PREVIOUS：直近の前の時刻のカラム値を出力します。

例)
```example
名前: trend_data1

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00       10
  2023-01-01T00:00:10       30
  2023-01-01T00:00:20       30
  2023-01-01T00:00:30       50
  2023-01-01T00:00:40       50
  2023-01-01T00:00:50       70


○集計演算
SELECT ts,avg(value) FROM trend_data1
  WHERE ts BETWEEN TIMESTAMP('2023-01-01T00:00:00Z') AND TIMESTAMP('2023-01-01T00:01:00Z')
  GROUP BY RANGE(ts) EVERY (20,SECOND)

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00       20
  2023-01-01T00:00:20       40
  2023-01-01T00:00:40       60
```

```example
名前: trend_data2

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00        5
  2023-01-01T00:00:10       10
  2023-01-01T00:00:20       15
  　(データ欠損時刻)
  2023-01-01T00:00:40       25
  

  ○補間演算
SELECT * FROM trend_data2
  WHERE ts BETWEEN TIMESTAMP('2023-01-01T00:00:00Z') AND TIMESTAMP('2023-01-01T00:01:00Z')
  GROUP BY RANGE(ts) EVERY (10,SECOND) FILL (LINEAR)

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00        5
  2023-01-01T00:00:10       10
  2023-01-01T00:00:20       15
  2023-01-01T00:00:30       20
  2023-01-01T00:00:40       25

```


## 演算子

SQL文で使用する演算子を以下に説明します。

### 演算子一覧

演算子の一覧は次の通りです。

| 分類 | 演算子 | 説明 |
|------|-------|------|
| 算術  | +     | 加算します  |
|       | -     | 減算します  |
|       | *     | 乗算します  |
|       | /     | 除算します  |
|       | %      | 剰余を求めます |
| 文字  | \|\|   | 任意の型の値を文字列として連結します。<br>いずれかの値がNULLの場合はNULLを返します。 |
| 比較  | =, ==    | 等しいかどうかを比較します |
|       | !=, \<\> | 等しくないかどうかを比較します |
|       | >      | より大きいかどうかを比較します |
|       | >=     | より大きい、または、等しいかどうかを比較します |
|       | <      | より小さいかどうかを比較します |
|       | <=     | より小さい、または、等しいかどうかを比較します |
|       | IS     | 等しいかどうかを比較します。<br>両方の式がNULLの場合はtrueを返します。<br>いずれかがNULLの場合はfalseを返します。 |
|       | IS NOT | 等しくないかどうかを比較します。<br>両方の式がNULLの場合はfalseを返します。<br>いずれかがNULLの場合はtrueを返します。 |
|       | ISNULL | 左辺の式がNULLかを判定します |
|       | NOTNULL | 左辺の式がNULLでないかを判定します |
|       | [LIKE](#op_like)    | 文字列を検索します。 |
|       | [GLOB](#op_glob)    | 文字列を検索します。 |
|       | [BETWEEN](#op_between) | 指定した範囲の値を取り出します。|
|       | [IN](#op_in) | 値の集合の中に指定した値が含まれるかどうかを返します。 |
| ビット | &      | A & B ：AとBのビットのANDをとります |
|       | \|     | A \| B ：AとBのビットのORをとります |
|       | ~      | ~A ：AのビットのNOTをとります |
|       | \<\<   | A \<\< B ：Aを左へBビット分シフトします |
|       | \>\>   | A \>\> B ：Aを右へBビット分シフトします |
| 論理  | AND    | 両方の式がtrueの場合はtrueを返します。<br>いずれかがfalseの場合はfalseを返します。<br>それ以外の場合はNULLを返します。 |
|       | OR     | いずれかの式がtrueの場合はtrueを返します。<br>両方がfalseの場合はfalseを返します。<br>それ以外の場合はNULLを返します。 |
|       | NOT    | 式がtrueの場合はfalseを返します。<br>falseの場合はtrueを返します。<br>それ以外の場合はNULLを返します。 |

<a id="op_like"></a>
### LIKE

文字列を検索します。

**構文**

|                                   |
|-----------------------------------|
| *str* \[NOT\] LIKE *pattern_str* \[ESCAPE *escape_str* \] |

**仕様**

- [LIKE関数](#like-1)を参照してください。

<a id="op_glob"></a>
### GLOB

**構文**

文字列を検索します。

|      |
|-------|
| *str* GLOB *pattern_str* |

**仕様**

- [GLOB関数](#glob-1)を参照してください。

<a id="op_between"></a>
### BETWEEN

指定した範囲の値を取り出します。

**構文**

|                                   |
|-----------------------------------|
| *式1* \[NOT\] BETWEEN *式2* AND *式3* |

**仕様**

- 次の条件を満たす場合はtrueを返します。

  ``` example
  式2 ≦ 式1 ≦ 式3
  ```

- NOTを指定すると上記の条件を満たさない場合にtrueを返します。

<a id="op_in"></a>
### IN

値の集合の中に指定した値が含まれるかどうかを返します。

**構文**

|                                   |
|-----------------------------------|
| *式1* \[NOT\] IN ( [*式2* \[, *式3* ...\]] ) |

**仕様**

- *式1*の値が、*式N*の結果に含まれる場合はtrueを返します。
- INは[サブクエリ](#in_sub_query)に対しても使用できます。


<a id="label_function"></a>
## 関数

SQL文で使用する関数を以下に説明します。

### 関数一覧

SQL文には以下の関数が用意されています。

| 分類 | 関数名           | 説明 |
|------|-----------------|------|
| [集計](#aggregation) | [AVG](#avg)   | 平均値を返します    |
|      | [COUNT](#count)               | ロウ数を返します     |
|      | [MAX](#Aggregate_MAX)         | 最大値を返します     |
|      | [MIN](#Aggregate_MIN)         | 最小値を返します     |
|      | [SUM](#sumtotal)              | 合計値を返します     |
|      | [TOTAL](#sumtotal)            | 合計値を返します     |
|      | [GROUP_CONCAT](#group_concat) | 値を連結します     |
|      | [STDDEV_SAMP](#stddev_samp)   | 標本標準偏差を返します     |
|      | [STDDEV](#stddevstddev0)      | 標本標準偏差を返します     |
|      | [STDDEV0](#stddevstddev0)     | 標本標準偏差を返します     |
|      | [STDDEV_POP](#stddev_pop)     | 母標準偏差を返します     |
|      | [VAR_SAMP](#var_samp)    | 標本分散を返します     |
|      | [VARIANCE](#variancevariance0)  | 標本分散を返します     |
|      | [VARIANCE0](#variancevariance0) | 標本分散を返します     |
|      | [VAR_POP](#var_pop)      | 母分散を返します     |
|      | [MEDIAN](#MEDIAN)             | 中央値を返します     |
|      | [PERCENTILE_CONT](#PERCENTILE_CONT)  | パーセンタイル値を返します     |
| [算術](#算術関数) | [ABS](#abs)       | 絶対値を返します     |
|      | [ROUND](#round)               | 四捨五入します     |
|      | [RANDOM](#random)             | 乱数を返します     |
|      | [MAX](#Arithmetic_MAX)        | 最大値を返します     |
|      | [MIN](#Arithmetic_MIN)        | 最小値を返します     |
|      | [LOG](#Arithmetic_LOG)        | 対数を返します     |
|      | [SQRT](#Arithmetic_SQRT)      | 平方根を返します |
|      | [TRUNC](#Arithmetic_TRUNC)    | 数値を切り捨てます |
|      | [HEX_TO_DEC](#Arithmetic_HEX_TO_DEC)    | 16進数の文字列を10進数の数値に変換します |
| [文字](#文字関数) | [LENGTH](#length) | 文字列の長さを返します     |
|      | [LOWER](#lower)               | 文字列を小文字に変換します     |
|      | [UPPER](#upper)               | 文字列を大文字に変換します     |
|      | [SUBSTR](#substr)             | 文字列の一部を切り出します     |
|      | [REPLACE](#replace)           | 文字列を置換します     |
|      | [INSTR](#instr)               | 文字列の中から特定の文字列の位置を返します     |
|      | [LIKE](#like-1)                 | 文字列を検索します     |
|      | [GLOB](#glob-1)                 | 文字列を検索します     |
|      | [TRIM](#trim)                 | 文字列の両端から特定の文字を除きます     |
|      | [LTRIM](#ltrim)               | 文字列の左端から特定の文字を除きます     |
|      | [RTRIM](#rtrim)               | 文字列の右端から特定の文字を除きます     |
|      | [QUOTE](#quote)               | 文字列をシングルクォートで囲みます     |
|      | [UNICODE](#unicode)           | 文字のUnicodeコードポイントを返します     |
|      | [CHAR](#char)                 | Unicodeコードポイントを文字に変換して連結します     |
|      | [PRINTF](#printf)             | フォーマット変換した文字列を返します     |
|      | [TRANSLATE](#translate)       | 文字列を置換します     |
| [日時](#time_function) | [NOW](#now)       | 現在時刻を返します       |
|      | [TIMESTAMP](#timestamp)             | 時刻の文字列表記をミリ秒精度のTIMESTAMP型に変換します     |
|      | [TIMESTAMP_MS](#timestamp_ms)             | 時刻の文字列表記をミリ秒精度のTIMESTAMP型(TIMESTAMP(3))に変換します     |
|      | [TIMESTAMP_US](#timestamp_us)             | 時刻の文字列表記をマイクロ秒精度のTIMESTAMP型(TIMESTAMP(6))に変換します     |
|      | [TIMESTAMP_NS](#timestamp_ns)             | 時刻の文字列表記をナノ秒精度のTIMESTAMP型(TIMESTAMP(9))に変換します     |
|      | [TIMESTAMP_ADD](#timestamp_add)       | 時刻を加算します        |
|      | [TIMESTAMP_DIFF](#timestamp_diff)     | 時刻の差分を返します     |
|      | [TO_TIMESTAMP_MS](#to_timestamp_ms) | 時刻'1970-01-01T00:00:00.000Z'に経過時間を加算します    |
|      | [TO_EPOCH_MS](#to_epoch_ms)         | 時刻'1970-01-01T00:00:00.000Z'からの経過時間を返します     |
|      | [EXTRACT](#extract)                 | 時刻から特定のフィールドの値を取り出します    |
|      | [STRFTIME](#strftime)               | 時刻をフォーマット変換した文字列を返します    |
|      | [MAKE_TIMESTAMP](#make_timestamp)   | 時刻を生成します    |
|      | [TIMESTAMP_TRUNC](#timestamp_trunc) | 時刻を切り捨てます    |
| [WINDOW](#window_function) | [ROW_NUMBER](#row_number)       | 結果のロウに対して、一意となる連番値を割り振ります       |
| [その他](#other_function) | [COALESCE](#coalesce)      | NULLではない最初の引数を返します     |
|      | [IFNULL](#ifnull)          | NULLではない最初の引数を返します     |
|      | [NULLIF](#nullif)          | 2つの引数が同じ場合はNULL、異なる場合は最初の引数を返します     |
|      | [RANDOMBLOB](#randomblob)  | BLOB型の値(乱数)を返します     |
|      | [ZEROBLOB](#zeroblob)      | BLOB型の値(0x00)を返します     |
|      | [HEX](#hex)                | BLOB型の値を16進表記に変換します     |
|      | [TYPEOF](#typeof)          | 値のデータ型を返します     |


関数の説明では、以下のテーブルのデータを実行例として使用します。

```example
テーブル: employees

 id   first_name   last_name   age     department    enrollment_period
----+------------+-----------+-------+-------------+-------------------
  0   John         Smith       43      Sales         15.5
  1   William      Jones       59      Development   23.2
  2   Richard      Brown       (NULL)  Sales          7.0
  3   Mary         Taylor      31      Research      (NULL)
  4   Lisa         (NULL)      29      (NULL)         4.9
  5   James        Smith       43      Development   10.3

テーブル: departments

 id   department   
----+------------
  0   Sales
  1   Development
  2   Research

テーブル: travelexpenses

 id    date         empId   amount
-----+------------+-------+--------
  101  2020/02/01   0       200
  102  2020/02/03   2       2500
  103  2020/02/03   3       60
  104  2020/02/04   0       200
  105  2020/02/05   0       150
  106  2020/02/06   3       80
```

[メモ]
- NULLの値は(NULL)と表記します。


<a id="aggregation"></a>
### 集計関数

値を集計する関数です。
集計関数の引数には、DISTINCTまたはALLを指定できます。

|      |      |
|------|-------|
| 書式 | function( [DISTINCT \| ALL] *argument*) |

| 項目     | 意味  |
|---------|-------|
| DISTINCT | 重複する値のロウは除外して集計します |
| ALL     | 重複する値も含めてすべてのロウを集計します |

指定を省略した場合は、ALLを指定した場合と同じになります。

[メモ]
- 集計関数は、SELECT句にしか使えません。
- 計算の対象となる行が存在しない場合、COUNTの結果は0になります。その他の集計関数の結果はNULLになります。


また、集計関数は、分析関数としてOVER句と共に利用可能です。詳細はOVER句を参照ください。   

例)集計関数SUMとOVER句を利用した例
```example
SELECT id, date, empId, amount, SUM(amount) OVER(PARTITION BY empID ORDER BY id) as accumulated FROM travelexpenses;
結果：
 id    date         empId   amount   accumulated
-----+------------+-------+--------+-------------
  101  2020/02/01   0       200      200
  104  2020/02/04   0       200      400
  105  2020/02/05   0       150      550
  102  2020/02/03   2       2500     2500
  103  2020/02/03   3       60       60
  106  2020/02/06   3       80       140
```
【注意事項】
- GROUP_CONCATとMEDIANは、OVER句と共に利用できません。



#### AVG

|      |      |
|------|-------|
| 書式 | AVG( [DISTINCT \| ALL] *n*) |

*n*の平均値を返します。

- 引数*n*には、数値型の値を指定します。
- *n*の値がNULLのロウは、計算の対象外になります。
- 結果の型はDOUBLE型です。

例)
```example
SELECT AVG(age) FROM employees;
結果：41.0

SELECT AVG(DISTINCT age) FROM employees;
結果：40.5

SELECT department, AVG(age) avg FROM employees GROUP BY department;
結果：
  department   avg
  ------------+-----
  Development  51.0
  Research     31.0
  Sales        43.0
  (NULL)       29.0
  ```



#### COUNT

|      |      |
|------|-------|
| 書式 | COUNT(* \| [DISTINCT \| ALL] *x*) |

ロウの数を返します。

- *x*の値がNULLのロウは、計算の対象外になります。ロウ数にはカウントされません。
- 結果の型はLONG型です。

例)
```example
SELECT COUNT(*) FROM employees;
結果：6

// 値がNULLのロウは無視してカウントします
SELECT COUNT(department) FROM employees;
結果：5

SELECT COUNT(DISTINCT department) FROM employees;
結果：3
```

<a id="Aggregate_MAX"></a>
#### MAX

|      |      |
|------|-------|
| 書式 | MAX( [DISTINCT \| ALL] *x*) |

最大値を返します。

- 引数*x*には、任意の型の値を指定します。
  - 文字列型の場合は、先頭文字から順に比較して文字コードが最大である文字列を返します。
  - TIMESTAMP型の場合は、最も新しい日時を返します。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型は、引数*x*の型と同じです。

例)
```example
SELECT MAX(age) FROM employees;
結果：59

SELECT MAX(first_name) FROM employees;
結果：William
```


<a id="Aggregate_MIN"></a>
#### MIN

|      |      |
|------|-------|
| 書式 | MIN( [DISTINCT \| ALL] *x*) |

最小値を返します。

- 引数*x*には、任意の型の値を指定します。
  - 文字列型の場合は、先頭文字から順に比較して文字コードが最小である文字列を返します。
  - TIMESTAMP型の場合は、最も古い日時を返します。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型は、引数*x*の型と同じです。


例)
```example
SELECT MIN(age) FROM employees;
結果：29

SELECT MIN(first_name) FROM employees;
結果：James
```



#### SUM/TOTAL

|      |      |
|------|-------|
| 書式 | SUM( [DISTINCT \| ALL] *n*) |
| 書式 | TOTAL( [DISTINCT \| ALL] *n*) |

合計値を返します。

- 引数*n*には、数値型の値を指定します。
- *n*の値がNULLのロウは、計算の対象外になります。

- SUMとTOTALの違いは以下の通りです。
  - *n*が整数型の値のみの場合、SUMは整数(LONG型)で値を返します。TOTALは浮動小数点数(DOUBLE型)で値を返します。
  - *n*に浮動小数点数型の値が含まれる場合は、両方とも浮動小数点数(DOUBLE型)で値を返します。
  - *n*の値がNULLのみの場合、SUMはNULLを返します。TOTALは0を返します。

例)
```example
SELECT SUM(age) FROM employees;
結果：205

SELECT TOTAL(age) FROM employees;
結果：205.0

SELECT department, SUM(age) sum FROM employees GROUP BY department;
結果：
  department   sum
  ------------+-----
  Development  102
  Research      31
  Sales         43
  (NULL)        29
```



#### GROUP_CONCAT

|      |      |
|------|-------|
| 書式 | GROUP_CONCAT( [DISTINCT \| ALL] *x* [, *separator*] ) |

*x*の値を連結した文字列を返します。
*separator*は、連結するセパレータを指定します。指定しない場合は","で連結します。

- 引数*x*には、任意の型の値を指定します。
  - TIMESTAMP型(精度指定付きTIMESTAMP型)の場合は、'YYYY-MM-DDThh:mm:ss.SSS(Z|±hh:mm)'形式の時刻の文字列表記に変換して連結します。時刻の小数部の桁数は引数のTIMESTAMP型の精度に応じて決まります。詳細は、[TIMESTAMP_MS関数](#timestamp_ms)など、対応する精度の関数の説明を参照してください。
  - *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型はSTRING型です。

例)
```example
// 名前last_nameを'/'で連結します
SELECT GROUP_CONCAT(last_name, '/') from employees;
結果： Smith/Jones/Brown/Taylor/Smith

// 部署departmentごとに、名前first_nameを連結します
SELECT department, GROUP_CONCAT(first_name) group_concat from employees GROUP BY(department);
結果：
   department    group_concat
  -------------+--------------
   Development  William,James
   Research     Mary
   Sales        John,Richard
   (NULL)       Lisa

SELECT GROUP_CONCAT(age, ' + ') FROM employees;
結果：43 + 59 + 31 + 29 + 43
```



#### STDDEV_SAMP

|      |      |
|------|-------|
| 書式 | STDDEV_SAMP( [DISTINCT \| ALL] *x*) |

標本標準偏差を返します。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- *x*が1件の場合は、NULLを返します。
- 結果の型はDOUBLE型です。

例)
```example
SELECT department, STDDEV_SAMP(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
結果：
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       (NULL)

```



#### STDDEV/STDDEV0

|      |      |
|------|-------|
| 書式 | STDDEV( [DISTINCT \| ALL] *x*) |
| 書式 | STDDEV0( [DISTINCT \| ALL] *x*) |

標本標準偏差を返します。STDDEVはSTDDEV_SAMP関数の別名です。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型はDOUBLE型です。
- STDDEVとSTDDEV0の違いは以下の通りです。
  - STDDEVは、*x*が1件の場合に、NULLを返します。
  - STDDEV0は、*x*が1件の場合に、0を返します。

例)
```example
SELECT department, STDDEV(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
結果：
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       (NULL)

SELECT department, STDDEV0(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
結果：
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       0.0

SELECT STDDEV(enrollment_period) enrollment_period_stddev from employees WHERE age >= 55;
結果：
   enrollment_period_stddev
  --------------------------
   (NULL)

SELECT STDDEV0(enrollment_period) enrollment_period_stddev from employees WHERE age >= 55;
結果：
   enrollment_period_stddev
  --------------------------
   0.0
```



#### STDDEV_POP

|      |      |
|------|-------|
| 書式 | STDDEV_POP( [DISTINCT \| ALL] *x*) |

母標準偏差を返します。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型はDOUBLE型です。

例)
```example
SELECT department, STDDEV_POP(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
結果：
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  6.450000000000002
   Research     (NULL)
   Sales        4.25
   (NULL)       0.0

```



#### VAR_SAMP

|      |      |
|------|-------|
| 書式 | VAR_SAMP( [DISTINCT \| ALL] *x*) |

標本分散を返します。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- *x*が1件の場合は、NULLを返します。
- 結果の型はDOUBLE型です。

例)
```example
SELECT department, VAR_SAMP(enrollment_period) enrollment_period_variance from employees GROUP BY department;
結果：
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       (NULL)

```



#### VARIANCE/VARIANCE0

|      |      |
|------|-------|
| 書式 | VARIANCE( [DISTINCT \| ALL] *x*) |
| 書式 | VARIANCE0( [DISTINCT \| ALL] *x*) |

標本分散を返します。VARIANCEはVAR_SAMP関数の別名です。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型はDOUBLE型です。
- VARIANCEとVARIANCE0の違いは以下の通りです。
  - VARIANCEは*x*が1件の場合に、NULLを返します。
  - VARIANCE0は*x*が1件の場合に、0を返します。

例)
```example
SELECT department, VARIANCE(enrollment_period) enrollment_period_variance from employees GROUP BY department;
結果：
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       (NULL)

SELECT department, VARIANCE0(enrollment_period) enrollment_period_variance from employees GROUP BY department;
結果：
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       0.0

SELECT VARIANCE(enrollment_period) enrollment_period_variance from employees WHERE age >= 55;
結果：
   enrollment_period_variance
  ----------------------------
   (NULL)

SELECT VARIANCE0(enrollment_period) enrollment_period_variance from employees WHERE age >= 55;
結果：
   enrollment_period_variance
  ----------------------------
   0.0
```


#### VAR_POP

|      |      |
|------|-------|
| 書式 | VAR_POP( [DISTINCT \| ALL] *x*) |

母分散を返します。

- 引数*x*には、数値型の値を指定します。
  - 式に集計関数、WINDOW関数/OVER句を含めることはできません。
- *x*の値がNULLのロウは、計算の対象外になります。
- 結果の型はDOUBLE型です。

例)
```example
SELECT department, VAR_POP(enrollment_period) enrollment_period_variance from employees GROUP BY department;
結果：
   department    enrollment_period_variance
  -------------+----------------------------
   Development  41.60250000000002
   Research     (NULL)
   Sales        18.0625
   (NULL)       0.0

```



#### MEDIAN

|      |      |
|------|-------|
| 書式 | MEDIAN(*n*) |

*n*の中央値を返します。計算対象のロウ数が偶数の場合は、中央に近い2つのロウの平均値を返します。

- 引数*n*には、数値型の値を指定します。
  - サブクエリは指定できません。
- *n*の値がNULLのロウは、計算の対象外になります。
- 結果の型は、*n*が整数のみの場合はLONG型、浮動小数点数の場合はDOUBLE型です。
- 同一SELECT句内でのWINDOW関数/OVER句の複数利用や、WINDOW関数/OVER句とMEDIAN関数の同時利用はできません。

例)
```example
SELECT MEDIAN(age) FROM employees;
結果：43

SELECT department, MEDIAN(age) mn FROM employees GROUP BY department ORDER BY mn DESC;
結果：
  department   mn
  ------------+-----
  Development  51
  Sales        43
  Research     31
  (NULL)       29
```


#### PERCENTILE_CONT

|      |      |
|------|-------|
| 書式 | PERCENTILE_CONT(*percentile*) WITHIN GROUP ( ORDER BY *sort_key* ) |

*sort_key*で指定されたソート順序での連続分布モデルに基づく、*percentile*で指定のパーセンタイルに対応する値を求めます。

- *percentile*には、0以上1以下のDOUBLE定数を指定できます。
- *sort_key*に存在するNULL値は、集計対象外となります。
- 集計対象が1つも存在しない場合のみ、NULLが求められます。
- 同一SELECT句内でのWINDOW関数/OVER句、MEDIAN関数、他のPERCENTILE_CONTとは同時には利用はできません。

例)
```example
SELECT PERCENTILE_CONT(0.25) WITHIN GROUP( ORDER BY age ) FROM employees;
結果：18

```


### 算術関数

#### ABS

|      |      |
|------|-------|
| 書式 | ABS(*n*) |

*n*の絶対値を返します。正の数はそのままの値、負の数は-1を掛けた値を返します。

- 引数*n*には、数値型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 値が-2<sup>63</sup>の整数の場合は、オーバフローエラーになります。
- 結果の型は、*n*が整数のみの場合はLONG型、浮動小数点数の場合はDOUBLE型です。

例)
```example
SELECT first_name, ABS(age) abs FROM employees;
結果：
  first_name    abs
  ------------+-------
  John          43
  William       59
  Richard       (NULL)
  Mary          31
  Lisa          29
  James         43
```



#### ROUND

|      |      |
|------|-------|
| 書式 | ROUND(*n* [, *m*]) |

四捨五入します。*n*の値を、小数点以下*m*桁で四捨五入した値を返します。

- 引数*n*には、数値型のカラムを指定します。
- 引数*m*には、0以上の整数を指定します。省略した場合、*m*は0になります。
- 値がNULLの場合は、NULLを返します。
- 結果の型は、*n*が整数のみの場合はLONG型、浮動小数点数の場合はDOUBLE型です。

例)
```example
SELECT first_name, ROUND(enrollment_period, 0) round FROM employees;
結果：
  first_name    round
  ------------+-------
  John          16.0
  William       23.0
  Richard        7.0
  Mary          (NULL)
  Lisa           5.0
  James         10.0
```



#### RANDOM

|      |      |
|------|-------|
| 書式 | RANDOM() |

乱数を返します。乱数は、-2<sup>63</sup>から2<sup>63</sup>-1までの範囲の整数です。

- 結果の型はLONG型です。

例)
```example
SELECT first_name, RANDOM() random FROM employees;
結果：
  first_name    random
  ------------+----------------------
  John          -3382931580741820003
  William       -7362300487836647182
  Richard        8834368641333737477
  Mary          -8544493602797564288
  Lisa          -7727163797274657674
  James          6751560427268247384
```


<a id="Arithmetic_MAX"></a>
<a id="Arithmetic_MIN"></a>
#### MAX/MIN

|      |      |
|------|-------|
| 書式 | MAX(*x1*, *x2* [,...]) |

値*xN*の中で、最大の値を返します。

|      |      |
|------|-------|
| 書式 | MIN(*x1*, *x2* [,...]) |

値*xN*の中で、最小の値を返します。


例)
```example
SELECT first_name, age, enrollment_period, MAX(age, enrollment_period) max FROM employees;
結果：
  first_name    age    enrollment_period   max
  ------------+-------+------------------+--------
  John          43      15.5               43.0
  William       59      23.2               59.0
  Richard       (NULL)   7.0               (NULL)
  Mary          31      (NULL)             (NULL)
  Lisa          29       4.9               29.0
  James         43      10.3               43.0
```

<a id="Arithmetic_LOG"></a>
#### LOG

|     |               |
| --- | ------------- |
| 書式  | LOG(*n*, *m*) |

*n*を底とした*m*の対数を返します。

- 引数*n*には、0より大きく1以外の数値型の値を指定します。
- 引数*m*には、0より大きい数値型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型は、DOUBLE型です。

例)

```example
SELECT LOG(2, 8);
結果：3.0

SELECT LOG(0.5, 2.0);
結果：-1.0
```

<a id="Arithmetic_SQRT"></a>
#### SQRT

|     |           |
| --- | --------- |
| 書式  | SQRT(*n*) |

*n*の正の平方根を返します。

- 引数*n*には、0以上の数値型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型はDOUBLE型です。

例)

```example
SELECT SQRT(4);
結果：2.0

SELECT SQRT(16.0);
結果：4.0
```

<a id="Arithmetic_TRUNC"></a>
#### TRUNC

|     |                                                    |
| --- | -------------------------------------------------- |
| 書式  | TRUNC(*n* [,*m*]) |

*m>=0*の場合、*n*の値の小数点*m*桁未満を切り捨てた値を返します。

*m<0*の場合、*n*の値の整数-*m*桁以下を切り捨てた値を返します。

- 引数*n*には、数値型の値を指定します。
- 引数*m*には、整数を指定します。省略した場合、*m*は0になります。309以上、もしくは-308以下の値を与えることはできません。
- 値がNULLの場合は、NULLを返します。
- 結果の型は、引数nに整数が指定された場合はLONG型、小数が指定された場合はDOUBLE型です。

例)

```example
SELECT TRUNC(123.4567);
結果：123.0

SELECT TRUNC(123.4567, 2);
結果：123.45

SELECT TRUNC(123.4567, -1);
結果：120.0

SELECT TRUNC(123.4567, -3);
結果：0.0

SELECT TRUNC(1234567, -2);
結果：1234500
```

<a id="Arithmetic_HEX_TO_DEC"></a>
#### HEX_TO_DEC

|     |               |
| --- | ------------- |
| 書式  | HEX_TO_DEC(*str*) |

16進数文字列*str*を10進数の数値型に変換します。

- 引数*str*には、16進数変換できる文字列型の値(0-9, a-f, A-F)を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型はLONG型です。

例)
```example
SELECT HEX_TO_DEC('FF');
結果：255

SELECT HEX_TO_DEC('10');
結果：16
```

### 文字関数


#### LENGTH

|      |      |
|------|-------|
| 書式 | LENGTH(*str*) |

文字列*str*の長さを返します。

- 引数*str*には、文字列型の値を指定します。
  - 文字列型はUnicodeコードポイントを文字とします。
- 値がNULLの場合は、NULLを返します。
- 結果の型はLONG型です。
- 引数にはBLOB型を指定することも可能です。

例)
```example
SELECT last_name, LENGTH(last_name) length FROM employees;
結果：
  last_name     length
  ------------+----------------------
  Smith         5
  Jones         5
  Brown         5
  Taylor        6
  (NULL)        (NULL)
  Smith         5
```



#### LOWER

|      |      |
|------|-------|
| 書式 | LOWER(*str*) |

文字列*str*のアルファベットをすべて小文字に変換します。

- 引数*str*には、文字列型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型は文字列型です。
- ASCII英字相当以外のUnicode文字は変換対象外です。

例)
```example
SELECT last_name, LOWER(last_name) lower FROM employees;
結果：
  last_name     lower
  ------------+----------------------
  Smith         smith
  Jones         jones
  Brown         brown
  Taylor        taylor
  (NULL)        (NULL)
  Smith         smith
```



#### UPPER

|      |      |
|------|-------|
| 書式 | UPPER(*str*) |

文字列*str*のアルファベットをすべて大文字に変換します。

- 引数*str*には、文字列型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型は文字列型です。
- ASCII英字相当以外のUnicode文字(キリル文字など)は変換対象外です。

例)
```example
SELECT last_name, UPPER(last_name) upper FROM employees;
結果：
  last_name     upper
  ------------+----------------------
  Smith         SMITH
  Jones         JONES
  Brown         BROWN
  Taylor        TAYLOR
  (NULL)        (NULL)
  Smith         SMITH
```



#### SUBSTR

|      |      |
|------|-------|
| 書式 | SUBSTR(*str*, *index* [, *length*]) |

文字列を部分的に切り出します。文字列*str*の開始位置*index*の文字から、長さ*length*の文字列を切り出して返します。

- 引数*str*には、文字列型の値を指定します。
- 引数*index*には、1からの整数を指定します。文字列先頭の開始位置は1です。
- 引数*length*を省略した場合は、文字列*str*の最後尾までを切り出します。
- *str*の値がNULLの場合は、NULLを返します。
- 結果の型は文字列型です。
- 引数にはBLOB型を指定することも可能です。

例)
```example
SELECT SUBSTR('abcdefg', 3);
結果：cdefg

SELECT SUBSTR('abcdefg', 3, 2);
結果：cd
```



#### REPLACE

|      |      |
|------|-------|
| 書式 | REPLACE(*str*, *search_str*, *replacement_str*) |

文字列を置換します。
文字列*str*の中で、文字列*search_str*に一致する部分をすべて*replacement_str*に置き換えます。

- 引数*str*、*search_str*、*replacement_str*には、文字列型の値を指定します。
- *str*の値がNULLの場合は、NULLを返します。
- 結果の型は文字列型です。

例)
```example
SELECT REPLACE('abcdefabc', 'abc', '123');
結果：123def123
```



#### INSTR

|      |      |
|------|-------|
| 書式 | INSTR(*str*, *search_str* \[, offset\] \[, occurrence\]) |

文字列*str*の中から文字列*search_str*を探し、その開始位置を返します。見つからなかった場合は0を返します。

- 引数*str*、*search_str*には、文字列型もしくはBLOB型の値を指定します。*str*と*search_str*には同じデータ型を指定する必要があります。引数*offset*、*occurrence*には、LONG型の値を指定します。
- 文字列型の場合はUnicodeのコードポイント単位、BLOB型の場合はバイト単位で計算します。
- *offset*は検索開始位置を表し、正の場合は前方先頭から、負の場合は後方末尾から順に計算します。0の場合は一致なしとみなし、0を返します。
- *occurrence*は出現回数を表し、指定の回数だけ繰り返した最後のマッチ位置を計算します。0の場合は一致なしとみなし、0を返します。
- いずれかの引数の値がNULLの場合は、NULLを返します。
- 結果の型はLONG型です。


例)
```example
SELECT INSTR('abcdef', 'cd');
結果：3

SELECT INSTR('abcdef', 'gh');
結果：0

SELECT INSTR('abcabcabcde', 'ab', 2, 2);
結果：7

SELECT INSTR('abcabcabcde', 'ab', -1, 2);
結果：4

```



#### LIKE

|      |      |
|------|-------|
| 書式 | LIKE(*pattern_str*, *str* [, *escape_str*]) |

文字列を検索します。
文字列*str*が照合パターン*pattern_str*と一致する場合はtrueを返します。一致しない場合はfalseを返します。
照合パターンには次の2つのワイルドカードが使用できます。

| ワイルドカード | 意味 |
|------|-----|
| _     | 任意の1文字 |
| %     | 任意の0文字以上の文字列 |

ワイルドカードの文字_または%を含む*str*に対して、文字_または%を検索する場合には、エスケープ文字*escape_str*を指定します。
ワイルドカードの文字の前にエスケープ文字を指定すると、ワイルドカードと解釈されなくなります。

- 引数*str*、*pattern_str*、*escape_str*には、文字列型の値を指定します。
- いずれかの引数の値がNULLの場合は、NULLを返します。
- 大文字小文字は区別しません。
- 結果の型はBOOL型です。

例)
```example
SELECT last_name, LIKE('%mi%', last_name) like_name FROM employees;
結果：
  last_name     like_name
  ------------+----------------------
  Smith         true
  Jones         false
  Brown         false
  Taylor        false
  (NULL)        (NULL)
  Smith         true


SELECT LIKE('%C%E%',  'ABC%DEF');
結果：true

SELECT LIKE('%C@%E%', 'ABC%DEF', '@');
結果：false

SELECT LIKE('%C@%D%', 'ABC%DEF', '@');
結果：true
```



#### GLOB

|      |      |
|------|-------|
| 書式 | GLOB(*pattern_str*, *str*) |

文字列を検索します。
文字列*str*が照合パターン*pattern_str*と一致する場合はtrueを返します。一致しない場合はfalseを返します。
照合パターンにはワイルドカードが使用できます。

| ワイルドカード | 意味 |
|------|-----|
| ?     | 任意の1文字 |
| *     | 任意の0文字以上の文字列 |
| [abc] | 文字a、bまたはcのいずれかに一致 |
| [a-e] | 文字aからeまでのいずれかに一致 |

- 引数*str*、*pattern_str*には、文字列型の値を指定します。
- いずれかの引数の値がNULLの場合は、NULLを返します。
- 大文字小文字は区別します。
- 結果の型はBOOL型です。

例)
```example
SELECT GLOB('*[BA]AB?D', 'AABCD');
結果：true
```



#### TRIM

|      |      |
|------|-------|
| 書式 | TRIM(*str* [, *trim_str*]) |

文字列*str*の両端から、文字列*trim_str*のすべての文字を削除します。

- 引数*str*, *trim_str*には、文字列型の値を指定します。
- 引数*trim_str*の文字列に含まれるすべての文字を削除します。省略すると、*str*の両端からスペースを削除します。
- 結果の型は文字列型です。

例)
```example
SELECT TRIM(' ABC ');
結果：ABC  (両端にスペース無し)

SELECT TRIM('ABCAA', 'BA');
結果：C
```



#### LTRIM

|      |      |
|------|-------|
| 書式 | LTRIM(*str* [, *trim_str*]) |

文字列*str*の左端から、文字列*trim_str*のすべての文字を削除します。

- 引数*str*, *trim_str*には、文字列型の値を指定します。
- 引数*trim_str*の文字列に含まれるすべての文字を削除します。省略すると、*str*の左端からスペースを削除します。
- 結果の型は文字列型です。

例)
```example
SELECT LTRIM(' ABC ');
結果：ABC  (左端にスペース無し)

SELECT LTRIM('ABCAA', 'A');
結果：BCAA
```



#### RTRIM

|      |      |
|------|-------|
| 書式 | RTRIM(*str* [, *trim_str*]) |

文字列*str*の右端から、文字列*trim_str*のすべての文字を削除します。

- 引数*str*, *trim_str*には、文字列型の値を指定します。
- 引数*trim_str*の文字列に含まれるすべての文字を削除します。省略すると、*str*の右端からスペースを削除します。
- 結果の型は文字列型です。

例)
```example
SELECT RTRIM(' ABC ');
結果： ABC  (右端にスペース無し)

SELECT RTRIM('ABCAA', 'A');
結果：ABC
```



#### QUOTE

|      |      |
|------|-------|
| 書式 | QUOTE(*x*) |

*x*の値をシングルクォートで囲んだ文字列を返します。

- 引数*x*には、文字列型、数値型、TIMESTAMP型、BLOB型の値を指定します。
  - 文字列型の場合は、文字列に含まれるシングルクォートは2つのシングルクォート''にエスケープします。
  - 数値型の場合は、そのままの数値が返ります。シングルクォートでは囲まれません。
    - TIMESTAMP型(精度指定付きTIMESTAMP型)の場合は、'YYYY-MM-DDThh:mm:ss.SSS(Z|±hh:mm)'形式の時刻の文字列表記に変換して連結します。シングルクォートでは囲まれません。時刻の小数部の桁数は引数のTIMESTAMP型の精度に応じて決まります。詳細は、[TIMESTAMP_MS関数](#timestamp_ms)など、対応する精度の関数の説明を参照してください。
  - BLOB型の場合は、X'BLOB型の値'の文字列を返します。
- 結果の型は文字列型です。

例)
```example
SELECT QUOTE(last_name) last_name, QUOTE(age) age FROM employees;
結果：
  last_name     age
  ------------+-------
  'Smith'       43
  'Jones'       59
  'Brown'       (NULL)
  'Taylor'      31
  (NULL)        29
  'Smith'       43

SELECT QUOTE(RANDOMBLOB(4));
結果：X'A45EA28D'

// カラムvalueの値は「Today's news」の文字列
SELECT value, QUOTE(value) FROM testcontainer;
結果：
   value            QUOTE(value)
  ---------------+-------------------
   Today's news     'Today''s news'
```



#### UNICODE

|      |      |
|------|-------|
| 書式 | UNICODE(*str*) |

文字列*str*の最初の文字のUNICODEコードポイントを返します。

- 引数*str*には、文字列型の値を指定します。
- 結果の型はLONG型です。

例)
```example
SELECT last_name, UNICODE(last_name) unicode FROM employees;
結果：
  last_name     unicode
  ------------+----------------------
  Smith         83
  Jones         74
  Brown         66
  Taylor        84
  (NULL)        (NULL)
  Smith         83
```



#### CHAR

|      |      |
|------|-------|
| 書式 | CHAR(*x1* [, *x2*, ... , *xn*]) |

Unicodeコードポイントの値*xn*の文字を連結した文字列を返します。

- 引数*xn*には、Unicodeコードポイントの値を指定します。
- 結果の型はSTRING型です。

例)
```example
SELECT CHAR(83, 84, 85);
結果：STU
```



#### PRINTF

|      |      |
|------|-------|
| 書式 | PRINTF(*format* [, *x1*, *x2*, ..., *xn*]) |

指定されたフォーマット*format*に合わせて変換した文字列を返します。
標準Cライブラリのprintf関数と同等のフォーマットが使用できます。
それ以外のフォーマットとしては以下の2つがあります。

| フォーマット | 説明 |
|----|-----------------------|
| %q | 文字列中にシングルクォートがある場合、2つのシングルクォート''にエスケープします。  |
| %Q | 文字列中にシングルクォートがある場合、2つのシングルクォート''にエスケープします。<br>文字列の両端をシングルクォートで囲みます。   |

例)
```example
SELECT enrollment_period, PRINTF('%.2f', enrollment_period) printf FROM employees;
結果：
  enrollment_period   printf
  ------------------+-----------
  15.5                15.50
  23.2                23.20
   7.0                 7.00
  (NULL)               0.00
   4.9                 4.90
  10.3                10.30
```

#### TRANSLATE
|     |                                                   |
| --- | ------------------------------------------------- |
| 書式  | TRANSLATE(*str*, *search_str*, *replacement_str*) |

文字列を置換します。文字列*str*のうち、文字列*search_str*と一致する文字が、*search_str*と同じ位置にある文字列*replacement_str*の文字で置換されます。*replacement_str*が*search_str*より短く、置換後の文字がない場合、置換対象の文字は削除されます。

- 引数*str*、*search_str*、*replacement_str*には、文字列型の値を指定します。
- 値がNULLの場合は、NULLを返します。
- 結果の型は文字列型です。

例)

```example
SELECT TRANSLATE('abcde', 'ace', '123');
結果：1b2d3

SELECT TRANSLATE('abcdeca', 'ace', '123');
結果：1b2d321

SELECT TRANSLATE('abcde', 'ac', '123');
結果：1b2de

SELECT TRANSLATE('abcde', 'ace', '12');
結果：1b2d

SELECT TRANSLATE('abcde', 'AB', '123');
結果：abcde

SELECT TRANSLATE('abcde', 'abc', '');
結果：de
```

<a id="time_function"></a>
### 日時関数

#### NOW

|      |      |
|------|-------|
| 書式 | NOW() |

現在時刻の値を返します。

- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はTIMESTAMP型(ミリ秒精度)です。

例)
```example
SELECT NOW();
結果：2019-09-17T04:07:31.825Z

SELECT NOW();
結果：2019-09-17T13:09:20.918+09:00
```

#### TIMESTAMP

|      |      |
|------|-------|
| 書式 | TIMESTAMP(*timestamp_string* [, timezone]) |

時刻の文字列表現*timestamp_string*の値を、TIMESTAMP型(ミリ秒精度)に変換します。
次のTIMESTAMP_MSと同等の関数です。詳細についてはTIMESTAMP_MSを参照してください。

#### TIMESTAMP_MS

|      |      |
|------|-------|
| 書式 | TIMESTAMP_MS(*timestamp_string* [, timezone]) |

時刻の文字列表現*timestamp_string*の値を、ミリ秒精度のTIMESTAMP(3)型に変換します。

- 引数*timestamp_string*には、時刻の文字列表現として、次の形式の文字列を指定します。
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DD
  - hh:mm:ss
  
  | 表記 | 内容      | 値の範囲   |
  |------|----------|------------|
  | YYYY | 年(西暦) | 1970～ |
  | MM   | 月　　　 | 1～12 |
  | DD   | 日　　　 | 1～31 |
  | hh   | 時間(24時間表記)　　　 | 0～23 |
  | mm   | 分　　　 | 0～59 |
  | ss   | 秒　　　 | 0～59 |
  | SSS  | ミリ秒　 | 0～999 |
  | Z    | タイムゾーン | Z\|±hh:mm\|±hhmm |

- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。*timestamp_string*にタイムゾーン情報が含まれる場合は指定する必要はありません。指定して矛盾がある場合はエラーが返ります。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はミリ秒精度のTIMESTAMP(3)型です。
- TIMESTAMP_MS関数の逆変換(ミリ秒精度のTIMESTAMP(3)型から文字列型への変換)は、[CAST](#cast)を用いてください。
  + CAST(*timestamp* AS STRING)

例)
```example
// カラムdate(TIMESTAMP型)の値が、時刻'2018-12-01T10:30:00Z'より新しいロウを検索します
SELECT * FROM timeseries WHERE date > TIMESTAMP('2018-12-01T10:30:00Z');
```


#### TIMESTAMP_US

|      |      |
|------|-------|
| 書式 | TIMESTAMP_US(*timestamp_string* [, timezone]) |

時刻の文字列表現*timestamp_string*の値を、マイクロ秒精度のTIMESTAMP(6)型に変換します。

- 引数*timestamp_string*には、時刻の文字列表現として、次の形式の文字列を指定します。
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSZ
  - YYYY-MM-DD
  - hh:mm:ss
    
  | 表記 | 内容      | 値の範囲   |
  |------|----------|------------|
  | YYYY | 年(西暦) | 1970～ |
  | MM   | 月　　　 | 1～12 |
  | DD   | 日　　　 | 1～31 |
  | hh   | 時間(24時間表記)　　　 | 0～23 |
  | mm   | 分　　　 | 0～59 |
  | ss   | 秒　　　 | 0～59 |
  | SSSSSS  | マイクロ秒　 | 0～999999 |
  | Z    | タイムゾーン | Z\|±hh:mm\|±hhmm |

- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。*timestamp_string*にタイムゾーン情報が含まれる場合は指定する必要はありません。指定して矛盾がある場合はエラーが返ります。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はマイクロ秒精度のTIMESTAMP(6)型です。
- TIMESTAMP_US関数の逆変換(マイクロ秒精度のTIMESTAMP(6)型から文字列型への変換)は、[CAST](#cast)を用いてください。
  + CAST(*timestamp* AS STRING)


#### TIMESTAMP_NS

|      |      |
|------|-------|
| 書式 | TIMESTAMP_NS(*timestamp_string* [, timezone]) |

時刻の文字列表現*timestamp_string*の値を、ナノ秒精度のTIMESTAMP(9)型に変換します。

- 引数*timestamp_string*には、時刻の文字列表現として、次の形式の文字列を指定します。
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSSSSZ
  - YYYY-MM-DD
  - hh:mm:ss

  
  | 表記 | 内容      | 値の範囲   |
  |------|----------|------------|
  | YYYY | 年(西暦) | 1970～ |
  | MM   | 月　　　 | 1～12 |
  | DD   | 日　　　 | 1～31 |
  | hh   | 時間(24時間表記)　　　 | 0～23 |
  | mm   | 分　　　 | 0～59 |
  | ss   | 秒　　　 | 0～59 |
  | SSSSSSSSS  | ナノ秒　 | 0～999999999 |
  | Z    | タイムゾーン | Z\|±hh:mm\|±hhmm |

- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。*timestamp_string*にタイムゾーン情報が含まれる場合は指定する必要はありません。指定して矛盾がある場合はエラーが返ります。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はマイクロ秒精度のTIMESTAMP(9)型です。
- TIMESTAMP_NS関数の逆変換(ナノ秒精度のTIMESTAMP(9)型から文字列型への変換)は、[CAST](#cast)を用いてください。
  + CAST(*timestamp* AS STRING)


#### TIMESTAMP_ADD

|      |      |
|------|-------|
| 書式 | TIMESTAMP_ADD(*time_unit*, *timestamp*, *duration* [, timezone])

時刻*timestamp*に、期間*duration*(単位*time_unit*)を加算した値を返します。

- 引数*timestamp*には、TIMESTAMP型の値を指定します。また、精度値指定付きTIMESTAMP型も指定できます。
- 引数*duration*には、整数を指定します。負の数を指定した場合は、時刻を減算します。
- 引数*time_unit*には、次のいずれかの識別子を指定します。
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND
- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。
- 年もしくは月の加算によって算出された月に同一日が存在しない場合、日はその月の最終日に丸められます。例えば5月31日に1月加算すると、6月31日は存在しないため6月30日に丸められます。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はTIMESTAMP型です。
- 関数の別名としてTIMESTAMPADDも利用可能です。

例)
```example
// 時刻'2018-12-01T11:22:33.444Z'に10日間を加算します
SELECT TIMESTAMP_ADD(DAY, TIMESTAMP('2018-12-01T11:22:33.444Z'), 10);
結果：2018-12-11T11:22:33.444Z

SELECT TIMESTAMP_ADD(MONTH, TIMESTAMP('2019-05-31T01:23:45.678Z'), 1);
結果：2019-06-30T01:23:45.678Z

SELECT TIMESTAMP_ADD(MONTH, TIMESTAMP('2019-05-31T01:23:45.678Z'), 1, '-02:00');
結果：2019-07-01T01:23:45.678Z

```

 - 時間単位*time_unit*の指定はMILLISECONDまでです。


#### TIMESTAMP_DIFF

|      |      |
|------|-------|
| 書式 | TIMESTAMP_DIFF(*time_unit*, *timestamp1*, *timestamp2* [, timezone])

時刻*timestamp1*と*timestamp2*の差分の時間(*timestamp1*-*timestamp2*)を、時間単位*time_unit*で表した値で返します。
差分を時間単位で表す際に、小数点以下は切り捨てます。

- 引数*timestamp1*と*timestamp2*には、TIMESTAMP型の値を指定します。また、精度値指定付きTIMESTAMP型も指定できます。
- 引数*time_unit*には、次のいずれかの識別子を指定します。識別子で指定の単位のみの差分を計算するのではなく、識別子未満の単位も計算に利用されます。例えばMONTH指定で2019/09/30と2019/10/02を比較する場合、0ヶ月2日が差分となるため、出力は1ではなく0となります。
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND
- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が差分計算に利用されます。
- 結果の型はLONG型です。
- 関数の別名としてTIMESTAMPDIFFも利用可能です。

例)
```example

// 時間単位：月
SELECT TIMESTAMP_DIFF(MONTH, TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
結果：0

// 時間単位：日
SELECT TIMESTAMP_DIFF(DAY,   TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
結果：10
SELECT TIMESTAMP_DIFF(DAY,   TIMESTAMP('2018-12-01T11:00:00.000Z'), TIMESTAMP('2018-12-11T10:30:15.555Z'));
結果：-9

// 時間単位：時間
SELECT TIMESTAMP_DIFF(HOUR,  TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
結果：240

// 時間単位：分
SELECT TIMESTAMP_DIFF(MINUTE, TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
結果：14430

// タイムゾーンによって結果が変わる例を示します。
SELECT TIMESTAMP_DIFF(MONTH, MAKE_TIMESTAMP(2019, 8, 1), MAKE_TIMESTAMP(2019, 6, 30), 'Z');
結果：2

SELECT TIMESTAMP_DIFF(MONTH, MAKE_TIMESTAMP(2019, 8, 1), MAKE_TIMESTAMP(2019, 6, 30), '-01:00');
結果：1

```

【メモ】
- 時間単位*time_unit*の指定は、MILLISECONDまでです。


#### TO_TIMESTAMP_MS

|      |      |
|------|-------|
| 書式 | TO_TIMESTAMP_MS(*milliseconds*) |

時刻'1970-01-01T00:00:00.000Z'に、引数millisecondsの値をミリ秒として加算した時刻を返します。

この関数は、TO_EPOCH_MS関数の逆変換です。

- 引数*milliseconds*には、整数を指定します。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。
- 結果の型はTIMESTAMP型です。

例)
```example
SELECT TO_TIMESTAMP_MS(1609459199999);
結果：2020-12-31T23:59:59.999Z
```



#### TO_EPOCH_MS

|      |      |
|------|-------|
| 書式 | TO_EPOCH_MS(*timestamp*) |

時刻'1970-01-01T00:00:00.000Z'から時刻*timestamp*までの経過時間(ミリ秒)を返します。

この関数は、TO_TIMESTAMP_MS関数の逆変換です。

- 引数*timestamp*には、TIMESTAMP型の値を指定します。精度指定付きTIMESTAMP型の値も指定可能です。
- 結果の型はLONG型です。

例)
```example
SELECT TO_EPOCH_MS(TIMESTAMP('2020-12-31T23:59:59.999Z'));
結果：1609459199999

SELECT TO_EPOCH_MS(TIMESTAMP('2020-12-31T23:59:59.999+09:00'));
結果：1609426799999
```

【メモ】
- マイクロ秒、ナノ秒精度のTIMESTAMP型の値を指定した場合も、求められる値はミリ秒精度になります。
  

#### EXTRACT

|      |      |
|------|-------|
| 書式 | EXTRACT(*time_field*, *timestamp* [, timezone]) |

時刻*timestamp*から、日時フィールド*time_field*の値を取り出します。時刻はUTCの値になります。

- 引数*timestamp*には、TIMESTAMP型の値を指定します。また、精度値指定付きTIMESTAMP型も指定できます。
- 引数*time_field*には、次の値のいずれかを指定します。
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND | MICROSECOND | NANOSECOND | DAY_OF_WEEK | DAY_OF_YEAR
    - DAY_OF_WEEKは日曜が始点で0、土曜が終点で6となります。
    - DAY_OF_YEARは1月1日が始点で1、12月31日が終点で365または366となります。
    - MILISECOND,MICROSECOND,NANOSECONDを指定した場合、いずれも時刻の小数部全てを含む値になります。
- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。
- 接続時にタイムゾーンが指定されている場合、オフセット計算された値が返ります。引数*timezone*と両方で指定されている場合は引数で指定したものが利用されます。
- 結果の型はLONG型です。


例)
```example
// 時刻'2018-12-01T10:30:02.392Z'の年、日、ミリ秒の値を求めます

// 年の値
SELECT EXTRACT(YEAR, TIMESTAMP('2018-12-01T10:30:02.392Z'));
結果：2018

SELECT EXTRACT(DAY, TIMESTAMP('2018-12-01T10:30:02.392Z'));
// 日の値
結果：1

// ミリ秒の値
SELECT EXTRACT(MILLISECOND, TIMESTAMP('2018-12-01T10:30:02.392Z'));
結果：392


// タイムゾーンを考慮します。
SELECT EXTRACT(HOUR, TIMESTAMP('2018-12-01T10:30:02.392Z'), '+09:00');
結果：19
```

#### STRFTIME

|      |      |
|------|-------|
| 書式 | STRFTIME(*format*, *timestamp* \[, *modifier*,...\])

指定されたフォーマットに合わせて時刻を文字列に変換して返します。

- 引数*format*には、下記を指定して時刻情報を取り出します。

| フォーマット | 説明 |
|----|-----------------------|
| %Y | 年をYYYY形式で取り出します。  |
| %m | 月をMM形式で取り出します。 |
| %d | 日をDD形式で取り出します。 |
| %H | 時をhh形式で取り出します。 |
| %M | 分をmm形式で取り出します。 |
| %S | 秒をss形式で取り出します。 |
| %3f| 小数部をミリ秒精度のSSS形式で取り出します。 |
| %6f| 小数部をマイクロ秒精度のSSSSSS形式で取り出します。 |
| %9f| 小数部をナノ秒精度のSSSSSSSSS形式で取り出します。 |
| %z | タイムゾーンを±hh:mm形式で取り出します。 |
| %w | 曜日をD形式(0~6)で取り出します。日曜日が起点で0、土曜日が終点で6となります。 |
| %W | その年の初めから何週目かをDD形式(00~53)で取り出します。最初の月曜日を1週目とし、それ以前の曜日は0週目とみなします。 |
| %j | 1月1日を起点とした日数をDDD形式(001~366)で取り出します。 |
| %c | 時刻をYYYY-MM-DDThh:mm:ss[.SSS]\(Z&#124;±hh:mm&#124;±hhmm)形式で取り出します。 |
| %% | %を文字として出力します。 |

- 引数*timestamp*には、TIMESTAMP型の値を指定します。また、精度値指定付きTIMESTAMP型も指定できます。
- 引数*modifier*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。
- 結果の型はSTRING型です。

例)
```example

SELECT STRFTIME('%c', TIMESTAMP('2019-06-19T14:15:01.123Z'));
結果：2019-06-19T14:15:01.123Z

SELECT STRFTIME('%H:%M:%S%z', TIMESTAMP('2019-06-19T14:15:01.123Z'), '+09:00');
結果：23:15:01+09:00

SELECT STRFTIME('%W', TIMESTAMP('2019-01-19T14:15:01.123Z'));
結果：02
```

#### MAKE_TIMESTAMP

|      |      |
|------|-------|
| 書式 | MAKE_TIMESTAMP(year, month, day [, timezone])<br>MAKE_TIMESTAMP(year, month, day, hour, min, sec [, timezone])|

TIMESTAMP型の値を生成して返します。

- 引数*hour*, *min*, *sec*を指定しない場合は、全て0を指定したものとみなします。
- 引数*sec*には、ミリ秒単位の指定が可能です。ミリ秒未満は四捨五入した値に丸められますが、浮動小数点演算の仕組みに基づく誤差が生じる可能性があります。
- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。
- 結果の型はTIMESTAMP型(ミリ秒精度)です。

例)
```example

SELECT MAKE_TIMESTAMP(2019, 9, 19);
結果：2019-09-19T00:00:00.000Z

SELECT MAKE_TIMESTAMP(2019, 9, 19, 10, 30, 15.123, '+09:00');
結果：2019-09-19T01:30:15.123Z

```

#### TIMESTAMP_TRUNC

|      |      |
|------|-------|
| 書式 | TIMESTAMP_TRUNC(field, timestamp [, timezone])|

時刻情報を切り捨てます。

- 引数*field*には、次のいずれかの識別子を指定します。
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND | MICROSECOND | NANOSECOND
- 引数*timestamp*には、TIMESTAMP型の値を指定します。また、精度値指定付きTIMESTAMP型も指定できます。
- 引数*timezone*には、タイムゾーン(Z|±hh:mm|±hhmm)を指定します。

例)
```example

SELECT TIMESTAMP_TRUNC(HOUR, MAKE_TIMESTAMP(2019, 9, 19, 10, 30, 15.123));
結果：2019-09-19T10:00:00.000Z

SELECT TIMESTAMP_TRUNC(DAY, MAKE_TIMESTAMP(2019, 5, 15), '-01:00');
結果：2019-05-14T01:00:00.000Z

```


<a id="window_function"></a>
### WINDOW関数

- WINDOW関数はOVER句と共に利用します。詳細は[OVER句](#over)を参照ください。

#### ROW_NUMBER

|      |      |
|------|-------|
| 書式 | ROW_NUMBER()  |

結果のロウに対して、一意となる連番値を割り振ります。

- OVER句と共に利用します。詳細は[OVER句](#over)を参照してください。

例)
```example
SELECT ROW_NUMBER() OVER(PARTITION BY department ORDER BY age) no, first_name, age, department FROM employees;
結果：
  no   first_name   age      department
  ----+------------+--------+-------------
  1    James        43       Development
  2    William      59       Development
  1    Mary         31       Research
  1    John         43       Sales
  2    Richard      (NULL)   Sales
  1    Lisa         29       (NULL)
```

#### LAG

|      |       |
|------|-------|
| 書式 | 書式 LAG( x [, offset [, default ] ] ) |

現在行よりも、*offset*行数分、前方の*x*を返します。

- 引数*x*には、任意の型の値を指定します。
- 引数*offset*には、LONG型の値を指定します。   
  0は現在位置、正の数は前方位置、負の数は後方位置の行を表します。   
  指定を省略した場合、*offset*は1になります。
- 引数*default*には、任意の型の値を指定します。   
  該当行が存在しない場合に*default*が返却されます。   
  指定を省略した場合、*default*はNULLになります。
- 結果の型は引数*x*、もしくは*default*の型と同じです。
- 引数*x*、*default*には、同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。型の組み合わせは[CASE](#case)を参照してください。

例)
```example
SELECT id, date, empId, amount, LAG(amount) OVER(PARTITION BY empID ORDER BY id) as lag_amount FROM travelexpenses;
結果：
 id    date         empId   amount   lag_amount   
-----+------------+-------+--------+------------
  101  2020/02/01   0       200      (NULL)     
  104  2020/02/04   0       200      200        
  105  2020/02/05   0       150      200        
  102  2020/02/03   2       2500     (NULL)     
  103  2020/02/03   3       60       (NULL)     
  106  2020/02/06   3       80       60         
```

#### LEAD

|      |       |
|------|-------|
| 書式 | 書式 LEAD( x [, offset [, default ] ] ) |

現在行よりも、*offset*行数分、後方の*x*を返します。

- 引数*x*には、任意の型の値を指定します。
- 引数*offset*には、LONG型の値を指定します。   
  0は現在位置、正の数は後方位置、負の数は前方位置の行を表します。   
  指定を省略した場合、*offset*は1になります。
- 引数*default*には、任意の型の値を指定します。   
  該当行が存在しない場合に*default*が返却されます。   
  指定を省略した場合、*default*はNULLになります。
- 結果の型は引数*x*、もしくは*default*の型と同じです。
- 引数*x*、*default*には、同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。型の組み合わせは[CASE](#case)を参照してください。

例)
```example
SELECT id, date, empId, amount, LEAD(amount) OVER(PARTITION BY empID ORDER BY id) as lead_amount FROM travelexpenses;
結果：
 id    date         empId   amount   lead_amount 
-----+------------+-------+--------+-------------
  101  2020/02/01   0       200      200         
  104  2020/02/04   0       200      150         
  105  2020/02/05   0       150      (NULL)      
  102  2020/02/03   2       2500     (NULL)      
  103  2020/02/03   3       60       80          
  106  2020/02/06   3       80       (NULL)      
```


<a id="other_function"></a>
### その他の関数

#### COALESCE

|      |      |
|------|-------|
| 書式 | COALESCE(*x1*, *x2* [,..., *xn*]) |

指定された引数*xn*の中で、NULLではない最初の引数の値を返します。

- 引数*xn*には、同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。型の組み合わせは[CASE](#case)を参照してください。


- 引数の値がすべてNULLの場合は、NULLを返します。


例)
```example
SELECT last_name, COALESCE(last_name, 'XXX') coalesce FROM employees;
結果：
  last_name     coalesce
  ------------+----------------------
  Smith         Smith
  Jones         Jones
  Brown         Brown
  Taylor        Taylor
  (NULL)        XXX
  Smith         Smith

SELECT age, COALESCE(age, -1) coalesce FROM employees;
結果：
  age       coalesce
  --------+-----------
  43         43
  59         59
  (NULL)     -1
  31         31
  29         29
  43         43
```



#### IFNULL

|      |      |
|------|-------|
| 書式 | IFNULL(*x*, *y*) |

指定された引数*x*と*y*のうち、NULLではない最初の引数の値を返します。IFNULL関数は、引数を2つ指定したCOALESCE関数と同等です。

- 引数*x*と*y*には、同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。型の組み合わせは[CASE](#case)を参照してください。
- 引数の値がすべてNULLの場合は、NULLを返します。

例)
```example
SELECT last_name, IFNULL(last_name, 'XXX') ifnull FROM employees;
結果：
  last_name     ifnull
  ------------+----------------------
  Smith         Smith
  Jones         Jones
  Brown         Brown
  Taylor        Taylor
  (NULL)        XXX
  Smith         Smith

SELECT age, IFNULL(age, -1) ifnull FROM employees;
結果：
  age       coalesce
  --------+-----------
  43         43
  59         59
  (NULL)     -1
  31         31
  29         29
  43         43
```



#### NULLIF

|      |      |
|------|-------|
| 書式 | NULLIF(*x*, *y*) |

指定された2つの引数が同じ値の場合はNULL、異なる場合は最初の引数を返します。

- 引数*x*と*y*には、同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。型の組み合わせは[CASE](#case)を参照してください。

例)
```example
// value1とvalue2の値で、NULLIFを実行します
SELECT value1, value2, NULLIF(value1, value2) nullif FROM container_sample;
結果：
   value1   value2   nullif
  --------+--------+--------
      10       10    (NULL)
       5        0      5
   (NULL)       4    (NULL)
       3    (NULL)     3
   (NULL)   (NULL)   (NULL)


// value1/value2の計算で、ゼロ除算エラーを防ぐために0をNULLに変換します
SELECT value1, value2, value1/NULLIF(value2, 0) division FROM container_sample;
結果：
   value1   value2   division
  --------+--------+--------
      10       10      1
       5        0    (NULL)
   (NULL)       4    (NULL)
       3    (NULL)   (NULL)
   (NULL)   (NULL)   (NULL)
```



#### RANDOMBLOB

|      |      |
|------|-------|
| 書式 | RANDOMBLOB(*size*) |

BLOB型の値(乱数)を返します。

- 引数*size*は、BLOB型の値のサイズ(バイト数)を整数で指定します。
- 結果の型はBLOB型です。

例)　
```example
// 10バイトのBLOB型の値(乱数)を生成します
SELECT HEX(RANDOMBLOB(10));
結果：7C8C893C8087F07883AF
```



#### ZEROBLOB

|      |      |
|------|-------|
| 書式 | ZEROBLOB(*size*) |

BLOB型の値(0x00)を返します。

- 引数*size*は、BLOB型の値のサイズ(バイト数)を整数で指定します。
- 結果の型はBLOB型です。

例)　
```example
// 10バイトのBLOB型の値(0x00)を生成します
SELECT HEX(ZEROBLOB(10));
結果：00000000000000000000
```



#### HEX

|      |      |
|------|-------|
| 書式 | HEX(*x*) |

BLOB型の値を16進表記に変換します。
引数*x*をBLOB型の値として解釈して、16進数に変換した文字列(大文字)を返します。

- 引数*x*には、BLOB型、文字列型を指定します。
  - 文字列型の場合は、すべての文字のUnicodeコードポイントを16進数に変換した文字列を返します。
- 結果の型は文字列型です。

例)
```example
SELECT HEX(RANDOMBLOB(2));
結果：E18D

SELECT first_name, HEX(first_name) hex FROM employees;
結果：
  first_name    hex
  ------------+----------------------
  John          4A6F686E
  William       57696C6C69616D
  Richard       52696368617264
  Mary          4D617279
  Lisa          4C697361
  James         4A616D6573
```



#### TYPEOF

|      |      |
|------|-------|
| 書式 | TYPEOF(*x*) |

*x*の値のデータ型を表す文字列を返します。

- データ型と、TYPEOF関数が返す文字列の対応を以下に示します。

  | データ型           | TYPEOF関数が返す文字列 |
  |-------------------|-----------------------|
  | BOOL型            | BOOL |
  | STRING型          | STRING |
  | BYTE型            | BYTE |
  | SHORT型           | SHORT |
  | INTEGER型         | INTEGER |
  | LONG型            | LONG |
  | FLOAT型           | FLOAT |
  | DOUBLE型          | DOUBLE |
  | TIMESTAMP(3)型    | TIMESTAMP |
  | TIMESTAMP(6)型    | TIMESTAMP(6) |
  | TIMESTAMP(9)型    | TIMESTAMP(9) |
  | GEOMETRY型        | NULL |
  | BLOB型            | BLOB |
  | 配列型            | NULL |

- 結果の型は文字列型です。
- NULL値を指定した場合は、'NULL'が返ります。

例)
```example
SELECT TYPEOF(ABS(-10)) abs, TYPEOF(RANDOMBLOB(10)) randomblob,
    TYPEOF(TIMESTAMP('2018-12-01T10:30:02.392Z')) timestamp;
結果：
   abs    randomblob   timestamp
  ------+------------+-----------
   LONG   BLOB         TIMESTAMP
```

　

## その他構文

### CAST

|      |      |
|------|-------|
| 書式 | CAST(*x* AS *data_type*) |

値*x*を、データ型*data_type*に変換します。

- 引数*data_type*には、変換後のデータ型に応じて以下の値を指定します。

  | 変換後のデータ型   | *data_type*に指定する値 |
  |-------------------|-----------------------|
  | BOOL型            | BOOL |
  | STRING型          | STRING |
  | BYTE型            | BYTE |
  | SHORT型           | SHORT |
  | INTEGER型         | INTEGER |
  | LONG型            | LONG |
  | FLOAT型           | FLOAT |
  | DOUBLE型          | DOUBLE |
  | TIMESTAMP(3)型       | TIMESTAMPまたはTIMESTAMP(3) |
  | TIMESTAMP(6)型       | TIMESTAMP(6) |
  | TIMESTAMP(9)型       | TIMESTAMP(9) |
  | BLOB型            | BLOB |


#### 文字列型への変換

|      |      |
|------|-------|
| 書式 | CAST(*x* AS STRING) |

引数*x*を、文字列型に変換します。

*x*に指定できる値のデータ型と、変換後の値は以下の通りです。

| *x*のデータ型      | 文字列型に変換した値 |
|-------------------|--------------------|
| BOOL型            | trueの場合'true'、falseの場合'false' |
| STRING型          | 元のままの値   |
| BYTE型<br>SHORT型<br>INTEGER型<br>LONG型<br>FLOAT型<br>DOUBLE型    | 数値を文字列に変換した値   |
| TIMESTAMP(3)型       | ミリ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSS(Z\|±hh:mm)' <br>接続時のタイムゾーン設定が利用される |
| TIMESTAMP(6)型       | マイクロ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSSSSS(Z\|±hh:mm)' <br>接続時のタイムゾーン設定が利用される |
| TIMESTAMP(9)型       | ナノ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSSSSSSSS(Z\|±hh:mm)' <br>接続時のタイムゾーン設定が利用される |
| BLOB型            | [HEX関数](#hex)と同等の文字列   |


#### 数値型への変換

|      |      |
|------|-------|
| 書式 | CAST(*x* AS BYTE\|SHORT\|INTEGER\|LONG\|FLOAT\|DOUBLE) |

引数*x*を、数値型に変換します。

*x*に指定できる値のデータ型と、変換後の値は以下の通りです。

| *x*のデータ型      | 数値型に変換した値 |
|-------------------|--------------------|
| BOOL型            | trueの場合1、falseの場合0 |
| STRING型          | 文字列の数字を数値に変換した値  |
| BYTE型<br>SHORT型<br>INTEGER型<br>LONG型<br>FLOAT型<br>DOUBLE型    | 数値を変換した値   |

- 変換後の数値が、*data_type*に指定した数値型の値の範囲を超える場合は、エラーになります。

```example
// BYTE型の範囲(-128～127)を超える場合はエラー
SELECT CAST(128 AS BYTE);
結果：エラー

// INTEGER型の範囲(-2147483648 ～ 2147483647)を超える場合はエラー
SELECT CAST('2147483648' AS INTEGER);
結果：エラー
```

- 浮動小数点数型(FLOAT型、DOUBLE型)から整数型(BYTE型、SHORT型、INTEGER型、LONG型)への変換では、値が桁落ちする場合があります。

```example
SELECT CAST(10.5 AS INTEGER);
結果：10
```

- 文字列型から数値型への変換では、次の文字列が指定できます(大小同一視)。これら以外の文字列が指定されている場合はエラーになります。
  - 数字と記号(".","-","+")と"E"のいずれかを含む文字列
  - "Inf" (符号付きも可)
  - "Infinity" (符号付きも可)
  - "NaN"

```example
SELECT CAST('abc' AS INTEGER);
結果：エラー

SELECT CAST('-1.09E+10' AS DOUBLE);
結果：-1.09E10
```


#### 時刻型への変換

|      |      |
|------|-------|
| 書式 | CAST(*x* AS TIMESTAMP) |

引数*x*を、時刻型に変換します。接続時にタイムゾーンを指定している場合、その値がオフセット計算に利用されます。
精度指定付きTIMESTAMP型も指定可能です。

*x*に指定できる値のデータ型と、変換後の値は以下の通りです。

| *x*のデータ型                                       | 時刻型に変換した値                             |
|----------------------------------------------------|-----------------------------------------------|
| STRING型(ミリ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSS(Z\|±hh:mm)') | [TIMESTAMP_MS関数](#timestamp_ms)で変換した値と同等(精度指定によらず変換可能)   |
| STRING型(マイクロ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSSSSS(Z\|±hh:mm)') | [TIMESTAMP_US関数](#timestamp_us)で変換した値と同等(マイクロまたはナノ秒精度指定時のみ変換可能)   |
| STRING型(ナノ秒精度時刻の文字列表記'YYYY-MM-DDThh:mm:ss.SSSSSSSSS(Z\|±hh:mm)') | [TIMESTAMP_NS関数](#timestamp_ns)で変換した値と同等(ナノ秒精度指定時のみ変換可能)   |

```example
SELECT CAST('2018-12-01T10:30:00Z' AS TIMESTAMP);
結果：2018-12-01T10:30:00.000Z

SELECT CAST('2018-12-01T10:30:00+09:00' AS TIMESTAMP);
結果：2018-12-01T01:30:00.000Z
```


#### BOOL型への変換

|      |      |
|------|-------|
| 書式 | CAST(*x* AS BOOL) |

引数*x*を、BOOL型に変換します。

*x*に指定できる値のデータ型と、変換後の値は以下の通りです。

| *x*のデータ型      | 時刻型に変換した値 |
|-------------------|--------------------|
| STRING型          | 'true'の場合true、'false'の場合false (大文字小文字の区別なし)   |
| BYTE型<br>SHORT型<br>INTEGER型<br>LONG型 | 0の場合false、それ以外の場合true |


#### BLOB型への変換

|      |      |
|------|-------|
| 書式 | CAST(*x* AS BLOB) |

引数*x*を、BLOB型に変換します。

*x*に指定できる値のデータ型と、変換後の値は以下の通りです。

| *x*のデータ型      | BLOB型に変換した値 |
|-------------------|--------------------|
| STRING型          | 文字列を16進表記のデータとしてBLOB型に変換した値   |

　

### CASE

|      |      |
|------|-------|
| 書式 | CASE <br>WHEN *condition1* THEN *result1* <br>[WHEN *condition2* THEN *result2*]<br>... <br>[ELSE *resultElse*] <br>END |

条件式*conditionN*がtrueの場合は、対応する*resultN*の値を返します。
すべての条件式がfalseまたはNULLの場合は、ELSEが指定されていれば*resultElse*の値を返します。ELSEが指定されていない場合は、NULLを返します。

　

|      |      |
|------|-------|
| 書式 | CASE *x* <br>WHEN *value1* THEN *result1* <br>[WHEN *value2* THEN *result2*]<br>... <br>[ELSE *resultElse*] <br>END |

*x*の値が*valueN*の場合は、対応する*resultN*の値を返します。
すべての値に当てはまらない場合は、ELSEが指定されていれば*resultElse*の値を返します。ELSEが指定されていない場合は、NULLを返します。

　

*resultN*には同じ型の値を指定します。ただし、異なる型でも指定できる場合があります。
- 引数が異なる型同士の場合は、以下の型の組合せのみ演算できます。これ以外の組合せの場合はエラーになります。

  | 引数の型 | 引数の型                 | 2つの引数を演算する際の型 |
  |-|-|-|
  | SHORT   | BYTE                              | LONG |
  | INTEGER | BYTE, SHORT                       | LONG |
  | LONG    | BYTE, SHORT, INTEGER              | LONG |
  | FLOAT   | BYTE, SHORT, INTEGER, LONG        | DOUBLE |
  | DOUBLE  | BYTE, SHORT, INTEGER, LONG, FLOAT | DOUBLE |


例)
```example
// 従業員の年代(30代、40代、50代、それ以外)を表示する
SELECT id, first_name, age,
  CASE
    WHEN age > 50 THEN '50s'
    WHEN age > 40 THEN '40s'
    WHEN age > 30 THEN '30s'
    ELSE 'other'
  END AS period
FROM employees;

結果：
　id   first_name   age     period
　----+------------+-------+--------
　0    John          43      40s
　1    William       59      50s
　2    Richard      (NULL)   other
　3    Mary          31      30s
　4    Lisa          29      other
　5    James         43      40s


// 部署に応じて所在地を表示する
SELECT id, first_name, department,
  CASE department
    WHEN 'Sales' THEN 'Tokyo'
    WHEN 'Development' THEN 'Osaka'
    ELSE 'Nagoya'
  END AS location
FROM employees;

結果：
　id   first_name   department    location
　----+------------+-------------+---------
　0    John         Sales         Tokyo
　1    William      Development   Osaka
　2    Richard      Sales         Tokyo
　3    Mary         Research      Nagoya
　4    Lisa         (NULL)        Nagoya
　5    James        Development   Osaka
```

<a id="sub_query"></a>
### サブクエリ

サブクエリはFROM句やWHERE句だけでなく、SQL文の様々な部分で指定できます。
また、サブクエリに対するいくつかの演算種別も提供しています。それらについて
説明します。

<a id="in_sub_query"></a>
#### IN

サブクエリの実行結果の中に、指定した値が含まれるかどうかを返します。

**構文**

|                                   |
|-----------------------------------|
| *式1* \[NOT\] IN ( *sub_query* ) |

**仕様**

- *式1*の値が、サブクエリの結果に含まれる場合はtrueを返します。
- サブクエリは1列のみを返すものでなければなりません。

例)
```example
// departmentsテーブルのid=1の部署に所属する従業員の情報をemployeesテーブルから表示します
SELECT * FROM employees
WHERE department IN(
  SELECT department FROM departments
  WHERE id = 1
);
結果：
  id   first_name   last_name   age     department    enrollment_period
  ----+------------+-----------+-------+-------------+-------------------
   1   William      Jones       59      Development   23.2
   5   James        Smith       43      Development   10.3
```

#### EXISTS

サブクエリの実行結果が存在するかどうかを返します。

**構文**

|      |
|-------|
| [NOT] EXISTS( *sub_query* ) |


**仕様**

- サブクエリの実行結果が存在するかどうかを確認します。実行結果が1件以上の場合はtrue、0件の場合はfalseを返します。

- 結果の型はBOOL型です。

例)
```example
// departmentsテーブルのid=1の部署に所属する従業員の情報をemployeesテーブルから表示します
SELECT * FROM employees
WHERE EXISTS(
   SELECT * FROM departments
   WHERE employees.department=departments.department AND departments.id=1
);
結果：
  id   first_name   last_name   age     department    enrollment_period
  ----+------------+-----------+-------+-------------+-------------------
   1   William      Jones       59      Development   23.2
   5   James        Smith       43      Development   10.3
```

#### スカラサブクエリ

ひとつの結果を返すサブクエリです。SELECT文の結果や、式などに使用できます。

例)
```example
SELECT id, first_name,
       (SELECT department FROM departments WHERE department_id=employees.department_id)
FROM employees;

結果：
  id  first_name  department
  ---+-----------+-------------
   0  John        Sales
   1  William     Development
   2  Richard     Sales
   3  Mary        (NULL)
   4  Lisa        Marketing
   5  James       Development
```

### プレースホルダ

プリペアードステートメントではSQL文にプレースホルダを記述できます。
プレースホルダはステートメント実行時に代入するパラメータの位置を示します。
パラメータの番号は1から始まります。

プレースホルダは他のデータベースとの互換性のため、幾つかの形式が使用できます。
ただし、いずれの形式で指定しても、パラメータの番号は既に割当てられている
パラメータ番号+1となります。

| 形式   | 説明   | 記述例 |
|--------|--------|--------|
| ?      | 標準のプレースホルダの形式 | ? |
| ?NNN   | NNNは数字を表す | ?56 |
| :AAAA  | AAAAは文字列を表す | :name |
| @AAAA  | AAAAは文字列を表す | @name |

なお、$から始まるプレースホルダは記述できません。

例)
```java
String sql = "SELECT * FROM users WHERE id > ? AND id != :exclude_id;";
PreparedStatement pstmt = con.prepareStatement(sql);
pstmt.setInt(1, 100);  // 1: ?
pstmt.setInt(2, 253);  // 2: :exclude_id
ResultSet rs = pstmt.executeQuery();
```

<a id="comment"></a>
## コメント

SQL文中にコメントを書くことができます。 書式は、--（ハイフンを2つ）の後ろに記述するか、/\* \*/で囲みます。
コメントの後ろには改行が必要です。

```example
SELECT * -- comment
FROM employees;

SELECT *
/*
  comment
*/
FROM employees;
```



<a id="hint"></a>
## ヒント機能

GridDBでは、実行計画を示すヒントをクエリに指定することで、SQL文を変えることなく実行計画を制御できます。




### エラーの扱い

以下の場合は構文エラーとなります。
- ヒント用のブロックコメントを複数記述した場合
- ヒントを記述できない位置にヒントを記述した場合
- ヒント句の記述に構文上の誤りがあった場合
- 同じテーブルに対して同じ分類のヒント句を重複して指定した場合

以下の場合はテーブル指定エラーとなります。
- ヒント句対象のテーブル指定に誤りがあった場合

【メモ】
- テーブル指定エラーが起こった場合、対象のヒント句を無視し、それ以外のヒント句を使ってクエリを実行します。
- 構文エラーとテーブル指定エラーが同時に起こった場合は構文エラーとなります。


<a id="label_metatables"></a>
# メタテーブル


## メタテーブルとは

GridDBの管理用のメタデータを参照することができるテーブル群です。

【メモ】
- メタテーブルは、参照のみ可能です。データの登録や削除はできません。
- [SELECT](#select)で参照する際、メタテーブル名は引用符"で囲んでください。
- 一般的なメタテーブルは接続ユーザの権限を問わず該当ユーザが接続しているデータベースの情報のみ取得できます。
- 但し、一部のメタテーブル(統計メタテーブル)はデータベース管理者権限を持つユーザは、接続以外の全てのデータベースの情報を取得可能とします。

【注意事項】
- メタテーブルのスキーマは将来のバージョンで変更される可能性があります。


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
| EXPIRATION_DIVISION_COUNT   | 期限解放分割数                                      | INTEGER  |
| PARTITION_TYPE             | パーティショニング種別                              | STRING  |
| PARTITION_COLUMN           | パーティショニングキー                              | STRING  |
| PARTITION_INTERVAL_VALUE    | 分割値（インターバル/インターバルハッシュの場合）   | STRING |
| PARTITION_INTERVAL_UNIT     | 分割単位（インターバル/インターバルハッシュの場合） | STRING  |
| PARTITION_DIVISION_COUNT    | 分割数（ハッシュの場合）                            | INTEGER |
| SUBPARTITION_TYPE          | パーティショニング種別<br>（インターバルハッシュの場合にHASH）| STRING  |
| SUBPARTITION_COLUMN        | パーティショニングキー<br>（インターバルハッシュの場合）| STRING  |
| SUBPARTITION_INTERVAL_VALUE | 分割値                                              | STRING |
| SUBPARTITION_INTERVAL_UNIT  | 分割単位                                            | STRING  |
| SUBPARTITION_DIVISION_COUNT | 分割数 <br>（インターバルハッシュの場合）             | INTEGER |
| EXPIRATION_TYPE            | 期限解放種別  <br>PARTITION                  | STRING  |


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
| INDEX_TYPE       | 索引種別 <br>TREE / SPATIAL     | STRING |
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
| CLUSTER_PARTITION_INDEX | クラスタパーティション番号            | INTEGER |
| CLUSTER_NODE_ADDRESS | ノードアドレス:ポート番号            | STRING |
| WORKER_INDEX | 処理スレッド番号            | INTEGER |

**仕様**

- ロウ1件がひとつのデータパーティションの情報を表します。
  - 例えば、分割数128のハッシュパーティショニングテーブルの場合、TABLE_NAMEにテーブル名を指定して検索するとロウが128個表示されます。
- 上記の列以外にも複数の列が表示されます。
- 下限値でソートする場合、対象のパーティショニングキーの型に合わせてキャストする必要があります。
- 各区間の配置先は、「CLUSTER_PARTITION_INDEX」、「CLUSTER_NODE_ADDRESS」、「WORKER_INDEX」によって把握することが可能です。

**例**

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

実行中のSQL(クエリまたはジョブ)に関する統計情報を取得できます。

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
| USER_NAME                  | ユーザ名                                            | STRING   |

【メモ】
 - データベース管理者は全てのデータベースを横断した情報を取得できます。

## 実行中イベント情報

実行中のイベントに関する統計情報を取得できます。

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
| WORKER_INDEX               | 処理スレッド番号                              | INTEGER  |
| CLUSTER_PARTITION_INDEX    | クラスタパーティション番号                          | INTEGER  |
| DATABASE_ID    | データベースID                          | LONG  |


## コネクション情報

接続中のコネクションに関する統計情報を取得できます。

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
| CREATION_TIME              | 接続時刻                                    | TIMESTAMP|
| DISPATCHING_EVENT_COUNT    | イベントハンドリングの要求を開始した総回数          | LONG     |
| SENDING_EVENT_COUNT        | イベント送信を開始した総回数                        | LONG     |
| DATABASE_NAME        | データベース名                        | LONG     |

SOCKET_TYPE(ソケット種別)は次の通り。

|値        |説明|
|-|-|
|SERVER    |サーバ間同士のTCP接続       |
|CLIENT    |クライアントとのTCP接続     |
|MULTICAST |マルチキャストソケット      |
|NULL      |接続中など現時点で不明の場合|

CREATION_TIME(接続時刻)は次の通り。
- SERVICE_TYPEがSQLの場合、CREATION_TIMEはJDBC接続を開始した時刻です。
- SERVICE_TYPEがTRANSACTIONの場合、CREATION_TIMEはNoSQL接続を開始した時刻です。


**例**

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

## データベース一覧

データベース名一覧と、それに対応したデータベースIDを取得できます。

**テーブル名**

\#databases

**スキーマ**

| 列名                       | 内容                                                | 型      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME              | データベース名                                      | STRING  |
| DATABASE_ID                  | データベースID                                            | INTEGER  |

 ## データベース統計情報

データベース単位で集計した統計情報を取得できます。

**テーブル名**

\#database_stats

**スキーマ**

| 列名 | 内容 | 型 |
|------|-----------------------------------------------------|---------|
| DATABASE_ID | データベースID | LONG |
| NODE_ADDRESS | ノードアドレス | STRING |
| NODE_PORT | ノードポート番号 | INTEGER |
| TRANSACTION_CONNECTION_COUNT | トランザクション接続数 | LONG |
| TRANSACTION_REQUEST_COUNT | トランザクションリクエスト数 | LONG |
| SQL_CONNECTION_COUNT | SQL接続数 | LONG |
| SQL_REQUEST_COUNT | SQLリクエスト数 | LONG |
| STORE_BLOCK_SIZE | ストア利用ブロックサイズ(※) | LONG |
| STORE_MEMORY_SIZE | ストア利用バッファサイズ(※) | LONG |
| STORE_SWAP_READ_SIZE | ストアスワップリードサイズ(※) | LONG |
| STORE_SWAP_WRITE_SIZE | ストアスワップライトサイズ(※) | LONG |
| SQL_WORK_MEMORY_SIZE | SQLワークメモリサイズ | LONG |
| SQL_STORE_USE_SIZE | SQLストア利用サイズ | LONG |
| SQL_STORE_SWAP_READ_SIZE | SQLストアスワップリードサイズ | LONG |
| SQL_STORE_SWAP_WRITE_SIZE | SQLストアスワップライトサイズ | LONG |
| SQL_TASK_COUNT | 実行中SQLタスク総数 | LONG |
| SQL_PENDING_JOB_COUNT | 送信抑制により停止中のSQLジョブ数 | LONG |
| SQL_SEND_MESSAGE_SIZE | SQL送信メッセージサイズ | LONG |

【メモ】
 - データベース管理者は全てのデータベースを横断した情報を取得できます。
 - 項目自体はサーバ全体で集計したものと同じになります。
 - ※の項目の正確な値を表示させるためには起動ファイルでの設定が必要となります。
 - 各統計値の項目の詳細および設定方法については『[GridDB 機能リファレンス](../md_reference_feature/md_reference_feature.md)』を参照してください。


# 予約語

GridDBのSQLでは、以下の単語が予約語として定義されています。

ABORT ACTION AFTER ALL ANALYZE AND AS ASC BEGIN BETWEEN BY CASE CAST COLLATE COLUMN COMMIT CONFLICT CREATE CROSS DATABASE DAY DELETE DESC DISTINCT DROP ELSE END ESCAPE EXCEPT EXCLUSIVE EXISTS EXPLAIN EXTRACT FALSE FOR FROM GLOB GRANT GROUP HASH HAVING HOUR IDENTIFIED IF IN INDEX INITIALLY INNER INSERT INSTEAD INTERSECT INTO IS ISNULL JOIN KEY LEFT LIKE LIMIT MATCH MILLISECOND MINUTE MONTH NATURAL NO NOT NOTNULL NULL OF OFFSET ON OR ORDER OUTER PARTITION PARTITIONS PASSWORD PLAN PRAGMA PRIMARY QUERY RAISE REGEXP RELEASE REPLACE RESTRICT REVOKE RIGHT ROLLBACK ROW SECOND SELECT SET TABLE THEN TIMESTAMPADD TIMESTAMPDIFF TO TRANSACTION TRUE UNION UPDATE USER USING VALUES VIEW VIRTUAL WHEN WHERE WITHOUT XOR YEAR
