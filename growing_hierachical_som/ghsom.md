# GHSOM(Growing Hierarchical Self-Organizing Map)論文まとめ

## 概要
SOMは高次元データを2次元空間に写像することで、マップ上のクラスタを視覚的に認識できるようになる。しかし、SOMには以下の2つの問題点がある。
* ニューロン数やマップ構造が固定のため、入力データに応じたモデル構造を決定するが難しい
* 入力データ間の階層的な関係が反映されず、全てのデータが同一マップ上で表示されるため、識別が困難

上記のSOMの問題を解決するために、本論文ではGHSOMを提案する。本論文でこれらの問題を改善するために、提案するGHSOMは独立したGSOMで構成された階層構造のニューラルネットワークモデルである。GHSOMでは入力データに応じて適切なモデル構造に変わり、データの階層的な関係を可視化し、直感的に解釈することが可能になる。

本研究では、GHSOMの有用性を2つの文書アーカイブの整理タスクに適用して示す。最初の実験は、TIME Magazineに基づくものである。このデータは、国際政治から社会的なゴシップまで様々なトピックをカバーするTIME Magazineの420の記事から構成されている。2つ目の実験は、オーストリアの日刊紙Der Standardから10,000以上の記事を集めた、より大規模なデータになっている。

## 先行・関連研究
自己組織化マップをデータマイニングに活用するために、様々な改良が提案されている。特に、クラスタ間およびクラスタ内の類似性の識別に取り組まれた、U-Matrix [15]、Adaptive Coordinates、Cluster Connections [16]などのアプローチは、SOMにおけるクラスタ構造の検出と可視化に重点を置いた技術である。これらの手法では，隣接するユニット間の距離を分析したり，モデルベクトル適応の効果を2次元の出力空間に反映させたりする．同様のクラスタ情報は，[17]，[18]で紹介されたLabelSOM法でも得ることができる．LabelSOM法では，さまざまなユニットの特性を，マッピングされた入力パターンの特徴から決定する．同じ特徴を持つユニットをグループ化することで，SOMの出力空間内のクラスターを特定することができる．

<!--また，自己組織化場(Self-Organizing Field)[19]やGenerative Topographic Mapping（GTM）[20]のように，格子構造へのマッピングに起因する問題を解決し，出力空間として滑らかな多様体を提供するアーキテクチャの改良もある．しかし、上記の方法は、データに内在する階層構造を検出したり、ネットワークのサイズを調整することはできません。-->

### 階層化に関する研究
* **The Hierarchical Feature Map**[21]は、モデルの学習前に、層の数や各層のマップサイズも含めてモデル構造を定義し、固定する。これは、階層構造を持ったSOMを学習し、1つのユニットにマッピングされたデータは、そのユニットに割り当てられた下位のマップでさらに詳細なレベルで表現される。しかし、このモデルはデータの階層構造を反映しているのではなく、単にデータを階層的に表現しているに過ぎない。

* **Tree-Structured SOM**[22],[23]は、SOMを階層的に改良したモデルである。しかし，このモデルでは，ユニットをツリー状に構成することで，勝者選択時の計算速度を向上させることに重点が置かれている。このモデルでは、入力空間を階層的に分解することはできず、入力パターンは再び1つの平面のSOMで構成される。

### マップのサイズに関する研究
* **Incremental Grid Growing[4]**
  * マップの境界に新しいユニットを追加することができる。さらに、マップのユニット間の接続は、それぞれのモデルベクトルの類似性に基づいて、いくつかの閾値設定に従って確立および削除することができる。これにより、いくつかの分離された不規則な形状のマップ構造になる可能性がありうる。
* **Growing Grid[5]**
  * 学習プロセス中にユニットの行と列を追加し、最初は2×2のユニットで構成されたSOMからスタートし、どこに新しいユニットを挿入するかの決定は、各ユニットに対する何らかの指標（例えば勝者カウンタ）の計算によって決定される。
* **Growing SOM[6]**
  * マップの成長を制御するためにSpread Factoerを使用する。手動でパラメータを調整することにより、マップの広がりを調整することが出来る。
* **Hypercubical SOM[24]**
  * SOMを2次元以上に成長させることができるため、視覚的な解釈性を犠牲にしても、データの表現力を向上させることができる。

本論文著者は、これらの提案モデルについて次の問題点を挙げている。上記の自己組織化マップの改良型は、入力データ数が不均衡に多いユニットの近傍に新しいユニットを追加することで、入力パターンをマップ全体に均等に分布させることに主眼が置かれていると言える。また、これらのモデルは、特定の領域にマッピングされた入力データの数ではなく、全体の量子化誤差で表現されるため、あるレベルの詳細な表現の概念を反映できておらず、データの本質的な階層構造も考慮されていない。

## GHSOM
![image](https://user-images.githubusercontent.com/50240567/117566030-51253f80-b0ef-11eb-9495-d8f9fa45e5c9.png)

### 初期設定と全体のネットワーク制御
あるユニットiの平均量子化誤差は、ユニットの参照ベクトルm_iと、n_C個の入力ベクトル集合C_iの要素であるx_jを用いて、以下のように表せる。

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+mqe_i%3D%5Cfrac%7B1%7D%7Bn_C%7D+%5Ccdot+%5Csum_%7Bx_j+%5Cin+C_i%7D%7C%7Cm_i-x_j%7C%7C%2C+n_C%3D%7CC_i%7C%2CC_i+%5Cneq+%5Cphi" 
alt="mqe_i=\frac{1}{n_C} \cdot \sum_{x_j \in C_i}||m_i-x_j||, n_C=|C_i|,C_i \neq \phi">

GHSOMは訓練の最初に、第0層のユニットの平均量子化誤差mqe_0を以下のように計算する。ここで、n_Iは入力データ集合Iのデータ数である。

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+mqe_0%3D%5Cfrac%7B1%7D%7Bn_I%7D+%5Ccdot+%5Csum_%7Bx_i+%5Cin+I%7D%7C%7Cm_0-x_i%7C%7C%2C+n_I%3D%7CI%7C" 
alt="mqe_0=\frac{1}{n_I} \cdot \sum_{x_i \in I}||m_0-x_i||, n_I=|I|">

上記の平均量子化誤差はモデルの階層化やマップの成長に際に使用する。階層化の際にはすべてのユニットで以下の式を満たすように訓練される。

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+mqe_i+%3C+%5Ctau_2+%5Ccdot+mqe_0" 
alt="mqe_i < \tau_2 \cdot mqe_0">

また、上記式の代わりに、以下の量子化誤差が使われる場合もある。

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+qe_i+%3D+%5Csum_%7Bx_j+%5Cin+C_i%7D+%7C%7Cm_i-x_j%7C%7C" 
alt="qe_i = \sum_{x_j \in C_i} ||m_i-x_j||">

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+qe_i+%3C+%5Ctau_2+%5Ccdot+qe_0" 
alt="qe_i < \tau_2 \cdot qe_0">

GHSOMは第0層のマップの下に初期サイズ2x2のGSOMを設定し、ランダムな参照ベクトルとすることで初期化される。

### 成長中のSOMの学習・成長過程
* 全てのマップ中のユニットに対して、qeを求める
* あるマップ中で最もqeの大きいユニットeと、その近傍で、eとの参照ベクトルの誤差が一番大きいユニットを探索する

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+e+%3D+%5Cargmax_%7Bi%7D+%28%5Csum_%7Bx_j+%5Cin+C_i%7D+%7C%7Cm_i+-+x_j%7C%7C%29%2C+n_C%3D%7CC_i%7C%2C+C_i+%5Cneq+%5Cphi" 
alt="e = \argmax_{i} (\sum_{x_j \in C_i} ||m_i - x_j||), n_C=|C_i|, C_i \neq \phi">

<img src=
"https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+d+%3D+%5Cargmax_%7Bi%7D+%28%7C%7Cm_e+-+m_i%7C%7C%29%2C+m_i+%5Cin+N_e%0A" 
alt="d = \argmax_{i} (||m_e - m_i||), m_i \in N_e
">

* eとdの間にユニットを挿入する。尚、挿入ユニットの初期値は、挟み込んでいる近傍ユニットの平均値である。
* 

### 成長過程の終了
### 全体のマップ適応による階層的な成長
### GHSOMの特徴の分析

## 実験

### 実験1:雑誌TIME

### 実験2:新聞記事Der Standardの2つの階層

## 結論
