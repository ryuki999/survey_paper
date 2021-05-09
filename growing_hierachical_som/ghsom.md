# GHSOM(Growing Hierarchical Self-Organizing Map)論文まとめ
## 概要
SOMは高次元データのクラスタリング・データマイニングのための有名な手法である。しかし、SOMは静的なモデルであることと、データの階層的な関係を表現する機能が限られていることの2つの問題がある。本論文でこれらの問題を改善するために、提案するGHSOMは独立したGSOMの構成された階層構造のニューラルネットワークモデルである。GHSOMの提案背景は入力データに応じて適切な構造に変わるモデルを提供することにあり、もう一つ利点として、データの階層的な関係を可視化し、直感的に解釈できることにある。

## 導入

## 先行・関連研究

The Hierarchical Feature Map[21]は

Tree-Structured SOM[22],[23]は

## GHSOM

### 原理

### 訓練アルゴリズム
![image](https://user-images.githubusercontent.com/50240567/117566030-51253f80-b0ef-11eb-9495-d8f9fa45e5c9.png)

## 実験

### 実験1:雑誌TIME

### 実験2:新聞記事Der Standardの2つの階層

## 結論
