---
layout: '../../layouts/BlogPostLayout.astro'
title: '国交省の海岸線データをいい感じに整形する'
pubDate: 2022-07-01
description: '国土交通省が提供する海岸線データのXMLファイルを使用するとき，データの整形に手間取ったのでメモ．'
author: 'halphy'
tags: ["Python", "アルゴ"]
---

この前，日本の海岸線データが必要になることがあったので，その際にしたことを書き残しておく．

## やったこと
### XMLファイルをダウンロードする
日本の海岸線データは，国土交通省の提供する[国土数値情報ダウンロードサイト](https://nlftp.mlit.go.jp/)の「[海岸線データ](https://nlftp.mlit.go.jp/ksj/jpgis/datalist/KsjTmplt-C23.html)」からダウンロードすることができる．

データは都道府県ごとにXML形式で保存されていたので，ためしに茨城県のファイル`C23-06_08.xml`をダウンロードしてみた．

### XMLファイルの中身を調べる
まずは適当なエディタでXMLファイルを開いてみると，こんな感じの構造になっている（一部省略・整形している）．

```xml
<dataset>
    <ksj:object>
        <ksj:AA01>
            <ksj:RES idref="rs001" />
                <ksj:OBJ>
                    <jps:GM_Point id="p_00001">
                        <!-- GM_Pointの中身 -->
                    </jps:GM_Point>
                    <!-- ... -->
                    <jps:GM_Curve id="c_00001">
                        <!-- GM_Curveの中身 -->
                    </jps:GM_Curve>
                    <!-- ... -->
                </ksj:OBJ>
            </ksj:RES>
        </ksj:AA01>
    </ksj:object>
</dataset>
```

てっきり，海岸線データはひとつながりの折れ線で記述されていて，折れ線を構成する頂点の座標が順番に格納されていると思っていたが，どうやら違うらしい．

実際には次のようになっている．

- 海岸線はいくつかの断片に分かれており，それぞれの断片の情報が独立に格納されている．各断片の情報は，`<jps:GM_Curve>`要素に格納されている．
    - `<jps:GM_Curve>`要素の`id`属性は，断片ごとに付与されいる固有の識別IDを表す．
    - 断片の折れ線を構成する点の座標は，`<jps:GM_Curve>`要素の子要素`<jps:GM_PointArray>`内に格納されている．
- 断片の端点（便宜上**代表点**と呼ぶことにする）にだけ固有の識別IDが付与されており，あらかじめ`<jps:GM_Point>`要素で定義されている．

このように，海岸線データは断片ごとに分かれて記述されているので，実際にデータを利用する際には，これを**ひとつながりの折れ線データに変換したほうが便利**である．

困ったことに，**断片の並び順はばらばら**で，必ずしも実際の海岸線の順番に並んでいない．加えて，データの中には，本体の海岸線データのほかに**島の海岸線データも含まれている**．そんなわけで，データの整形はちょっと面倒そう．

### 海岸線データを整形する
書き慣れているPythonで書くことにした．

XMLファイルの読み込みとパースには`xml.etree.ElementTree`モジュールを使う．XMLのノードの子要素を検索するときには`find`, `findall`メソッドを使うとよいが，要素名に独自の名前空間`ksj`, `jps`が使われているので少しはまった．調べてみると，`namespaces`プロパティに名前空間を定義するDictionaryを指定すれば解決するらしい．

```python
import xml.etree.ElementTree as ET

namespaces = {
    'ksj': 'http://nlftp.mlit.go.jp/ksj/schemas/ksj-app',
    'jps': 'http://www.gsi.go.jp/GIS/jpgis/standardSchemas'
}

tree = ET.parse("C23-06_08.xml")
root = tree.getroot()

for child in root:
    if child.tag == "dataset":
        for gm_point in child.findall(
            path="ksj:object/ksj:AA01/ksj:OBJ/jps:GM_Point",
            namespaces=namespaces
        ):
            # 処理
            pass
```

XMLが読み込めたら，データを整形する（それぞれの断片をつなぎあわせる）作業に移る．

競プロ脳なのでグラフの問題に帰着させて考えてしまうのだけれど，要するにこういうことである：

- 「代表点」を頂点，代表点同士をつなぐ断片を辺にもつグラフを考えると，このグラフの連結成分は，パスグラフかサイクルグラフのいずれかである．
    - パスグラフが本土の海岸線，サイクルが島に相当する．
- データを整形するには，それぞれの連結成分に対して以下を行えばよい．
    - **パスグラフの場合**：次数1の頂点を始点として深さ優先探索（DFS）すれば，通る頂点を探索順に並べたものがそのまま折れ線データになる．
    - **サイクルの場合**：適当な頂点を始点としてDFSすれば探索順がそのまま折れ線データになる．

実装する際には頂点を「代表点」のIDで管理すればよい．
