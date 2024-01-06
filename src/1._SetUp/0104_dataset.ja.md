# データセットの準備

## 地下構造モデル

地表海底，あるいは速度不連続面を表す複数の`NetCDF`ファイルによって3次元的な不均質構造を表現する．
その一例として，地震調査研究推進本部と(Koketsu et al., 2012)による日本の全国１次地下構造モデル（Japan Integrated Velocity Structure Model; JIVSM）およびそれをより広い領域に拡張したextended JIVSM (eJIVSM)モデル（下図）の生成スクリプトが同梱されている．これらのモデルは，地表・海底地形から，日本列島に沈み込むプレートの海洋性モホ面までを含むモデルである．このモデル生成スクリプトを動作させるにはGMTがインストールされている必要がある．これらのモデルを使うのでなければ，本節の作業は必要ない．

!!! Note "JIVSM/eJIVSMの定義領域と断面図"
    ![](../fig/jivsm_ext_section.png)
    JIVSMモデルとeJIVSMモデル．中央地図で標高値による色が塗られた部分がJIVSMで構造が定義されている領域，灰色部分がeJIVSMモデルによって拡張される未定義領域である．地図上の青・赤線に沿った構造モデル不連続面の深さ断面を上下左右にそれぞれ示す．

このモデルを構築するには，まず`dataset/vmodel`ディレクトリに移動し，以下のシェルスクリプトを実行する: 
```bash
$ ./gen_JIVSM.sh
```

正常に動作すると，ディレクトリ`JIVSM`と`eJIVSM`以下にそれぞれ23個の`NetCDF`ファイルが生成される．これらはGMTの`grd`ファイルでもあり，GMTの`grdimage`,
`grdcontour`等で直接扱うことができる．ファイル名には連番のほか5つの数字が埋め込まれており，それぞれ左から密度（kg/m${}^3$）,
P波速度（m/s）, S波速度（m/s）,QP, QSに対応する．
これらの物性値はそのファイルで規定される不連続面の下の構造を表している．
また，本コードの入力に用いる構造モデルリストファイル`jivsm.lst`,`ejivsm.lst`が作成される．

## 観測点リスト

`dataset/station`ディレクトリに防災科学技術研究所Hi-netにより配布されている観測点一覧からOpenSWPC用の入力ファイルを作成するスクリプト`gen_stlst_hinet.sh`がある．スクリプト実行のためには，観測点一覧をあらかじめダウンロードし，同ディレクトリに配置しておく必要がある．

観測点リストは主として緯度経度からなる単純なテキストファイルである（次章参照）ので，必ずしもこのスクリプトによって作成する必要はない．


## 埋め込みパラメータの調整

`OpenSWPC`の挙動は原則としてパラメータファイルで制御されるが，計算速度の向上のため，幾つかの変更頻度の低いと期待されるパラメタがソースコードに埋め込まれている．これらはいずれも`m_global.F90`で定義されている．以下の値を修正した場合には，コード全体の再コンパイル（`make clean; make`）が必要である．


!!! Info "Parameters"

    **`UC`**
    : 計算に用いる混合座標系（距離km, 密度g/cm$^3$, 地震波速度km/sなどの慣用単位）での結果をSI単位に換算する定数． 計算の単位系を変更する場合には調整が必要となる．デフォルト値は `1e-15` (3D) もしくは `1e-12` (2D)．  

    **`MP`**
    : 混合精度差分法の精度．`DP`の場合，必要な変数については倍精度で，それ以外の変数を単精度でそれぞれ計算する．これをSPに変更すると，計算すべてを単精度で実行するようになる．単精度計算では特に震源周辺で桁あふれに伴う不安定が起こる場合があるが，必要となるメモリ量は約2/3に小さくなり，計算も速い．デフォルト値は倍精度を意味するパラメタ`DP`．具体的な値はFortranコンパイラによって自動的に決定されるが，多くのコンパイラで`8`を取る．

    **`NM`**
    : メモリ変数を用いて計算する一般化Zener粘弾性体の並列数．1より大きな値の場合には，パラメタ`fq_*`に応じて指定された周波数範囲において周波数一定の$Q$値を取るように最適化される．一方，0の場合には完全弾性体の問題となり，内部減衰は考慮されない．特に3次元計算においては，計算に必要な時間やメモリ量がこのパラメタによって大きく変動する．