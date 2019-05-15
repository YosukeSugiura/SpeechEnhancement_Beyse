# MAP推定に基づく音声強調

## 背景

雑音が混入した音声信号から原音声信号を推定する手法を音声強調と言う．
入力から出力を生成するための様々な"アルゴリズム(関数，あるいはフィルタとも呼べる)"が提案されている．

音声強調アルゴリズムを設計するには，まず入出力の関係を数理モデルで表す必要がある．
すなわち，入力を x ，出力を y としたとき，

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;y=G(x)" title="\small y=G(x)" />

で表される音声強調モデル G を考える必要がある．
一般の音声強調では，モデル G を**線形システム**と考えることが多い．

内部パラメータの最適化には，**最小二乗法**が最も使われる．
すなわち，入力を<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;x"/>，
モデルを<img src="https://latex.codecogs.com/gif.latex?\dpi{110}&space;\small&space;G"/>，
出力を<img src="https://latex.codecogs.com/gif.latex?\dpi{110}&space;\small&space;G(x)"/>と置いたとき，

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}||x-G(x)||^2_2"/>
    
を最小化するように内部パラメータを調整する．
入力<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;x"/>としてどのような特徴量を設定するか，あるいは内部パラメータに対する拘束条件を何か付与するか，
がこの手法における大きな研究課題である．
最も単純なモデルとして，入力と雑音が定常であり，無限時間観測であることを仮定したものが**ウィナーフィルタ**である．さらに入力・雑音の定常性の仮定を廃し，時系列の変化を考慮したシステム（=**線形動的システム**）を導入したものが**カルマンフィルタ**である．
現在ある様々な音声強調手法のほとんどが，このウィナーフィルタとカルマンフィルタに基づき開発されている．

さて，ベイスの枠組みから観ると，この最小二乗法は雑音の生成分布を正規分布と仮定した場合の**最尤推定**と一致する．
最尤推定はシンプルな推定法で，とりわけ音声強調においては，簡略化のため原音声の生成分布を無視する．
ただし実際の音声信号は0付近の値が頻出するなど，明らかに生成に"偏り"があるため，
これを無視した最尤推定は音声強調性能が低い．
最小二乗法ではさらに雑音を正規分布と仮定しているため，有効に機能する状況は非常に限定的である．

> 最尤推定は未知量 = 原音声を確率変数とみなさないために，未知量に生成分布を仮定しない(=決定論的発想)．
> 未知量を確率変数として扱わないということは，ベイズの枠組みでは，
> 事前分布（=原信号の生成分布）を一様分布としていることと同義である．
> 
> 厳密には，最尤推定は事前分布が一様分布，事後分布が最適値で+∞，
> 他で０となると仮定した場合のベイズ推定と一致する．


# 1. システムモデル

ベイズの枠組みを利用すると，入力信号の生成分布を考慮することができる．
ベイズの枠組みにおいて，入力信号から所望信号を推定する手法は**MAP推定**と呼ばれる（最尤推定はMAP推定の一部）．
MAP推定は入力信号の生成分布を加味できるので，音声強調性能が向上すると期待できる．

## 音声強調モデル

ここでは，雑音混入音声<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;x"/>を原音声<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;s" />と加法性雑音<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;w" />の加算で表す．

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}x=s+w"/>
    
音声強調を行う関数を<img src="https://latex.codecogs.com/gif.latex?\dpi{110}&space;\small&space;G" />と置くと，雑音混入音声<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;x" />と出力音声<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;y" />との間に

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}y=G(x)"/>
    
という関係がある．y は原音声との間に
    
　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}y=s+\epsilon"/>

という関係が成り立つ．<img src="https://latex.codecogs.com/gif.latex?\inline&space;\dpi{150}&space;\small&space;\epsilon"/>は推定誤差である．
ここで変数をまとめると，

　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;s"/>  : 所望信号（原音声）のサンプル値  
　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;w"/>  : 雑音信号 のサンプル値  
　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;x"/> : 入力信号（雑音混入音声）のサンプル値  
　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;y"/> : 出力信号 のサンプル値  
　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{110}&space;\small&space;G"/> : 音声強調モデル  
　　- <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;\epsilon"/> : 推定誤差  
    
である．音声強調の研究において，音声強調モデル G としてどのようなモデルを用いるかが重要な課題となる．

## 音声信号モデル

音声の生成分布は，分析フレームを長く設定すると正規分布に近似できることが知られている
（これは中心極限定理からも納得できる）．

一方で，分析フレームを短くすると，音声の時変性が強くなり，
音声の状態（例えば，無音，子音，母音等）によって生成分布が異なる．
無音や音声パワーが小さい状態であれば，"0"に近いサンプル値が頻出するので，生成分布はラプラス分布に近づくと考えられる．
また子音や母音であればその生成分布は正規分布に近づくはずである．
ただし，分布の分散や平均は音声の状態に依存して時々刻々と変化する．

今，加法性雑音が正規分布に従って生成されると仮定すると，
入力信号の生成分布は 所望信号 s のサンプル値を中心に正規分布を形成する．
> 一般に，正規分布に従い生成される雑音は決して多くないが（白色雑音等），ここでは雑音の生成分布を正規分布と仮定する．
> この仮定は信号処理分野において一般的であり，従来から検討されていた最小二乗の規範からも妥当だと考えられる．


# 2. ベイズ推定とMAP推定

## ベイズ推定

音声強調モデル G を設計するにあたり，まず信号復元の仕組みを**ベイズ推定**に当てはめて考える．

　　<img src="https://latex.codecogs.com/gif.latex?\inline&space;\dpi{150}&space;\small&space;y={\rm&space;argmax}_s&space;\&space;p(s|G,x)"/>
    
　　<img src="https://latex.codecogs.com/gif.latex?\inline&space;\dpi{150}&space;\small&space;p(s|G,x)=\frac{p(G,x|s)&space;p(s)}{p(x)}" title="\small p(s|G,x)=\frac{p(G,x|s) p(s)}{p(x)}" />

ここで，

　　- <img src="https://latex.codecogs.com/gif.latex?\small&space;p(s|G,x)" />　: 事後分布．G, x が発生したときのその原因が s である確率分布．  
　　- <img src="https://latex.codecogs.com/gif.latex?\small&space;p(G,x|s)" />　: 尤度．s を原因として G, x が発生する確率分布．  
　　- <img src="https://latex.codecogs.com/gif.latex?\small&space;p(s)" />　: 事前分布．s が生成される確率分布．  
　　- <img src="https://latex.codecogs.com/gif.latex?\small&space;p(x)" />　: 周辺分布．x が生成される確率分布．  

である．
ベイズ推定は，下の式にある事後分布<img src="https://latex.codecogs.com/gif.latex?\small&space;p(s|G,x)" />を推定することである．
事後分布さえ推定できれば，上の式から未知量 y = s を推定することができる．

音声強調の観点から説明すると，ベイズ推定は「音声強調関数 G と 雑音混入音声 x から，それらの原因が原音声 s である確率分布を求める」ことである．

## MAP推定

 x が s と独立であると仮定すると，周辺分布<img src="https://latex.codecogs.com/gif.latex?\small&space;p(x)" />は s と独立した確率分布となり，
 y の推定に寄与しないことがわかる．
したがって，実際に y を推定する際には<img src="https://latex.codecogs.com/gif.latex?\small&space;p(x)" />を無視して，
 
　　<img src="https://latex.codecogs.com/gif.latex?\inline&space;\dpi{150}y={\rm&space;argmax}_s&space;\&space;p(G,x|s)p(s)"/>

を解くことになる．
この推定の枠組みを最大事後確率推定（=**MAP推定**）と呼ぶ．
音声強調におけるMAP推定の働きを噛み砕いて説明すると，
「音声強調関数 G と雑音混入音声 x が与えられたとき，取りうる原音声 s の値それぞれに対して，x の原因としてどれだけふさわしいか（=尤度）を計算する．
さらに尤度を s の発生確率で補正し，尤度が最大となる s の値を最終的な出力信号 y をとして推定する．」
ことである．

式からわかるように，**事後分布 p(G, x | s) と事前分布 p(s) が設定できれば，MAP推定を用いることができる**．
ここで事前分布<img src="https://latex.codecogs.com/gif.latex?\small&space;p(s)" />は推定量 s のデータ分布であり，事前に適当に決定しておく必要がある．

原音声 s の生成分布を一様分布と仮定すると，MAP推定は最尤推定に一致する．
また， s の生成分布がラプラスに従うと仮定して，STSA法により音声強調を行う手法(Joint MAP法)もある．
音声生成分布をどう設計するかは，大きな研究テーマとなっている．

### 補足１：最尤推定

事前分布は真となるデータの分布であり，事前に適切な分布を設定しなければ，推定精度が大きく劣化する．
しかし，実際の応用では真のデータの分布が未知であることが多く，
事前分布をモデル化することが困難である場合がある．

この場合，周辺分布と事前分布を未知 = 定数 とみなして，
MAP推定における p( s ) を無視して

　　<img src="https://latex.codecogs.com/gif.latex?\inline&space;\dpi{150}y={\rm&space;argmax}_s&space;\&space;p(G,x|s)"/>
 
とすればよい．この推定法を**最尤推定**と呼ぶ．
事前分布を定数とみなすことは，無限に平坦な分布を仮定することと等しい．
音声強調において，この仮定は「原信号があらゆる値を一様に取りうる」ことを意味する．
しかし音声は0付近の値が頻出するなど明らかに"偏り"があるわけで，この仮定は現実的ではない．
そこで現在でも事前分布をどう仮定するか様々な研究がなされている．

> まとめ：
>　　
> 1.  ベイズ推定     
>
>       事後分布 p( x | y ) を推定する．  
>       または，事後分布から未知量 x ，あるいは事前分布（= 未知量の確率分布 p( x ) ）を推定する．
>
>       <img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;p(x|y)=\frac{p(y|x)p(x)}{p(y)}" title="\small p(x|y)=\frac{p(y|x)p(x)}{p(y)}" />
>  
>  
> 2.  MAP推定   
>
>       事後分布 p( x | y ) を最大化する未知量 x を推定値とする．  
>       ベイズの枠組みにおいて，周辺尤度 p( y )（= 観測データ y の生成分布）を無視する．
>
>　　<img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;\tilde{x}={\rm&space;argmax}_x&space;\&space;p(y|x)p(x)"/>  
>  
>  　
>　　<img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;\because&space;p(x|y)=\frac{p(y|x)p(x)}{p(y)}\propto&space;p(y|x)p(x)" title="\small \because p(x|y)=\frac{p(y|x)p(x)}{p(y)}\propto p(y|x)p(x)" />
>
> 3.  最尤推定  
>
>       尤度 p( x | s ) を最大化する未知量 s を推定値とする．  
>       ベイズの枠組みにおいて，周辺尤度を無視し，事前分布を一様分布と仮定する．
>       
>　　<img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;\tilde{x}={\rm&space;argmax}_x&space;\&space;p(y|x)"/>  
> 　
>　　<img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;\because&space;p(x)=\rm&space;const" />
>  
>  
> 3.  最小二乗解（=MMSE推定）  
>
>       推定未知量 <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;{\color{Gray}\tilde{x}}" title="\small {\color{Gray}\tilde{x}}" /> と 未知量 x の平均二乗誤差を計算し，最小となる <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;{\color{Gray}\tilde{x}}" title="\small {\color{Gray}\tilde{x}}" /> を推定値とする．  
>       ベイズの枠組みにおいて，周辺尤度を無視し，事前分布を一様分布と仮定し，事後分布を正規分布と仮定する．
>
>　　<img src="https://latex.codecogs.com/gif.latex?\fn_phv&space;\small&space;\tilde{x}={\rm&space;argmin_\tilde{x}}&space;\&space;E&space;\left[||\tilde{x}-x||^2_2&space;\right]" title="\small \hat{x}={\rm argmin_\hat{x}} \ E \left[||\hat{x}-x||^2_2 \right]" />
>

### 補足２：Joint MAP 法 (MMSE-STSA法ベース) [[link](https://pdfs.semanticscholar.org/6e39/5084f54260ad90299c33d229aea6a840c873.pdf)]

MMSE-STSA法をベースに現実的な音声生成モデルを適用した音声強調手法．
MMSE-STSA法では原音声の振幅スペクトルの生成分布（=事前分布）が正規分布であると仮定していたが，
Joint MAP法では原音声の振幅スペクトルがラプラス分布に従って生成されると仮定した．
 
> MMSE-STSA : 
>
> スペクトルゲイン法に基づき，真の振幅スペクトルと推定振幅スペクトルの二乗誤差が最小となる
> スペクトルゲインを推定する手法．
> 音声スペクトルの生成分布（=事前分布）を正規分布と仮定し，
> 雑音スペクトルの生成分布（=尤度）もまた正規分布と仮定した．
>
> 文献：Y. Ephraim, D. Malah. Speech enhancement using a minimum-mean square error short-time spectral amplitude estimator. IEEE Trans. ASSP. vol.32, pp.1109-1121. 1984. [[link](https://ieeexplore.ieee.org/document/1164453)]

### 補足３：可変音声分布に基づく Joint MAP法 [[link](https://search.ieice.org/bin/summary.php?id=e90-a_8_1587)]


Joint MAP法をベースに音声生成モデルを時変にした音声強調手法．
原音声の振幅スペクトルが**レイリー分布**に従って生成されると仮定し，入力音声からそのレイリー分布のパラメータを推定した．


# 3. 音声強調モデルの設計

## 尤度の設定

MAP推定の式を再掲する．

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;y={\rm&space;argmax_s}&space;\&space;p(G,x|s)p(s)" title="\small y={\rm argmax_s} \ p(G,x|s)p(s)" />
    
MAP推定においては，事前分布 p( s ) を設定する他に，尤度 p( G, x | s ) を設計する必要がある．
音声強調において尤度 p( G, x | s ) は「原音声 s に起因して 雑音混入音声 x と音声処理関数 G が生成される確率」を表している．
誤差信号 e = s - G( x ) が正規分布に従うと仮定した場合，音声強調結果 G(x) は真の音声 s の周りで正規分布を形成するはずである．
したがって，尤度は以下の式のようにおくことができる．

　　<img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;p(G,x|s)=q\exp\left(\frac{-|G(&space;x&space;)-s|^2}{2\sigma^2}\right)" title="\small p(G,x|s)=q\exp\left(\frac{-|G( x )-s|^2}{2\sigma^2}\right)" />

ここで σ^2 は誤差信号の分散である．q は分散に起因する定数であるが，MAP推定では推定に寄与しない．
この関数は，G( x ) = s のときに最大値をとる．

## 音声強調モデルの学習

さて，これまでの議論はすべて音声強調モデル G が既知としていた．
実際の応用では G を推定することが最も重要である．
例えば G をニューラルネットワークなどでモデリングする場合，
Gの内部パラメータを学習する必要がある．
今，学習用の原音声 <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;\hat{s}" title="\small \hat{s}" /> と雑音混入音声 <img src="https://latex.codecogs.com/gif.latex?\dpi{150}&space;\small&space;\hat{x}" title="\small \hat{x}" /> のデータセットを用いて学習するとする．
このときの G の事後分布は

    <!--p(G|\hat{x},\hat{s})=p(\hat{x},\hat{s},G)p(G)-->

     G = argmax_G p( s | G, x ) = argmax_G p( G, x | s ) p( s )

左辺に対して対数を取ることで，次式に書き換えられる．

    G = argmin_G  { - log p( G, x | s ) - log p( s )  }
    
ここで - log p( G, x | s ) = | G( x ) - s |^2 / 2σ^2 なので
（ argmin に寄与しない q は無視している），- log p( s ) = F( s ) と置いて，

    G = argmin_G  E_G
    
    E_G = { | G( x ) - s |^2 / 2σ^2 + F( s ) } 
    
と変形する．
上の式の argmin_G  E_G を計算するには，E_G  を G で偏微分して最小となる G を求めればよい．

## 学習例

<!--
ここで G は一般的な関数として表現しているが，
例えばニューラルネットワークにおいては G を重みやバイアス等の学習したいパラメータに置き換えて計算すればよい．
その場合，学習したいパラメータを a とおくと，a を以下のような更新式で更新する．

    a = a  - μ  ∂ E_G / ∂ a  
    
仮に p( s ) が平均 0，分散 α^2 の正規分布に従うと仮定した場合，勾配は

    ∂ E_G / ∂ a   = | G( x ) - s | 
    

と計算できる．β は適当な定数である．
この場合，更新式は **L2正則化 + 最小二乗法**と一致する．
また F( s ) が平均 0，分散 α^2 のラプラス分布


執筆中．

## 音声の分布が変化する場合



