# 仕様

本章では、GridDB JDBCドライバの仕様について示します。主に、ドライバのサポート範囲ならびにJDBC標準との相違点について説明します。 特記事項がなくJDBC標準に準拠しているAPIの仕様については、JDKのAPIリファレンスを参照してください。将来のバージョンでは、特に次の点が変更される可能性があります。

-   JDBC標準に準拠していない挙動
-   未サポートの機能のサポート状況
-   エラーメッセージ

　

## 共通事項

### サポートされるJDBCバージョン

JDBC4.1の一部機能に対応し、次の機能はサポートされません。

-   トランザクション制御
-   ストアドプロシージャ
-   バッチ実行


### エラー処理

#### 未サポート機能の使用

-   標準機能
    -   JDBC仕様に準拠したドライバがサポートすべき標準機能のうち、本ドライバにおいて現状サポートされていない機能を使用した場合、SQLFeatureNotSupportedExceptionが発生します。この挙動は、本来のSQLFeatureNotSupportedExceptionの仕様とは異なります。エラー名(後述)はJDBC_NOT_SUPPORTEDとなります。
-   オプション機能
    -   JDBC仕様においてオプション機能に位置付けられており、SQLFeatureNotSupportedExceptionが発生する可能性のある機能のうち、本ドライバにおいてサポートされていない機能を使用した場合、JDBC仕様通りSQLFeatureNotSupportedExceptionが発生します。エラー名はJDBC_OPTIONAL_FEATURE_NOT_SUPPORTEDとなります。

#### クローズ済みのオブジェクトに対するメソッド呼び出し

JDBC仕様の通り、Connectionオブジェクトなどclose()メソッドを持つオブジェクトに対し、isClosed()以外のメソッドを呼び出すと、SQLExceptionが発生します。 エラー名はJDBC_ALREADY_CLOSEDとなります。

#### 不正なnull引数

APIのメソッド引数として、nullが許容されないにも関わらず指定された場合、JDBC_EMPTY_PARAMETERエラーからなるSQLExceptionが発生します。JDBC仕様または本書で明示的にnullの受け入れを明記している引数以外は、nullを許容しません。

#### 複数のエラー原因がある場合

複数のエラー原因がある場合は、いずれかのエラーを検知した時点でアプリケーションに制御が戻ります。
特に、未サポート機能を使用しようとした場合のエラーは、他のエラーよりも先に検知します。
たとえば、クローズ済みのConnectionオブジェクトに対してストアドプロシージャを作成しようとした場合は、クローズされていることではなく、未サポートであることを示すエラーが返ります。

#### 例外の内容

ドライバよりスローされるチェック例外は、SQLExceptionもしくはSQLExceptionのサブクラスのインスタンスからなります。 例外の詳細を取得するには、次のメソッドを使用します。

- getErrorCode()
  - サーバ・クライアントのいずれかでGridDBが検知したエラーについて、エラー番号を返却します。
- getSQLState()
  - 少なくともドライバ内で検知したエラー(エラーコード：14xxxx)では、非nullの値を返却します。それ以外は未定義です。
- getMessage()
  - エラー番号とエラーの説明を組にした、エラーメッセージを返却します。書式は次のようになります。

    ``` example
    [(エラー番号):(エラー名)] (エラーの説明)
    ```

  - エラー一覧と対応しない番号のエラーが発生した場合、エラーメッセージは次のようになります。

    ``` example
    (エラーの詳細)
    ```

#### エラー一覧

ドライバ内部で検出される主なエラーの一覧は次の通りです。

| エラー番号 | エラーコード名                        | エラー説明の書式                                                                              |
|------------|-------------------------------------|-----------------------------------------------------------------------------------------------|
| (別記)     | JDBC_NOT_SUPPORTED                  | Currently not supported                                                                       |
| (別記)     | JDBC_OPTIONAL_FEATURE_NOT_SUPPORTED | Optional feature not supported                                                                |
| (別記)     | JDBC_EMPTY_PARAMETER                | The parameter (引数名) must not be null                                                       |
| (別記)     | JDBC_ALREADY_CLOSED                 | Already closed                                                                                |
| (別記)     | JDBC_COLUMN_INDEX_OUT_OF_RANGE      | Column index out of range                                                                     |
| (別記)     | JDBC_VALUE_TYPE_CONVERSION_FAILED   | Failed to convert value type                                                                  |
| (別記)     | JDBC_UNWRAPPING_NOT_SUPPORTED       | Unwrapping interface not supported                                                            |
| (別記)     | JDBC_ILLEGAL_PARAMETER              | Illegal value: (引数名)                                                                       |
| (別記)     | JDBC_UNSUPPORTED_PARAMETER_VALUE    | Unsupported (パラメータ名)                                                                    |
| (別記)     | JDBC_ILLEGAL_STATE                  | Protocol error occurred                                                                       |
| (別記)     | JDBC_INVALID_CURSOR_POSITION        | Invalid cursor position                                                                       |
| (別記)     | JDBC_STATEMENT_CATEGORY_UNMATCHED   | Writable query specified for read only request Read only query specified for writable request |
| (別記)     | JDBC_MESSAGE_CORRUPTED              | Protocol error                                                                                |

エラーの発生源となる元のエラーがある場合などは、上記のエラー説明の末尾に追加の詳細メッセージが追加されることがあります。



## API仕様詳細

### Connectionインターフェース

Connectionインターフェースの各メソッドについて説明します。 特に説明のない限り、Connectionがクローズされていない場合の説明のみを記載します。

#### トランザクション制御

トランザクション制御では、自動コミットモードのみのためコミット/ロールバックはサポートしません。
ただし、トランザクションを使用するアプリケーションにおいても疑似的に動作するよう、コミットやロールバックを実行された場合は要求を無視します。
SQLFeatureNotSupportedExceptionは発生しません。

トランザクション分離レベルは、TRANSACTION_READ_COMMITTEDのみサポートします。他のレベルは設定できません。

##### JDBC仕様との相違があるメソッド

| メソッド | 内容 | JDBC仕様との相違点 |
|-----|----|----|
| void commit()                           | コミットします。            | 自動コミットモードのみのため、コミット要求を無視します。|
| void rollback()                         | ロールバックします。        | 自動コミットモードのみのため、ロールバック要求を無視します。|
| void setAutoCommit(boolean autoCommit)  | コミットモードを設定します。 | 自動コミットモードのみのため、モードの設定はできません。autoCommitを無視して常にtrueを設定します。|


##### 一部未サポートのメソッド

| メソッド | 内容 | 一部未サポートの点 |
|-----|----|----|
| Statement createStatement(int resultSetType, int resultSetConcurrency) | ステートメントを作成します。 | resultSetTypeはResultSet.TYPE_FORWARD_ONLYのみ、resultSetConcurrencyはResultSet.CONCUR_READ_ONLYのみサポートします。それ以外の値を設定すると、SQLFeatureNotSupportedExceptionが発生します。 |
| Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) | ステートメントを作成します。 | resultSetTypeはResultSet.TYPE_FORWARD_ONLYのみ、resultSetConcurrencyはResultSet.CONCUR_READ_ONLYのみ、resultSetHoldabilityはResultSet.CLOSE_CURSORS_AT_COMMITのみサポートします。それ以外の値を設定すると、SQLFeatureNotSupportedExceptionが発生します。 |
| PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) | プリペアードステートメントを作成します。 | resultSetTypeはResultSet.TYPE_FORWARD_ONLYのみ、resultSetConcurrencyはResultSet.CONCUR_READ_ONLYのみサポートします。それ以外の値を設定すると、SQLFeatureNotSupportedExceptionが発生します。|
| PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) | プリペアードステートメントを作成します。| resultSetTypeはResultSet.TYPE_FORWARD_ONLYのみ、resultSetConcurrencyはResultSet.CONCUR_READ_ONLYのみ、resultSetHoldabilityはResultSet.CLOSE_CURSORS_AT_COMMITのみサポートします。それ以外の値を設定すると、SQLFeatureNotSupportedExceptionが発生します。|
| void setTransactionIsolation(int level) | トランザクション分離レベルを設定します。 | levelには、Connection.TRANSACTION_READ_COMMITTEDしか指定できません。それ以外の値を設定するとSQLExceptionが発生します。|


##### サポートしているメソッド

| メソッド | 内容 |
|---------|------|
| void close()                   | Connectionをクローズします。     |
| Statement createStatement()    | ステートメントを作成します。      |
| boolean getAutoCommit()        | コミットモードを取得します。      |
| DatabaseMetaData getMetaData() | DatabaseMetaDataを取得します。  |
| int getTransactionIsolation()  | トランザクション分離レベルを取得します。 |
| boolean isClosed()             | Connectionがクローズされているかを取得します。 |
| PreparedStatement prepareStatement(String sql) | プリペアードステートメントを作成します。 |



#### 属性の設定・取得

トランザクション制御のメソッド以外で、属性の設定や取得を行うメソッドについて説明します。

##### JDBC仕様との相違があるメソッド

| メソッド | 内容 | JDBC仕様との相違点 |
|-----|----|----|
| void setReadOnly(boolean readOnly) | Connectionオブジェクトの読み込み専用モードを設定します。 | readOnlyを無視して、常にfalseを設定します。|


##### 一部未サポートのメソッド

| メソッド | 内容 | 一部未サポートの点 |
|-----|----|----|
| void setHoldability(int holdability) | ResultSetオブジェクトの保持機能を設定します。 | holdabilityにはResultSet.CLOSE_CURSORS_AT_COMMITしか指定できません。それ以外の値を設定すると、SQLFeatureNotSupportedExceptionが発生します。 |


##### サポートしているメソッド

| メソッド | 内容 |
|---------|------|
| int getHoldability()         | ResultSetオブジェクトの保持機能を取得します。 |
| boolean isReadOnly()         | Connectionオブジェクトが読み込み専用モードかどうかを取得します。  |
| boolean isValid(int timeout) | 接続の状態を取得します。 |


#### 未サポートの機能

Connectionインターフェースの中で、未サポートのメソッド一覧を示します。実行すると、SQLFeatureNotSupportedExceptionが発生します。

- 標準機能
  - CallableStatement prepareCall(String sql)

- オプション機能
  - void abort(Executor executor)
  - Array createArrayOf(String typeName, Object[] elements)
  - Blob createBlob()
  - Clob createClob()
  - NClob createNClob()
  - SQLXML createSQLXML()
  - Struct createStruct(String typeName, Object[] attributes)
  - int getNetworkTimeout()
  - String getSchema()
  - Map<String,Class<?>> getTypeMap()
  - CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency)
  - CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability)
  - PreparedStatement prepareStatement(String sql, int autoGeneratedKeys)
  - PreparedStatement prepareStatement(String sql, int[] columnIndexes)
  - PreparedStatement prepareStatement(String sql, String[] columnNames)
  - void releaseSavepoint(Savepoint savepoint)
  - void rollback(Savepoint savepoint)
  - void setNetworkTimeout(Executor executor, int milliseconds)
  - void setSavepoint()
  - void setSchema(String schema)
  - void setTypeMap(Map<String,Class<?>> map)

　

### DatabaseMetaDataインターフェース

テーブルのメタデータを取得するDatabaseMetaDataインターフェースについて説明します。

#### ResultSetを返す属性

DatabaseMetaDataインターフェースで、実行結果としてResultSetを返すメソッドの中で、サポートしているメソッドは以下の通りです。
これら以外のResultSetを返すメソッドは未サポートです。

| メソッド | 内容 |
|--------|----|
| ResultSet getColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern) | テーブルのカラム情報を返します |
| ResultSet getIndexInfo(String catalog, String schema, String table, boolean unique, boolean approximate) | テーブルの索引情報を返します |
| ResultSet getPrimaryKeys(String catalog, String schema, String table) | テーブルのロウキーの情報を返します |
| ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types) | テーブルの一覧を返します |
| ResultSet getTableTypes() | テーブルの型を返します |
| ResultSet getTypeInfo() | カラムのデータ型一覧を返します |


上記のメソッドをそれぞれ説明します。


##### DatabaseMetaData.getColumns
```java
ResultSet getColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern)
```
- 指定されたテーブル名のパターンtableNamePatternに一致するテーブルのカラム情報を返します。パターンに指定されたワイルドカード「%」は0文字以上の文字、「\_」は任意の1文字に一致することを意味します。tableNamePatternにnullを指定した場合は全テーブルが対象になります。
- 他の絞り込み条件catalog, schemaPattern, columnNamePatternは無視されます。
- ビューのカラム情報は含まれません。
- 実行結果のResultSetが持つカラムは以下の通りです。

  | カラム名  　　      | 型     | 値                                   |
  |--------------------|--------|--------------------------------------|
  | TABLE_CAT          | String | null                                 |
  | TABLE_SCHEM        | String | null                                 |
  | TABLE_NAME         | String | テーブル名                            |
  | COLUMN_NAME        | String | カラム名                              |
  | DATA_TYPE          | int    | カラムのデータ型の値(下記の表を参照)    |
  | TYPE_NAME          | String | カラムのデータ型の名前(下記の表を参照)  |
  | COLUMN_SIZE        | int    | 131072                               |
  | BUFFER_LENGTH      | int    | 2000000000                           |
  | DECIMAL_DIGITS     | int    | 10                                   |
  | NUM_PREC_RADIX     | int    | 10                                   |
  | NULLABLE           | int    | PRIMARY KEYまたはNOT NULL制約があるカラムは 0 (定数DatabaseMetaData.columnNoNullsの値)<br>それ以外は 1 (定数DatabaseMetaData.columnNullableの値) |
  | REMARKS            | String | null                                 |
  | COLUMN_DEF         | String | null                                 |
  | SQL_DATA_TYPE      | int    | 0                                    |
  | SQL_DATETIME_SUB   | int    | 0                                    |
  | CHAR_OCTET_LENGTH  | int    | 2000000000                           |
  | ORDINAL_POSITION   | int    | カラムの番号(1からの連番)              |
  | IS_NULLABLE        | String | NOT NULL制約。PRIMARY KEYまたはNOT NULL制約があるカラムは'NO'<br>それ以外は'YES' |
  | SCOPE_CATALOG      | String | null                                 |
  | SCOPE_SCHEMA       | String | null                                 |
  | SCOPE_TABLE        | String | null                                 |
  | SOURCE_DATA_TYPE   | short  | 0                                    |
  | IS_AUTOINCREMENT   | String | 'NO'                                 |
  | IS_GENERATEDCOLUMN | String | 'NO'                                 |

- 該当カラムのデータ型に応じて、次のTYPE_NAMEとDATA_TYPEの値の組合せを返します。

  | カラムのデータ型    | TYPE_NAMEの値        | DATA_TYPEの値        |
  |--------------------|---------------------|----------------------|
  | BOOL型             | 'BOOL'              | -7 (Types.BIT)       |
  | STRING型　         | 'STRING'            | 12 (Types.VARCHAR)   |
  | BYTE型             | 'BYTE'              | -6 (Types.TINYINT)   |
  | SHORT型            | 'SHORT'             | 5 (Types.SMALLINT)   |
  | INTEGER型          | 'INTEGER'           | 4 (Types.INTEGER)    |
  | LONG型             | 'LONG'              | -5 (Types.BIGINT)    |
  | FLOAT型            | 'FLOAT'             | 6 (Types.FLOAT)      |
  | DOUBLE型           | 'DOUBLE'            | 8 (Types.DOUBLE)     |
  | TIMESTAMP型        | 'TIMESTAMP'         | 93 (Types.TIMESTAMP) |
  | BLOB型             | 'BLOB'              | 2004 (Types.BLOB)    |
  | GEOMETRY型         | 'GEOMETRY'          | 1111 (Types.OTHER)   |
  | BOOL型の配列型      | 'BOOL_ARRAY'        | 1111 (Types.OTHER)   |
  | STRING型の配列型    | 'STRING_ARRAY'      | 1111 (Types.OTHER)   |
  | BYTE型の配列型      | 'BYTE_ARRAY'        | 1111 (Types.OTHER)   |
  | SHORT型の配列型     | 'SHORT_ARRAY'       | 1111 (Types.OTHER)   |
  | INTEGER型の配列型   | 'INTEGER_ARRAY'     | 1111 (Types.OTHER)   |
  | LONG型の配列型      | 'LONG_ARRAY'        | 1111 (Types.OTHER)   |
  | FLOAT型の配列型     | 'FLOAT_ARRAY'       | 1111 (Types.OTHER)   |
  | DOUBLE型の配列型    | 'DOUBLE_ARRAY'      | 1111 (Types.OTHER)   |
  | TIMESTAMP型の配列型 | 'TIMESTAMP_ARRAY'   | 1111 (Types.OTHER)   |

- GEOMETRY型と配列型については、NoSQLインターフェースで作成された、これらのデータ型を持つテーブルの情報を取得した場合に値が返ります。JDBCでは、これらのデータ型を持つテーブルは作成できません。

　

##### DatabaseMetaData.getIndexInfo

```java
ResultSet getIndexInfo(String catalog, String schema, String table, boolean unique, boolean approximate)
```

- 指定されたテーブル名tableに一致するテーブルの索引情報を返します。指定された名前のテーブルが存在しない場合は、実行結果のResultSetは空が返ります。
- uniqueにはfalseを指定してください。uniqueがfalse以外の場合、実行結果のResultSetは空が返ります。
- 他の絞り込み条件catalog、schemaと、パラメータapproximateは無視されます。
- 実行結果のResultSetが持つカラムは以下の通りです。

  | カラム名          | 型      | 値               |
  |------------------|---------|------------------|
  | TABLE_CAT        | String  | null             |
  | TABLE_SCHEM      | String  | null             |
  | TABLE_NAME       | String  | テーブル名        |
  | NON_UNIQUE       | boolean | true             |
  | INDEX_QUALIFIER  | String  | null             |
  | INDEX_NAME       | String  | 索引名            |
  | TYPE             | short   | 2(ハッシュ索引を表す定数DatabaseMetaData.tableIndexHashedの値) または 3(ハッシュ以外の索引を表す定数DatabaseMetaData.tableIndexOtherの値) |
  | ORDINAL_POSITION | short   | 1から開始         |
  | COLUMN_NAME      | String  | カラム名          |
  | ASC_OR_DESC      | String  | null             |
  | CARDINALITY      | long    | 0                |
  | PAGES            | long    | 0                |
  | FILTER_CONDITION | String  | null             |

　

##### DatabaseMetaData.getPrimaryKeys

```java
ResultSet getPrimaryKeys(String catalog, String schema, String table)
```
- 指定されたテーブル名tableに一致するテーブルのロウキー情報を返します。指定された名前のテーブルが存在しない場合は、実行結果のResultSetは空が返ります。ただし、tableにnullを指定した場合は、ロウキーが設定されている全テーブルの情報が返ります。
- 他の絞り込み条件catalog, schemaは無視されます。
- 実行結果のResultSetが持つカラムは以下の通りです。

  | カラム名          | 型      | 値           |
  |------------------|---------|--------------|
  | TABLE_CAT        | String  | null         |
  | TABLE_SCHEM      | String  | null         |
  | TABLE_NAME       | String  | テーブル名    |
  | COLUMN_NAME      | String  | カラム名      |
  | KEY_SEQ          | short   | 1            |
  | PK_NAME          | String  | null         |

　

##### DatabaseMetaData.getTables

```java
ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types)
```
- 指定されたテーブル名のパターンtableNamePatternに一致するテーブルの情報を返します。パターンに指定されたワイルドカード「%」は0文字以上の文字、「\_」は任意の1文字に一致することを意味します。tableNamePatternにnullを指定した場合は全テーブルが対象になります。
- typesにはnullまたは文字列の配列を指定します。文字列要素は"TABLE"および"VIEW"が指定できます。typesに"TABLE"または"VIEW"と一致する要素が存在しなければ常に空の結果を返します。types内の文字列要素の大文字小文字表記の違いは無視されます。(typesは、テーブルの種別であるコレクション、時系列コンテナを表す値ではありません。)
- 他の絞り込み条件catalog, schemaPatternは無視されます。
- 実行結果のResultSetが持つカラムは以下の通りです。

  | カラム名                    | 型      | 値               |
  |----------------------------|---------|------------------|
  | TABLE_CAT                  | String  | null             |
  | TABLE_SCHEM                | String  | null             |
  | TABLE_NAME                 | String  | テーブル名        |
  | TABLE_TYPE                 | String  | 'TABLE'または'VIEW'          |
  | REMARKS                    | String  | null             |
  | TYPE_CAT                   | String  | null             |
  | TYPE_SCHEM                 | String  | null             |
  | TYPE_NAME                  | String  | null             |
  | SELF_REFERENCING_COL_NAME  | String  | null             |
  | REF_GENERATION             | String  | null             |

　

##### DatabaseMetaData.getTableTypes

```java
ResultSet getTableTypes()
```
- テーブルのタイプを返します。結果は、カラム「TABLE_TYPE」に値'TABLE'または'VIEW'が格納された1件のみが返ります。
- 実行結果のResultSetが持つカラムは以下の通りです。

  | カラム名          | 型      | 値          |
  |------------------|---------|-------------|
  | TABLE_TYPE       | String  | 'TABLE'または'VIEW'     |

　

##### DatabaseMetaData.getTypeInfo()

```java
ResultSet getTypeInfo()
```
- カラムのデータ型一覧を返します。
- すべての型で共通の情報、型別の情報は以下の通りです。

  | カラム名            | 型      | 値                |
  |--------------------|---------|-------------------------|
  | TYPE_NAME          | String  | データ型の名前(下記の表を参照)     |
  | DATA_TYPE          | int     | データ型の値(下記の表を参照)     |
  | PRECISION          | int     | 0         |
  | LITERAL_PREFIX     | String  | null      |
  | LITERAL_SUFFIX     | String  | null      |
  | CREATE_PARAMS      | String  | null      |
  | NULLABLE           | short   | 1 (このデータ型でNULL値が許可されることを表す定数DatabaseMetaData.typeNullableの値)    |
  | CASE_SENSITIVE     | boolean | true      |
  | SEARCHABLE         | short   | 3 (このデータ型をWHERE節で使用できることを表す定数DatabaseMetaData.typeSearchableの値)    |
  | UNSIGNED_ATTRIBUTE | boolean | false     |
  | FIXED_PREC_SCALE   | boolean | false     |
  | AUTO_INCREMENT     | boolean | false     |
  | LOCAL_TYPE_NAME    | String  | null      |
  | MINIMUM_SCALE      | short   | 0         |
  | MAXIMUM_SCALE      | short   | 0         |
  | SQL_DATA_TYPE      | int     | 0         |
  | SQL_DATETIME_SUB   | int     | 0         |
  | NUM_PREC_RADIX     | int     | 10        |


- カラムTYPE_NAME、DATA_TYPEは、以下の組合せの値が全て返ります。

  | TYPE_NAMEの値       | DATA_TYPEの値         |
  |---------------------|----------------------|
  | 'BOOL'              | -7 (Types.BIT)       |
  | 'STRING'            | 12 (Types.VARCHAR)   |
  | 'BYTE'              | -6 (Types.TINYINT)   |
  | 'SHORT'             | 5 (Types.SMALLINT)   |
  | 'INTEGER'           | 4 (Types.INTEGER)    |
  | 'LONG'              | -5 (Types.BIGINT)    |
  | 'FLOAT'             | 6 (Types.FLOAT)      |
  | 'DOUBLE'            | 8 (Types.DOUBLE)     |
  | 'TIMESTAMP'         | 93 (Types.TIMESTAMP) |
  | 'BLOB'              | 2004 (Types.BLOB)    |
  | 'UNKNOWN'           | 0 (Types.NULL)       |

　

#### 単純値を返すメソッド

DatabaseMetaDataインターフェースのメソッドの中で、実行結果としてint型やString型などの単純値を返すメソッドについて、実行結果の一覧を示します。

| メソッド                                                | 結果                                                                               |
|---------------------------------------------------------|------------------------------------------------------------------------------------|
| allProceduresAreCallable()                              | false                                                                              |
| allTablesAreSelectable()                                | true                                                                               |
| autoCommitFailureClosesAllResultSets()                  | false                                                                              |
| dataDefinitionCausesTransactionCommit()                 | false                                                                              |
| dataDefinitionIgnoredInTransactions()                   | true                                                                               |
| deletesAreDetected(type)                                | false                                                                              |
| doesMaxRowSizeIncludeBlobs()                            | false                                                                              |
| generatedKeyAlwaysReturned()                            | false                                                                              |
| getCatalogSeparator()                                   | "."                                                                                |
| getCatalogTerm()                                        | "catalog"                                                                          |
| getDefaultTransactionIsolation()                        | TRANSACTION_READ_COMMITTED                                                |
| getExtraNameCharacters()                                | . - / = (順不同)                                                                   |
| getIdentifierQuoteString()                              | "                                                                                  |
| getMaxBinaryLiteralLength()                             | 0                                                                                  |
| getMaxCatalogNameLength()                               | 0                                                                                  |
| getMaxCharLiteralLength()                               | 0                                                                                  |
| getMaxColumnNameLength()                                | 0                                                                                  |
| getMaxColumnsInGroupBy()                                | 0                                                                                  |
| getMaxColumnsInIndex()                                  | 0                                                                                  |
| getMaxColumnsInOrderBy()                                | 0                                                                                  |
| getMaxColumnsInSelect()                                 | 0                                                                                  |
| getMaxColumnsInTable()                                  | 0                                                                                  |
| getMaxConnections()                                     | 0                                                                                  |
| getMaxCursorNameLength()                                | 0                                                                                  |
| getMaxIndexLength()                                     | 0                                                                                  |
| getMaxSchemaNameLength()                                | 0                                                                                  |
| getMaxProcedureNameLength()                             | 0                                                                                  |
| getMaxRowSize()                                         | 0                                                                                  |
| getMaxStatementLength()                                 | 0                                                                                  |
| getMaxStatements()                                      | 0                                                                                  |
| getMaxTableNameLength()                                 | 0                                                                                  |
| getMaxTablesInSelect()                                  | 0                                                                                  |
| getMaxUserNameLength()                                  | 0                                                                                  |
| getProcedureTerm()                                      | "procedure"                                                                        |
| getResultSetHoldability()                               | CLOSE_CURSORS_AT_COMMIT                                                    |
| getRowIdLifetime()                                      | true                                                                               |
| getSchemaTerm()                                         | "schema"                                                                           |
| getSearchStringEscape()                                 | "|                                                                                 |
| getSQLKeywords()                                        | ""                                                                                 |
| getSQLStateType()                                       | sqlStateSQL99                                                                      |
| getStringFunctions()                                    | ""                                                                                 |
| getSystemFunctions()                                    | ""                                                                                 |
| getURL()                                                | null                                                                               |
| getUserName()                                           | (ユーザ名)                                                                         |
| insertsAreDetected(type)                                | false                                                                              |
| isCatalogAtStart()                                      | true                                                                               |
| isReadOnly()                                            | false                                                                              |
| locatorsUpdateCopy()                                    | false                                                                              |
| nullPlusNonNullIsNull()                                 | true                                                                               |
| nullsAreSortedAtEnd()                                   | false                                                                              |
| nullsAreSortedAtStart()                                 | false                                                                              |
| nullsAreSortedHigh()                                    | true                                                                               |
| nullsAreSortedLow()                                     | false                                                                              |
| othersDeletesAreVisible(type)                           | false                                                                              |
| othersInsertsAreVisible(type)                           | false                                                                              |
| othersUpdatesAreVisible(type)                           | false                                                                              |
| ownDeletesAreVisible(type)                              | false                                                                              |
| ownInsertsAreVisible(type)                              | false                                                                              |
| ownUpdatesAreVisible(type)                              | false                                                                              |
| storesLowerCaseIdentifiers()                            | false                                                                              |
| storesLowerCaseQuotedIdentifiers()                      | false                                                                              |
| storesMixedCaseIdentifiers()                            | true                                                                               |
| storesMixedCaseQuotedIdentifiers()                      | false                                                                              |
| storesUpperCaseIdentifiers()                            | false                                                                              |
| storesUpperCaseQuotedIdentifiers()                      | false                                                                              |
| supportsAlterTableWithAddColumn()                       | false                                                                              |
| supportsAlterTableWithDropColumn()                      | false                                                                              |
| supportsANSI92EntryLevelSQL()                           | false                                                                              |
| supportsANSI92FullSQL()                                 | false                                                                              |
| supportsANSI92IntermediateSQL()                         | false                                                                              |
| supportsBatchUpdates()                                  | false                                                                              |
| supportsCatalogsInDataManipulation()                    | false                                                                              |
| supportsCatalogsInIndexDefinitions()                    | false                                                                              |
| supportsCatalogsInPrivilegeDefinitions()                | false                                                                              |
| supportsCatalogsInProcedureCalls()                      | false                                                                              |
| supportsCatalogsInTableDefinitions()                    | false                                                                              |
| supportsColumnAliasing()                                | true                                                                               |
| supportsConvert()                                       | false                                                                              |
| supportsConvert(fromType, toType)                       | false                                                                              |
| supportsCoreSQLGrammar()                                | true                                                                               |
| supportsCorrelatedSubqueries()                          | true                                                                               |
| supportsDataDefinitionAndDataManipulationTransactions() | false                                                                              |
| supportsDataManipulationTransactionsOnly()              | false                                                                              |
| supportsDifferentTableCorrelationNames()                | false                                                                              |
| supportsExpressionsInOrderBy()                          | true                                                                               |
| supportsExtendedSQLGrammar()                            | false                                                                              |
| supportsFullOuterJoins()                                | false                                                                              |
| supportsGetGeneratedKeys()                              | false                                                                              |
| supportsGroupBy()                                       | true                                                                               |
| supportsGroupByBeyondSelect()                           | true                                                                               |
| supportsGroupByUnrelated()                              | true                                                                               |
| supportsIntegrityEnhancementFacility()                  | false                                                                              |
| supportsLikeEscapeClause()                              | true                                                                               |
| supportsLimitedOuterJoins()                             | true                                                                               |
| supportsMinimumSQLGrammar()                             | true                                                                               |
| supportsMixedCaseIdentifiers()                          | false                                                                              |
| supportsMixedCaseQuotedIdentifiers()                    | true                                                                               |
| supportsMultipleOpenResults()                           | false                                                                              |
| supportsMultipleResultSets()                            | false                                                                              |
| supportsMultipleTransactions()                          | false                                                                              |
| supportsNamedParameters()                               | false                                                                              |
| supportsNonNullableColumns()                            | true                                                                               |
| supportsOpenCursorsAcrossCommit()                       | false                                                                              |
| supportsOpenCursorsAcrossRollback()                     | false                                                                              |
| supportsOpenStatementsAcrossCommit()                    | false                                                                              |
| supportsOpenStatementsAcrossRollback()                  | false                                                                              |
| supportsOrderByUnrelated()                              | true                                                                               |
| supportsOuterJoins()                                    | true                                                                               |
| supportsPositionedDelete()                              | false                                                                              |
| supportsPositionedUpdate()                              | false                                                                              |
| supportsResultSetConcurrency(type, concurrency)         | typeがTYPE_FORWARD_ONLY、concurrencyがCONCUR_READ_ONLYの場合のみ |
| supportsResultSetHoldability(holdability)               | CLOSE_CURSORS_AT_COMMITの場合のみ                                          |
| supportsResultSetType()                                 | TYPE_FORWARD_ONLYの場合のみ                                               |
| supportsSavepoints()                                    | false                                                                              |
| supportsSchemasInDataManipulation()                     | false                                                                              |
| supportsSchemasInIndexDefinitions()                     | false                                                                              |
| supportsSchemasInPrivilegeDefinitions()                 | false                                                                              |
| supportsSchemasInProcedureCalls()                       | false                                                                              |
| supportsSchemasInTableDefinitions()                     | false                                                                              |
| supportsSelectForUpdate()                               | false                                                                              |
| supportsStatementPooling()                              | false                                                                              |
| supportsStoredFunctionsUsingCallSyntax()                | false                                                                              |
| supportsStoredProcedures()                              | false                                                                              |
| supportsSubqueriesInComparisons()                       | false                                                                              |
| supportsSubqueriesInExists()                            | true                                                                               |
| supportsSubqueriesInIns()                               | true                                                                               |
| supportsSubqueriesInQuantifieds()                       | false                                                                              |
| supportsTableCorrelationNames()                         | false                                                                              |
| supportsTransactionIsolationLevel(level)                | TRANSACTION_READ_COMMITTEDの場合のみ                                      |
| supportsTransactions()                                  | true                                                                               |
| supportsUnion()                                         | true                                                                               |
| supportsUnionAll()                                      | true                                                                               |
| updatesAreDetected(type)                                | false                                                                              |
| usesLocalFilePerTable()                                 | false                                                                              |
| usesLocalFiles()                                        | false                                                                              |


#### 未サポートのメソッド

DatabaseMetaDataインターフェースのメソッドの中で、未サポートのメソッド一覧を示します。
実行するとSQLFeatureNotSupportedExceptionは発生せず、以下の結果が返ります。

| メソッド                           | 結果          |
|------------------------------------|---------------|
| ResultSet getAttributes(String catalog, String schemaPattern, String typeNamePattern, String attributeNamePattern) | 空のResultSet |
| ResultSet getBestRowIdentifier(String catalog, String schema, String table, int scope, boolean nullable) | 空のResultSet |
| ResultSet getCatalogs()                        | 空のResultSet |
| ResultSet getClientInfoProperties()            | 空のResultSet |
| ResultSet getColumnPrivileges(String catalog, String schema, String table, String columnNamePattern) | 空のResultSet |
| ResultSet getCrossReference(String parentCatalog, String parentSchema, String parentTable, String foreignCatalog, String foreignSchema, String foreignTable) | 空のResultSet |
| ResultSet getExportedKeys(String catalog, String schema, String table)  | 空のResultSet |
| ResultSet getFunctionColumns(String catalog, String schemaPattern, String functionNamePattern, String columnNamePattern) | 空のResultSet |
| ResultSet getFunctions(String catalog, String schemaPattern, String functionNamePattern)   | 空のResultSet |
| ResultSet getImportedKeys(String catalog, String schema, String table)  | 空のResultSet |
| ResultSet getProcedureColumns(String catalog, String schemaPattern, String procedureNamePattern, String columnNamePattern)  | 空のResultSet |
| ResultSet getProcedures(String catalog, String schemaPattern, String procedureNamePattern)  | 空のResultSet |
| ResultSet getPseudoColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern) | 空のResultSet |
| ResultSet getSchemas()                       | 空のResultSet |
| ResultSet getSchemas(String catalog, String schemaPattern)  | 空のResultSet |
| ResultSet getSuperTables(String catalog, String schemaPattern, String tableNamePattern)  | 空のResultSet |
| ResultSet getSuperTypes(String catalog, String schemaPattern, String typeNamePattern)    | 空のResultSet |
| ResultSet getTablePrivileges(String catalog, String schemaPattern, String tableNamePattern)  | 空のResultSet |
| ResultSet getUDTs(String catalog, String schemaPattern, String typeNamePattern, int[] types) | 空のResultSet |
| ResultSet getVersionColumns(String catalog, String schema, String table) | 空のResultSet |

　

### Statementインターフェース

#### フェッチサイズの設定・取得

指定された値のチェックのみ行います。

値のチェックでは、このStatementのgetMaxRows()で得られるロウ数より超えないこともチェックします。 この値に関する制限は、JDBC4.0より前のJDBC仕様でのみ明記されていました。 ただし、以前のJDBC仕様とは異なり、getMaxRows()の結果がデフォルト値0に設定されている場合を除きます。

#### フェッチ方向の設定・取得

フェッチ方向の設定はFETCH_FORWARDのみをサポートします。FETCH_FORWARD以外が指定された場合、SQLExceptionが発生します。

#### 未サポートの機能

-   バッチ処理
    -   バッチ処理はサポートしません。以下の機能を使用しようとすると、未サポートの標準機能を使用した場合と同一のエラーが発生します。
        -   addBatch(sql)
        -   clearBatch()
        -   executeBatch()
-   標準機能
    -   以下のメソッドの呼び出しは無視されます。JDBC仕様とは異なります。
        -   setEscapeProcessing(enable)
-   オプション機能
    -   以下のメソッドを呼び出すとSQLFeatureNotSupportedExceptionが発生します。
        -   closeOnCompletion()
        -   execute(sql, autoGeneratedKeys)
        -   execute(sql, columnIndexes)
        -   execute(sql, columnNames)
        -   executeUpdate(sql, autoGeneratedKeys)
        -   executeUpdate(sql, columnIndexes)
        -   executeUpdate(sql, columnNames)
        -   getGeneratedKeys()
        -   getMoreResults(current)
        -   isCloseOnCompletion()

　

### PreparedStatementインターフェース

#### パラメータの設定・取得

以下のメソッドをサポートします。設定されていないパラメータがある状態でexecuteQueryなどクエリ実行APIを呼び出すと、SQLExceptionが発生します。

-   clearParameters()
-   getMetaData()
-   getParameterMetaData()
-   setBinaryStream(int parameterIndex, InputStream x)
    -    setBinaryStream(オーバーロード含む)では、クライアント側で入力データの数倍のメモリが必要になる可能性があります。実行時にメモリ不足になった場合はJVM起動時のヒープメモリサイズを調整してください。
-   setBinaryStream(int parameterIndex, InputStream x, int length)
-   setBinaryStream(int parameterIndex, InputStream x, long length)
-   setBlob(int parameterIndex, Blob x)
    -    setBlob(オーバーロード含む)では、クライアント側で入力データの数倍のメモリが必要になる可能性があります。実行時にメモリ不足になった場合はJVM起動時のヒープメモリサイズを調整してください。
-   setBlob(int parameterIndex, InputStream inputStream)
-   setBlob(int parameterIndex, InputStream inputStream, long length)
-   setBoolean(int parameterIndex, boolean x)
-   setByte(int parameterIndex, byte x)
-   setDate(int parameterIndex, Date x)
-   setDouble(int parameterIndex, double x)
-   setFloat(int parameterIndex, float x)
-   setInt(int parameterIndex, int x)
-   setLong(int parameterIndex, long x)
-   setObject(int parameterIndex, Object x)
    -   TIMESTAMP型のパラメータに設定する値としては、java.util.Dateのサブクラスのオブジェクトを受け入れます。
-   setShort(int parameterIndex, short x)
-   setString(int parameterIndex, String x)
-   setTime(int parameterIndex, Time x)
-   setTimestamp(int parameterIndex, Timestamp x)

#### SQLの実行

以下のメソッドをサポートします。

-   execute()
-   executeQuery()
-   executeUpdate()

#### 未サポートの機能

-   標準機能
    -   以下のメソッドを呼び出すとSQLFeatureNotSupportedExceptionが発生します。この挙動はJDBC仕様とは異なります。
        -   addBatch()
        -   setBigDecimal(int parameterIndex, BigDecimal x)
        -   setDate(int parameterIndex, Date x, Calendar cal)
        -   setTime(int parameterIndex, Time x, Calendar cal)
        -   setTimestamp(int parameterIndex, Timestamp x, Calendar cal)
-   オプション機能
    -   以下のメソッドを呼び出すとSQLFeatureNotSupportedExceptionが発生します。引数を省略しているものは、全てのオーバーロードが未サポートです。
        -   setArray
        -   setAsciiStream
        -   setBytes
        -   setCharacterStream
        -   setClob
        -   setNCharacterStream
        -   setNClob
        -   setNString
        -   setNull
        -   setObject(int parameterIndex, Object x, int targetSqlType)
        -   setObject(int parameterIndex, Object x, int targetSqlType, int scaleOrLength)
        -   setRef
        -   setRowId
        -   setSQLXML
        -   setUnicodeStream
        -   setURL

　

### ParameterMetaDataインターフェース

PreparedStatementのメタデータを取得するParameterMetaDataインターフェースについて説明します。

JDBC仕様の全てのメソッドをサポートしますが、以下のメソッドは引数paramの値によらず常に固定の値を返します。

| メソッド                                 | 結果        |
|-----------------------------------------|-------------|
| int getParameterType(int param)         | Types.OTHER |
| String getParameterTypeName(int param)  | "UNKNOWN"   |
| int getPrecision(int param)             | 0           |
| int getScale(int param)                 | 0           |
| boolean isSigned(int param)             | false       |

　

### ResultSetインターフェース

#### フェッチサイズの設定・取得

指定された値のチェックのみ行い、設定の変更は実際のフェッチ処理には影響しません。 値のチェックでは、対象のResultSetの生成元のStatementのgetMaxRows()で得られるロウ数より超えないこともチェックします。 この制限は、JDBC4.0より前のJDBC仕様でのみ明記されていました。ただし、以前のJDBC仕様とは異なり、getMaxRows()の結果がデフォルト値0に設定されている場合を除きます。実際のフェッチ処理には影響しませんが、変更した設定値を取得できます。

#### フェッチ方向の設定・取得

フェッチ方向の設定はFETCH_FORWARDのみをサポートします。FETCH_FORWARD以外が指定された場合、 SQLExceptionが発生します。この挙動はJDBC仕様とは異なります。

#### カーソル情報の取得

カーソルに関する以下のメソッドをサポートします。
-   isAfterLast()
-   isBeforeFirst()
-   isFirst()
-   isLast()
-   next()

フェッチ方向はFETCH_FORWARDのみをサポートしているため、次のメソッドを呼び出すとFETCH_FORWARDタイプのResultSetに対する呼び出しを原因とするSQLExceptionが発生します。
-   absolute(row)
-   afterLast()
-   beforeFirst()
-   first()
-   last()
-   previous()
-   relative(rows)

#### 警告の管理

警告は記録されないため、警告を管理するメソッドの挙動は次のようになります。

| メソッド        | 挙動                 |
|-----------------|----------------------|
| getWarnings()   | nullを返却           |
| clearWarnings() | 警告はクリアされない |

#### 固定値を返す属性

ResultSetがオープンされている間、常に固定の値を返すメソッドのサポート状況は次の通りです。

| メソッド         | 結果                             |
|------------------|----------------------------------|
| getCursorName()  | nullを返却                       |
| getType()        | TYPE_FORWARD_ONLYを返却           |
| getConcurrency() | CONCUR_READ_ONLYを返却            |
| getMetaData()    | (JDBC準拠)                       |
| getStatement()   | (JDBC準拠)                       |

#### 型変換

指定のカラムの値を取得する際、ResultSetが保持する型と求める型とが異なる場合は、次の組み合わせに限り、型変換を試みます。

| 変換先のJava型 | BOOL | INTEGRAL ※1 | FLOATING ※2 | TIMESTAMP | STRING | BLOB |
|----------------|------|-------------|-------------|-----------|--------|------|
| boolean        | ○    | ○ ※3        |             |           | ○ ※4   |      |
| byte           | ○    | ○           | ○           |           | ○      |      |
| short          | ○    | ○           | ○           |           | ○      |      |
| int            | ○    | ○           | ○           |           | ○      |      |
| long           | ○    | ○           | ○           |           | ○      |      |
| float          |      | ○           | ○           |           | ○      |      |
| double         |      | ○           | ○           |           | ○      |      |
| byte\[\]       | ○    | ○           | ○           |           | ○      |      |
| java.sql.Date  |      |             |             | ○         | ○ ※5  |      |
| Time           |      |             |             | ○         | ○ ※5  |      |
| Timestamp      |      |             |             | ○         | ○ ※5  |      |
| String         | ○    | ○           | ○           | ○ ※6     | ○      | ○ ※7 |
| Blob           |      |             |             |           | ○ ※7   | ○    |
| Object         | ○    | ○           | ○           | ○         | ○      | ○    |

-   (※1). INTEGRAL: BYTE/SHORT/INTEGER/LONGのいずれかを表します。
-   (※2). FLOATING: FLOAT/DOUBLEのいずれかを表します。
-   (※3). 変換元数値が0ならばfalseに、0以外ならばtrueに変換します。
-   (※4). 変換元文字列が'false'ならばfalseに、'true'ならばtrueに変換します。ASCIIの大文字小文字は同一視します。それ以外は変換できずエラーとなります。
-   (※5). 以下のルールで文字列表現の時刻を変換します。
    -   表現をjava.text.SimpleDateFormatと同様のパターン文字列で表したものは以下のとおりです。ただしタイムゾーンを除きます。
        -   yyyy-MM-dd'T'HH:mm:ss.SSS
        -   yyyy-MM-dd'T'HH:mm:ss
        -   yyyy-MM-dd HH:mm:ss.SSS
        -   yyyy-MM-dd HH:mm:ss
        -   yyyy-MM-dd
        -   HH:mm:ss.SSS
        -   HH:mm:ss
    -   タイムゾーンについては、文字列に含まれるものが最優先で利用されます。次に、ResultSet#getTimeStamp()など取得APIの引数java.util.Calendarにおいてタイムゾーン指定があった場合はその内容を参照します。最後に接続時に指定したタイムゾーンが参照され、いずれも指定がない場合はUTC扱いとなります。タイムゾーン文字列としては、java.text.SimpleDateFormatの「z」または「Z」パターンで解釈できる表現のほか、UTCであることを示す「Z」表現を受け付けます。
-   (※6). 接続時に指定したタイムゾーン情報が反映され、指定がない場合はUTCとなります。
-   (※7). 16進数バイナリ表現とみなして、文字列とBLOBを相互に変換します。ASCIIの大文字小文字は同一視します。それ以外は変換できずエラーとなります。

#### カラムの値取得

サポートされている型変換先の型と対応するメソッドより、カラムの値を取得できます。 カラムの指定方法としては、カラムラベルとカラムインデックスの両方をサポートします。 その他、次の機能を使用できます。

-   getBinaryStream
    -   byte\[\]への型変換結果に相当します
-   wasNull
    -   JDBC準拠

#### エラー処理

-   不正なカラムインデックス
    -   不正なカラムインデックスを指定して値を取得しようとした場合、JDBC_COLUMN_INDEX_OUT_OF_RANGEエラーからなるSQLExceptionが発生します。
-   型変換エラー
    -   型変換に失敗した場合、JDBC_VALUE_TYPE_CONVERSION_FAILEDエラーからなるSQLExceptionが発生します。

#### 未サポートの機能

次のオプション機能は未サポートです。引数を省略しているものは、全てのオーバーロードが未サポートです。

-   cancelRowUpdates()
-   getArray
-   getAsciiStream
-   getBigDecimal
-   getClob
-   getNClob
-   getNCharacterStream
-   getNString
-   getObject(columnIndex, map)
-   getObject(columnLabel, map)
-   getObject(columnIndex, type)
-   getObject(columnLabel, type)
-   getRef
-   getRow()
-   getRowId
-   getSQLXML
-   getUnicodeStream
-   getURL
-   moveToInsertRow()
-   moveToCurrentRow()
-   refreshRow()
-   rowInserted()
-   rowDeleted()
-   rowUpdated()
-   insertで始まる全メソッド
-   updateで始まる全メソッド
-   deleteで始まる全メソッド

　

### ResultSetMetaDataインターフェース

検索結果ResultSetのメタデータを取得するResultSetMetaDataインターフェースについて説明します。

ResultSetMetaDataインターフェースのJDBC仕様の全メソッドについて、以下の分類で各メソッドの内容や実行結果などを説明します。

- カラムのデータ型を返すメソッド
- その他のメソッド
- 未サポートのメソッド

#### カラムの型

ResultSetMetaDataインターフェースには、検索結果ResultSetのカラムのデータ型を返すメソッドがあります。

| メソッド | 内容 |
|----------|------|
| String getColumnClassName(int column)  | 指定されたカラムのデータ型のクラス名を返します。 |
| int getColumnType(int column)          | 指定されたカラムのデータ型の値を返します。      |
| String getColumnTypeName(int column)   | 指定されたカラムのデータ型の名前を返します。     |

カラムのデータ型と、それぞれのメソッドを実行した値との対応付けを以下に示します。

| カラムのデータ型    | getColumnClassName  | getColumnType   | getColumnTypeName |
|--------------------|---------------------|-----------------|-------------------|
| BOOL型             | "java.lang.Boolean" | Types.BIT       | "BOOL"            |
| STRING型           | "java.lang.String"  | Types.VARCHAR   | "STRING"          |
| BYTE型             | "java.lang.Byte"    | Types.TINYINT   | "BYTE"            |
| SHORT型            | "java.lang.Short"   | Types.SMALLINT  | "SHORT"           |
| INTEGER型          | "java.lang.Integer" | Types.INTEGER   | "INTEGER"         |
| LONG型             | "java.lang.Long"    | Types.BIGINT    | "LONG"            |
| FLOAT型            | "java.lang.Float"   | Types.FLOAT     | "FLOAT"           |
| DOUBLE型           | "java.lang.Double"  | Types.DOUBLE    | "DOUBLE"          |
| TIMESTAMP型        | "java.util.Date"    | Types.TIMESTAMP | "TIMESTAMP"       |
| BLOB型             | "java.sql.Blob"     | Types.BLOB      | "BLOB"            |
| GEOMETRY型         | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |
| 配列型             | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |
| カラムのデータ型を特定できない場合(※1) | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |

<!--[!NOTE]-->
>#### メモ
>- (※1). 例えば「SELECT NULL」を実行して得られるResultSetのような場合
>- GEOMETRY型と配列型については、NoSQLインターフェースで作成された、これらのデータ型を持つテーブルを検索した場合に値が返ります。JDBCでは、これらのデータ型を持つテーブルは作成できません。


#### 単純値を返す属性

ResultSetMetaDataインターフェースで、カラムのデータ型を返す以外のメソッドの実行結果を以下に示します。

| メソッド                                   | 結果                         |
|-------------------------------------------|------------------------------|
| String getCatalogName(int column)         | ""                           |
| int getColumnCount()                      | カラムの数                    |
| int getColumnDisplaySize(int column)      | 131072                       |
| String getColumnLabel(int column)         | カラムのラベル名              |
| String getColumnName(int column)          | カラムの名前                  |
| int getPrecision(int column)              | 0                            |
| int getScale(int column)                  | 0                            |
| String getSchemaName(int column)          | ""                           |
| String getTableName(int column)           | ""                           |
| boolean isAutoIncrement(int column)       | false                        |
| boolean isCaseSensitive(int column)       | true                         |
| boolean isCurrency(int column)            | false                        |
| boolean isDefinitelyWritable(int column)  | true                         |
| int isNullable(int column)                | カラムにNULL値を許可する定数ResultSetMetaData.columnNullable(=1)、またはカラムにNULL値を許可しない定数columnNoNulls(=0)  |
| boolean isReadOnly(int column)            | false                        |
| boolean isSearchable(int column)          | true                         |
| boolean isSigned(int column)              | false                        |
| boolean  isWritable(int column)           | true                         |


#### 未サポートの機能

ResultSetMetaDataインターフェースで未サポートのメソッド(SQLFeatureNotSupportedExceptionが発生するメソッド)はありません。
