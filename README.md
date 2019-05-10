# MAP推定を利用した音声強調

入力信号から所望信号を**MAP推定法**により推定する．
ベイズの枠組みを利用し，入力信号の生成分布として正規分布より適切な分布をモデル化する．


# 1. システムモデル

## 音声強調モデル
 ここでは音声強調を以下の通りモデル化する．

    x = s + w

    y = G( x )

ここで，

    - s : 所望信号（原信号）のサンプル値
    - w : 雑音信号 のサンプル値
    - x : 入力信号（雑音混入音声）のサンプル値
    - y : 出力信号 のサンプル値
    - G : 音声強調処理
である．

上の式は，原信号 s に加法性雑音 w が重畳し入力音声が生成されることを示している．
下の式は，雑音混入音声 x から関数 G を使って音声 y を復元することを意味する．

## 音声信号モデル

音声の生成分布は，分析フレームを長く設定すると正規分布に近似できることが知られている．
これは中心極限定理からも理解できるだろう．

一方で，分析フレームを短くすると，音声の時変性が強くなり，
音声の状態（例えば，無音，子音，母音等）によって生成分布が異なる．
無音や音声パワーが小さい状態であれば，"0"に近いサンプル値が頻出するので，生成分布はラプラス分布に近づくと考えられる．
また子音や母音であればその生成分布は正規分布に近づくはずである．
ただし，分布の分散や平均は音声の状態に依存して時々刻々と変化する．

今，加法性雑音が正規分布に従って生成されると仮定すると，
入力信号の生成分布は 所望信号 s のサンプル値を中心に正規分布を形成する．
一般に，正規分布に従い生成される雑音は多くないが（白色雑音等），
ここでは雑音の生成分布を正規分布と仮定する．

> この仮定は信号処理分野において一般的であり，最小二乗の規範からも妥当だと考える．


# 2. ベイズ推定とMAP推定

## ベイズ推定
この信号復元モデルを**ベイズ推定**に当てはめると，以下の式が得られる．

    y = argmax_s  f(s|x)

    f(s|x) = f(x|s) f(s) / f(x)

ここで，

    - f(s|x) : **事後分布**．x を与えたときに s を得る確率分布．
    - f(x|s) : **尤度**．s を与えたときに x を得る確率分布．
    - f(s) : **事前分布**． s が生成される確率分布．
    - f(x) : **周辺分布**．x が生成される確率分布．

である．
ちなみに下の式がベイズの定理である．
上の式を噛み砕いて説明すると，
「入力信号 x が得られた場合に，所望信号 s として最もふさわしい値はどれかを探し，それを出力信号 y とする」ことを意味する．

## MAP推定

 x が  s と独立であると仮定すると，周辺分布 f(x) は s と独立した確率分布となり，
 y の推定に寄与しないことがわかる．したがって，実際の推定問題は f(x) を無視して，
 
    y = argmax_s  f(x|s) f(s)

と置くことができる．
この推定の枠組みを**MAP推定**と呼ぶ．
MAP推定における音声強調の仕組み噛み砕くと，
「入力信号 x が与えられたとき，取りうる s の値に対して，それぞれどれだけ s にふさわしいか（=尤度）を計算する．
さらに尤度を s の発生確率で補正し，尤度が最大となる s の値を最終的な出力信号 y をとして推定する．」
ことである．

ここで事前分布 f(s) は事前の s に対する知識から事前に決定しておく必要がある．
s の生成分布を一様分布と仮定すると，MAP推定は最尤推定に一致する．
また， s の生成分布がラプラスに従うと仮定して，STSA法により音声強調を行う手法(Joint MAP法)もある．
音声生成分布をどう設計するかは，大きな研究テーマとなっている．

### 補足１：最尤推定

事前分布は真となるデータの分布であり，事前に適切な分布を設定しなければ，推定精度が大きく劣化する．
しかし，実際の応用では真のデータの分布が未知であることが多く，
事前分布をモデル化することが困難である．

最尤推定では，周辺分布と事前分布を未知 = 定数 とみなす．
この場合，MAP推定における f(s) が無視できて

     y = argmax_s  f(x|s)
 
と変形される．
事前分布を定数とみなすことは，すなわち原信号の生成分布が一様分布であることと等しく，
あらゆる値が一様に発生しうることと同義である．
この仮定は現実的でないため，現在でも事前分布をどう仮定するか様々な研究がなされている．

### 補足２：Joint MAP 法 (MMSE-STSA法ベース)

MMSE-STSA法をベースに現実的な音声生成モデルを適用した手法．
MMSE-STSA法では原音声の振幅スペクトルの生成分布（=事前分布）が正規分布であると仮定していたが，
Joint MAP法では原音声の振幅スペクトルがラプラス分布に従って生成されると仮定した．
 
> MMSE-STSA : 
>
> スペクトルゲイン法に基づき，真の振幅スペクトルと推定振幅スペクトルの二乗誤差が最小となる
> スペクトルゲインを推定する手法．
> 音声スペクトルの生成分布（=事前分布）を正規分布と仮定し，
> 雑音スペクトルの生成分布（=尤度）もまた正規分布と仮定した．


### 補足３：可変音声分布に基づく Joint MAP法


Joint MAP法をベースに音声生成モデルを時変にした手法．
原音声の振幅スペクトルが**レイリー分布**に従って生成されると仮定し，入力音声からそのレイリー分布のパラメータを推定した．


# 3. 尤度の学習

MAP推定の式を再掲する．

    y = argmax_s  f(x|s) f(s)
    
    
MAP推定においては，事前分布 f(s) を設定する他に，尤度 f(x|s) を設計する必要があることがわかる．
この尤度は，音声強調した処理音声 y = G( x ) が原音声 s に近いほど高い値を示すよう設計すればよい．
ここでは，以下の式で尤度関数を定める．

     f(x|s) = exp(  -| G( x ) - s |^2 )

執筆中．
