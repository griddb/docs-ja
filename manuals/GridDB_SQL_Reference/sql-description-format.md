# SQL記述形式

ここでは、SQLインターフェースで使用できるSQLの記述形式について示します。

## 使用できる操作

SELECT文の他、CREATE TABLE等のDDL(Data Definition Language、データ定義言語)やINSERT/DELETE文などをサポートしています。詳細は[GridDBでサポートされるSQL文](sql-commands-supported.md#sql_commands_supported_by_griddb_ae)を参照して下さい。

---
## データ型

### <a id="data_types_used_in_data_storage"></a> データ格納に使用する型

SQLインターフェースでデータの格納に使用する型は次の通りです。この型名はテーブル作成時にカラム型として記述できます。

| データ型     | 内容詳細                                               |
|-------------|--------------------------------------------------------|
| BOOL型      | true/false                                             |
| BYTE型      | -2<sup>7</sup>から2<sup>7</sup>-1 (8ビット)の整数値     |
| SHORT型     | -2<sup>15</sup>から2<sup>15</sup>-1 (16ビット)の整数値  |
| INTEGER型   | -2<sup>31</sup>から2<sup>31</sup>-1 (32ビット)の整数値  |
| LONG型      | -2<sup>63</sup>から2<sup>63</sup>-1 (64ビット)の整数値  |
| FLOAT型     | 単精度型(32ビット) IEEE754で定められた浮動小数点数        |
| DOUBLE型    | 倍精度型(64ビット) IEEE754で定められた浮動小数点数        |
| TIMESTAMP型 | 日付と時刻の組  |
| STRING型    | Unicodeコードポイントを文字とする、任意個数の文字の列    |
| BLOB型      | 画像や音声などのバイナリデータのためのデータ型<br>入力したままの形式で保存されるラージオブジェクト<br>文字xあるいはXをつけて、X'23AB'のような16進表現もできる   |

また、テーブルにNULL値を格納することができます。NULL値に対して“IS NULL”などの演算子を使用すると、SQL仕様に沿った結果を返却します。

### テーブル作成時にカラム型として記述可能な表現

SQLインターフェースでは、テーブル作成時にカラム型として記述された型名について、[データ格納に使用する型](#data_types_used_in_data_storage)で列挙した型名と一致しなくても、ルールに従って解釈しデータの格納に使用する型を決定します。

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

NoSQLインターフェースのクライアントにおけるデータ型と同等の型をSQLインターフェイスで使用する場合は、以下のように記述してください。ただし、一部同等の型が存在せず、使用できないものがあります。

| NoSQLインターフェースのデータ型        | 同等の型となるSQLインターフェースのカラム型記述 |
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

### コンテナをテーブルとしてアクセスするときのデータ型と値の扱い

NoSQLインターフェースのクライアントで作成したコンテナを、SQLインターフェースでアクセスする場合のコンテナのカラム型および値の扱いを以下に示します。

| コンテナのカラム型  | SQLにマッピングされるデータ型    | 値             |
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

### SQLでサポートしていないデータ型の扱い

NoSQLインタフェースでサポートしているが、SQLインタフェースではサポートしていないデータ型は次の通りです。

- GEOMETRY型
- 配列型

これらのデータ型のデータに対して、SQLインタフェースでアクセスした場合の扱いについて説明します。

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
ユーザ、データベースの詳細は『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照ください。

## ネーミングの規則

ネーミングの規則は次の通りです。

-   データベース名・テーブル名・ビュー名・列名・索引名および一般ユーザ名は、1文字以上のASCII英数字ならびにアンダースコア「\_」、ハイフン「-」、ドット「.」、スラッシュ「/」、イコール「=」の列で構成されます。
-   テーブル名にはノードアフィニティ機能向けに「@」の文字も指定できます。

ノードアフィニティ機能、ネーミングの規則・制限についての詳細は、『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照ください。

<!--[!NOTE]-->
>#### メモ
>- 名前にASCII英数字とアンダースコア以外の文字を含む、または、先頭文字が数字のテーブルやカラムなどをSQL文に記述する場合は、引用符"で囲んでください。
>  ```
>  SELECT "column.a1" FROM "Table-5"
>  ```