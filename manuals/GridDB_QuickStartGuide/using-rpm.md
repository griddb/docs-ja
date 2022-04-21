# RPMファイルを利用する場合

  CentOS 7.9の環境での動作を確認しています。

<!--[!WARNING]-->
  >#### 注意
  >- 事前にPython3をインストールしてください。
  >- このパッケージをインストールすると、OS内にgsadmユーザが作成されます。運用コマンドはgsadmユーザで操作してください。  
  >    例
  >    ```
  >    # su - gsadm
  >    $ pwd
  >    /var/lib/gridstore
  >    ```
  >- gsadmユーザでログインすると環境変数 GS_HOMEとGS_LOGが自動的に設定されます。また、運用コマンドの場所が環境変数 PATHに設定されます。
  >- Javaクライアントのライブラリ(gridstore.jar)は/usr/share/java上に、サンプルは/usr/griddb-XXX/docs/sample/program上に配置されます。
  >- 過去版がインストールされている場合は、アンインストール後、/var/lib/gridstore上のconf/,data/を削除してください。

## インストール

対象OSのパッケージをインストールします。

    (CentOS)
    $ sudo rpm -ivh griddb-X.X.X-linux.x86_64.rpm

    ※ X.X.Xはバージョンを意味します。

### インストール後のユーザとグループ

GridDBパッケージのインストールを行うと、次のようなユーザやディレクトリ構成が作成されます。

#### GridDBユーザとグループ

OSのグループgridstoreとユーザgsadmが作成されます。ユーザgsadmは、GridDBを運用するためのユーザとして使用します。

| ユーザ | 所属グループ | ホームディレクトリ |
|---------|-------|---------------------|
| gsadm | gridstore | /var/lib/gridstore |

ユーザ gsadm (の .bash_profile ファイル) には、次の環境変数が定義されます。

| 環境変数 | 値 | 意味 |
|---------|----|------|
| GS_HOME | /var/lib/gridstore | ユーザgsadm/GridDBホームディレクトリ |
| GS_LOG  | /var/lib/gridstore/log | ノードのイベントログファイルの出力ディレクトリ |



### インストール後のディレクトリ構成

ノードの定義ファイルやデータベースファイルなどを配置するGridDBホームディレクトリと、インストールしたファイルを配置するインストールディレクトリが作成されます。

#### GridDBホームディレクトリ
```
/var/lib/gridstore/                      # GridDBホームディレクトリ
                   conf/                 # 定義ファイルディレクトリ
                        gs_cluster.json  # クラスタ定義ファイル
                        gs_node.json     # ノード定義ファイル
                        password         # ユーザ定義ファイル
                   data/                 # データファイル,チェックポイントログディレクトリ
                   txnlog/               # トランザクションログファイルディレクトリ
                   log/                  # ログディレクトリ
```

#### インストールディレクトリ
```
/usr/griddb/                            # インストールディレクトリ
            bin/                        # 運用コマンド、モジュールディレクトリ
            conf/                       # 定義ファイルディレクトリ
                gs_cluster.json         # クラスタ定義ファイル
                gs_node.json            # ノード定義ファイル
                password                # ユーザ定義ファイル
            conf_multicast/             # 定義ファイルディレクトリ(リモート接続用)
            3rd_party/                  
            docs/
                manual/
                sample/
```

## 環境設定

「[ソースコードを利用する場合](using-source-code.md#ソースコードを利用する場合)」の「[環境設定](using-source-code.md#環境設定)」と同じです。

## 起動／停止

gsadmユーザで操作してください。
また、デフォルトはローカル接続限定の設定になっていますので、以下の操作でコンフィグを変更してください。

    [gsadm]$ cp /usr/griddb-X.X.X/conf_multicast/* conf/.

それ以外は「[ソースコードを利用する場合](using-source-code.md#ソースコードを利用する場合)」の「起動／停止」と同じです。

## サンプルプログラムのビルド・実行方法

プログラムのビルドおよび実行方法の例を示します。

[Javaの場合]

1. 環境変数の設定
2. サンプルプログラムをgsSampleディレクトリにコピー
3. ビルド
4. 実行

```
$ export CLASSPATH=${CLASSPATH}:/usr/share/java/gridstore.jar:.
$ mkdir gsSample
$ cp /usr/gridstore-X.X.X/docs/sample/program/Sample1.java gsSample/.
$ javac gsSample/Sample1.java
$ java gsSample/Sample1 239.0.0.1 31999 設定したクラスタ名 admin 設定したパスワード
```


## GridDBのアンインストール

GridDBが不要となった場合にはパッケージをアンインストールします。root権限で、以下の手順でアンインストールを実行してください。

[実行例]

  ```
    (CentOS)
    $ sudo rpm -e griddb
  ```

<!--[!WARNING]-->
>#### 注意
>- 定義ファイルやデータファイルなど、GridDBホームディレクトリ下のファイルはアンインストールされません。不要な場合は手動で削除して下さい。
