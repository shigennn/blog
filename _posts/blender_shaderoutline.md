---
title: '【Blender】 Shaderでアウトラインをつくる'
excerpt: '検索ページでの説明文'
coverImage: '/assets/blog/dynamic-routing/blender_shaderoutline/1.png'
date: '2023-06-17'
ogImage:
  url: '/assets/blog/dynamic-routing/blender_shaderoutline/1.png'
tags:
  - 'Blender'
---

## 概要

簡単に雑なアウトラインをつけたいときがちょいちょいあるが、忘れることがあるのでまとめておく。

![](/assets/blog/dynamic-routing/blender_shaderoutline/1.png)
こんな感じ。

## アウトラインのつくりかた

我らがスザンヌで作っていきます。

### 1. アウトライン用のマテリアルをつくる

![](/assets/blog/dynamic-routing/blender_shaderoutline/2.png)
アウトライン専用のマテリアルを追加する。  
今回はメインのマテリアルが 1 つで良いので、1 つ目をアウトライン用として新規追加。

### 2. Solidify Modifier を追加

![](/assets/blog/dynamic-routing/blender_shaderoutline/3.png)
対象の Mesh に追加する。

![](/assets/blog/dynamic-routing/blender_shaderoutline/4.png)

- `Thickness` をマイナスに設定  
  この値がアウトラインの太さになる。

- `Normals > Flip` を有効にする
- `Materials > Material Offset` を `1` で設定したマテリアルのリスト番号に設定  
  今回は 2 番目なので、1 に設定。

### 3. アウトライン用のマテリアルを設定する

![](/assets/blog/dynamic-routing/blender_shaderoutline/5.png)

- Shader の設定  
  今回はとりあえずシンプルに Emisson のみにしました
- `Backface Culling` を ON にする
- `Blend Mode` を `Alpha Clip` にする
- `Shadow Mode` を `None` にする

Material の設定や、Solidify Modifier の Thickness を調整していい感じにする。  
完成。
