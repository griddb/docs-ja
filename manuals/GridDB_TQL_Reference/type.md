# データ型

フィールドならびにクエリ演算で用いる値の制約を示す、型について定義します。

## 基本型

他の型の組み合わせで表現できない、基本的な型を定義します。

### ブール(BOOL)型

真または偽のいずれかの値を表します。

### 文字列(STRING)型

Unicodeコードポイントを文字とする、任意個数の文字の列より構成されます。ただし、NULL文字(U+0000)は扱いません。
上限サイズについては『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照してください。

### 整数型

それぞれ次の範囲の整数値を表します。
-   BYTE型: -2<sup>7</sup>から2<sup>7</sup>-1 (8ビット)
-   SHORT型: -2<sup>15</sup>から2<sup>15</sup>-1 (16ビット)
-   INTEGER型: -2<sup>31</sup>から2<sup>31</sup>-1 (32ビット)
-   LONG型: -2<sup>63</sup>から2<sup>63</sup>-1 (64ビット)

### 浮動小数点数型

IEEE754で定められた浮動小数点数を表します。精度に応じた次の型があります。
-   FLOAT型: 単精度型(32ビット)
-   DOUBLE型: 倍精度型(64ビット)

※演算精度は原則IEEE754準拠ですが、実行環境により異なる場合があります。

### 時刻(TIMESTAMP)型

年月日ならびに時分秒からなる時刻を表します。表現範囲については付録の[値の範囲](annex.md#label_range_of_values)を参照してください。

### 空間(GEOMETRY)型

空間構造を表します。上限サイズについては『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照してください。
各構造の表す座標の数値として非数(NAN)や正負の無限大(INF、-INF)は扱いません。 また、SRID (Spatial Reference System Identifier)を整数型の値として格納できますが、SRIDの表す座標系による座標の範囲制限や、SRIDの変換による座標変換については対応していません。

### BLOB型

画像や音声などのバイナリデータを表します。上限サイズについては『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照してください。

## 複合型

基本型の組み合わせで構成される型を定義します。

### 配列型

値の列を表します。以下の型について配列型を提供します。長さとは、構成される配列要素の数であり、最小値は0となります。
長さの上限については『[GridDB 機能リファレンス](../GridDB_FeaturesReference/toc.md)』を参照してください。配列の要素にNULLは設定できません。
-   ブール型
-   文字列型
-   整数型
-   浮動小数点数型
-   時刻型