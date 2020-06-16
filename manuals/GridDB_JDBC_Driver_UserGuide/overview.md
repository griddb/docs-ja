# 概要

JDBCパラメータのプログラムでの指定形式や使用できるデータ型、使用上の注意点を説明します。

　

## 接続方法

### ドライバの指定

JDBCドライバファイル `gridstore-jdbc.jar` をクラスパスに追加します。これによりドライバが自動的に登録されます。 さらに必要に応じて、以下のようにしてドライバクラスを読み込みます。通常は不要です。

``` example
Class.forName("com.toshiba.mwcloud.gs.sql.Driver");
```

　

### 接続時のURL形式

URLは以下の (A)～(D) の形式となります。クラスタ構成方式がマルチキャスト方式の場合、通常は (A) の形式で接続してください。 GridDBクラスタ側で自動的に負荷分散が行われ適切なノードに接続されます。 GridDBクラスタとの間でマルチキャストでの通信ができない場合のみ、他の形式で接続してください。

#### (A) マルチキャスト方式のGridDBクラスタの適切なノードへ自動的に接続する場合

``` example
jdbc:gs://(multicastAddress):(portNo)/(clusterName)/(databaseName)
```

-   multicastAddress：GridDBクラスタとの接続に使うマルチキャストアドレス。(デフォルト: 239.0.0.1)
-   portNo：GridDBクラスタとの接続に使うポート番号。(デフォルト: 41999)
-   clusterName：GridDBクラスタのクラスタ名
-   databaseName：データベース名。省略した場合はデフォルトデータベース(public)に接続します。

#### (B) マルチキャスト方式のGridDBクラスタ内のノードに直接接続する場合

``` example
jdbc:gs://(nodeAddress):(portNo)/(clusterName)/(databaseName)
```

-   nodeAddress：ノードのアドレス
-   portNo：ノードとの接続に使うポート番号。(デフォルト: 20001)
-   clusterName：ノードが属するGridDBクラスタのクラスタ名
-   databaseName：データベース名。省略した場合はデフォルトデータベース(public)に接続します。

#### (C) 固定リスト方式のGridDBクラスタに接続する場合

クラスタ構成方式が固定リスト方式の場合、この形式で接続してください。

``` example
jdbc:gs:///(clusterName)/(databaseName)?notificationMember=(notificationMember)
```

-   clusterName：GridDBクラスタのクラスタ名
-   databaseName：データベース名。省略した場合はデフォルトデータベース(public)に接続します。
-   notificationMember：ノードのアドレスリスト(URLエンコードが必要)。デフォルトポートは20001
    -   例：192.168.0.10:20001,192.168.0.11:20001,192.168.0.12:20001

※notificationMemberはgs_cluster.jsonファイルを編集することで変更可能です。 アドレスリストで使うポートは、gs_node.jsonファイルを編集することで変更可能です。

#### (D) プロバイダ方式のGridDBクラスタに接続する場合

クラスタ構成方式がプロバイダ方式の場合、この形式で接続してください。

``` example
jdbc:gs:///(clusterName)/(databaseName)?notificationProvider=(notificationProvider)
```

-   clusterName：GridDBクラスタのクラスタ名
-   databaseName：データベース名。省略した場合はデフォルトデータベースに接続します
-   notificationProvider：アドレスプロバイダのURL(URLエンコードが必要)

※notificationProviderはgs_cluster.jsonファイルを編集することで変更可能です。

なお、(A)～(D)いずれの場合でも、ユーザ名・パスワードをURLに含める場合は、URLの末尾に次のように追加してください。

``` example
?user=(ユーザ名)&password=(パスワード)
```

　

### 接続タイムアウトの設定

以下の(A)、(B)どちらかの方法で接続タイムアウトを設定できます。両方が設定された場合、(B)の設定が優先されます。 どちらも設定されない場合、デフォルト値300秒(5分)が使用されます。

#### (A) DriverManager\#setLoginTimeout(int seconds)で指定する

secondsの値によって、以下のように設定されます。設定後、DriverManager\#getConnectionまたはDriver\#connectで取得する全てのGridDBへのConnectionに接続タイムアウトが設定されます。

-   値が1～Integer.MAX_VALUEの設定値の場合
    -   指定した秒数で設定されます
-   値がInteger.MIN_VALUEの設定値～0の場合
    -   設定されません

#### (B) DriverManager\#getConnection(String url, Properties info)またはDriver\#connect(String url, Properties info)で指定する

引数infoにキー”loginTimeout”でプロパティを追加してください。キー”loginTimeout”に対応する値が数値に変換できた場合、取得したConnectionにのみ以下のように接続タイムアウトが設定されます。

-   変換した値が1～Integer.MAX_VALUEの場合
    -   指定した秒数で設定されます
-   変換した値が0以下 または Integer.MAX_VALUEより大きい場合
    -   設定されません

　

### その他情報の設定
前述した設定のほか、接続時には次の情報も設定できます。

- アプリケーション名
- タイムゾーン(Z|±HH:MM|±HHMM)

※タイムゾーン処理を行う場合、GridDBからの取得コストが増えるため、極力アプリ側で処理を行うことを推奨します。


以下の(A)、(B)どちらかの方法でこれらは設定できます。両方が設定された場合は
エラーとなります。

#### (A) URLで指定する

アプリケーション名をURLに含める場合は、URLの末尾に次のように追加してください。

``` example
?applicationName=(アプリケーション名)
```

タイムゾーンをURLに含める場合は、URLの末尾に次のように追加してください。

``` example
?timeZone=(タイムゾーン)
```


ユーザ名・パスワードもURLに含める場合は、次のように追加してください。

``` example
?user=(ユーザ名)&password=(パスワード)&applicationName=(アプリケーション名)&timeZone=(タイムゾーン)
```

タイムゾーンの記号「:」など、URLエンコードが必要なものは適宜実施してください。

#### (B) DriverManager#getConnection(String url, Properties info)またはDriver#connect(String url, Properties info)で指定する

引数infoに下記のキーでプロパティを追加してください。

- アプリケーション名: applicationName
- タイムゾーン: timeZone

<!--[!WARNING]-->
>#### 注意
>- NoSQLインタフェース/NewSQLインタフェースの違い
>  - NoSQLインタフェースのクライアントで作成したコンテナは、NewSQLインタフェースのJDBCドライバで参照、更新可能です。更新には行の更新だけでなく、コンテナのスキーマや索引の変更を含みます。
>  - NewSQLインタフェースのJDBCドライバで作成したテーブルは、NoSQLインタフェースのクライアントで参照、更新可能です。
>  - NoSQLインタフェースの場合は「コンテナ」、NewSQLインタフェースの場合は「テーブル」と呼びます。呼び方が異なるだけでどちらも同じオブジェクトを指します。
>  - NoSQLインタフェースのクライアントで作成した時系列コンテナを、JDBCドライバからSQLで検索した場合、NoSQLクライアントからTQLで検索した場合と異なり、主キーに対するORDER BY句がなければ結果は時刻順とはなりません。SQL結果の時刻順整列が必要な場合には主キーに対するORDER BYを指定してください。
>- 一貫性について、インタフェースの違いによらず、トランザクションの分離レベルとしてREAD COMMITTEDをサポートしています。  
>  NewSQLインタフェースによる検索・更新結果は、NoSQLインタフェースにて部分実行オプションを有効にしてTQLを実行した場合と同様、実行開始時点の単一のスナップショットに基づくとは限りません。  
>  実行対象のデータ範囲に応じて分割された、個々の実行時点のスナップショットに基づく場合があります。
  この特性は、パーティショニングされていない単一のテーブルに対する、NoSQLインタフェースによるデフォルト設定での操作とは異なります。
