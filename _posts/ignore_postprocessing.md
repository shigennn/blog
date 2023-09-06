---
title: '【Unity】 URPでPostProcessingをかけないレイヤーをつくる'
excerpt: 'UI Textと3D Objectそれぞれに対して、特定のレイヤーだけPost Processingをかけない方法'
coverImage: '/assets/blog/dynamic-routing/ignore_postprocessing/4.png'
date: '2023-06-18'
ogImage:
  url: '/assets/blog/dynamic-routing/ignore_postprocessing/4.png'
tags:
  - 'Unity'
  - 'C#'
---

## 概要

URP で動的に取り込んだアバターを使ったショート動画生成システムとそのシーンを作っていた時に、メッセージテキストなど特定のオブジェクトにだけ Post Processing をかけないようにしたかったが、そのやり方について詰まったのでまとめてみた。

- UI の TextMeshPro に対して PostProcess をかけない方法
- 特定のレイヤーに設定した 3D オブジェクトに PostProcessing をかけない方法
- uGUI 上ではレイヤーだけ設定し、実行時に動的にそのレイヤーから Post Processing を除外する方法

## UI の TextMeshPro に対して PostProcess をかけない

![](/assets/blog/dynamic-routing/ignore_postprocessing/1.png)

`Canvas` > `Render Mode` を `Screen Space - Overlay` に設定する。以上。  
いつも Canvas が Scene 上で見たときに大きすぎるのが嫌だったので、深く考えず `Screen Space - Camera` に設定したので少しハマった。

## 特定のレイヤーに設定した 3D オブジェクトに PostProcessing をかけない

URP のカメラには `Base` と `Overlay` の 2 種類のカメラタイプの設定があり、Camera Stack を設定することで、`Overlay` を `Base` に結合して描画されることができる。

### 手順詳細

#### 1. レイヤーの設定

Post Processing をかけたくないレイヤーを決め、特定のオブジェクトに設定する 。
（ここでは「Ignore Post Processing」とした）  
![](/assets/blog/dynamic-routing/ignore_postprocessing/2.png)

#### 2. Main Camera(Base)の設定

- `Render Type` を `Base` にする
- `Post Processing` を ON にする
- `Culling Mask` から「Ignore Post Processing」レイヤーを除外

#### 3. Overlay Camera の設定

新しいカメラを生成して、下記の設定をする

- `Render Type` を `Overlay` にする
- `Post Processing` を OFF にする
- `Culling Mask` に「Ignore Post Processing」レイヤーのみ設定

このとき、Transform、FOV などのカメラ設定を Main Camera と同じにして、Main Camera の子に設定しておくと、[CinemaChine](https://unity.com/ja/unity/features/editor/art-and-design/cinemachine) などでカメラを動かしたときも常に同じ View になる。

![](/assets/blog/dynamic-routing/ignore_postprocessing/3.png)

#### 4. Main Camera の Stack 設定

Main Camera の `Stack` に Overlay Camera を追加する

こうなる  
![](/assets/blog/dynamic-routing/ignore_postprocessing/4.png)

## uGUI 上ではレイヤーだけ設定し、実行時に動的にそのレイヤーから Post Processing を除外する

作っていたショート動画生成プロジェクトでは、ほぼすべてのシーンでこの設定が必要だったので、実行時に動的に Overlay Camera を生成して、上記設定がされるメソッドを作った。

```cs
using UnityEngine;
using UnityEngine.Rendering.Universal;

class IgnorePPlayer : MonoBehaviour
{
    [SerializeField] private Camera mainCamera;
    [SerializeField] private LayerMask ignorePPLayer;

    private void Start()
    {
        ConstructIgnorePPLayer(mainCamera, ignorePPLayer);
    }

    private void ConstructIgnorePPLayer(Camera mainCamera, LayerMask ignorePPLayer)
    {
        if (mainCamera == null || ignorePPLayer == 0)
        {
            return;
        }

        // Main Cameraの子に同位置で同じ設定を持つカメラオブジェクトを生成
        GameObject overlayCameraObject = new GameObject("OverlayCamera");
        overlayCameraObject.transform.parent = mainCamera.transform;
        overlayCameraObject.transform.localPosition = Vector3.zero;
        overlayCameraObject.transform.localRotation = Quaternion.identity;
        Camera overlayCamera = overlayCameraObject.AddComponent<Camera>();
        overlayCamera.orthographic = mainCamera.orthographic;
        overlayCamera.fieldOfView = mainCamera.fieldOfView;
        overlayCamera.cullingMask = ignorePPLayer;
        overlayCamera.clearFlags = CameraClearFlags.Nothing;
        // Overlay Cameraの設定
        UniversalAdditionalCameraData overlayCameraData = overlayCamera.GetUniversalAdditionalCameraData();
        overlayCameraData.renderType = CameraRenderType.Overlay;
        overlayCameraData.renderPostProcessing = false;

        // Main Camera設定を変更する
        mainCamera.cullingMask &= ~ignorePPLayer;
        UniversalAdditionalCameraData mainCameraData = mainCamera.GetUniversalAdditionalCameraData();
        mainCameraData.cameraStack.Add(overlayCamera);
    }
}
```

シーン上のゲームオブジェクトにアタッチすると動く（はず）。  
実際のプロジェクトでは、これとほぼ同じメソッドを static class を作ってモジュールとして外部のクラスから利用している。

## 参考

[https://forum.unity.com/threads/post-processing-textmeshpro-unity-bug.680512/](https://forum.unity.com/threads/post-processing-textmeshpro-unity-bug.680512/)
[https://forum.unity.com/threads/post-processing-textmeshpro-unity-bug.680512/](https://note.com/npaka/n/n856472efa5bc)
