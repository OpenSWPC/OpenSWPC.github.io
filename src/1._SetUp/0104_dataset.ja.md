# データセットの準備

## 地下構造モデル

地表海底，あるいは速度不連続面を表す複数の`NetCDF`ファイルによって3次元的な不均質構造を表現する．
その一例として，Koketsu et al. (2012[^Koketsu2012])と地震調査研究推進本部による日本の[全国１次地下構造モデル（Japan Integrated Velocity Structure Model; JIVSM）](https://taro.eri.u-tokyo.ac.jp/saigai/computer.html#2008)およびそれをより広い領域に拡張したextended JIVSM (eJIVSM)モデル（下図）の生成スクリプトが同梱されている．これらのモデルは，地表・海底地形から，日本列島に沈み込むプレートの海洋性モホ面までを含むモデルである．このモデル生成スクリプトを動作させるにはGMTがインストールされている必要がある．これらのモデルを使うのでなければ，本節の作業は必要ない．

[^Koketsu2012]: Koketsu, K., H. Miyake, and H. Suzuki (2012). Japan Integrated Velocity Structure Model Version 1, Proceedings of the 15th World Conference on Earthquake Engineering, Paper No. 1773.

![](../fig/jivsm_ext_section.png)
/// caption
JIVSMモデルとeJIVSMモデル．中央地図で標高値による色が塗られた部分がJIVSMで構造が定義されている領域，灰色部分がeJIVSMモデルによって拡張される未定義領域である．地図上の青・赤線に沿った構造モデル不連続面の深さ断面を上下左右にそれぞれ示す．
///

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
