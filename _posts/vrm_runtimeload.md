---
title: '【Unity】 VRMをランタイムでロードする'
excerpt: 'VRM0.XをUniVRMを利用してランタイムロードするスクリプト'
coverImage: '/assets/blog/dynamic-routing/cover.jpg'
date: '2023-06-16'
ogImage:
  url: '/assets/blog/dynamic-routing/cover.jpg'
tags:
  - 'Unity'
  - 'C#'
---

## 概要

今まで Unity でアバターを扱う際には [Trlib](https://assetstore.unity.com/packages/tools/modeling/trilib-2-model-loading-package-157548?locale=ja-JP) を利用して FBX + Texture を動的にロードしていたが、 `Trilib` はシートライセンスであり、社内メンバーや客先にデモ共有する際にそれぞれ購入してもらう必要があった。  
動的にアバターをロードする要件であれば、 MIT ライセンスである `UniVRM` を利用して VRM ファイルを扱う方法でも実現できるため、調べて処理を作成した。  
結果として、Humanoid 形式のアバター設定に特に処理が不要な `VRM` の方が取り扱いやすかった。

## コード

### VRM をランタイムでロードする

using ディレクティブ

```cs
using UnityEngine;
using VRM;
using UniGLTF;
using VRMShaders;
using Cysharp.Threading.Tasks;
```

ローカルファイルからロード

```cs
public static async UniTask LoadVRMFromFile (string path)
{
    RuntimeGltfInstance instance = await VrmUtility.LoadAsync("", vrmBytes, new RuntimeOnlyAwaitCaller());
    instance.ShowMeshes();
}
```

byte 配列からロード

```cs
public static async UniTask LoadVRMFromBytes (byte[] vrmBytes)
{
    RuntimeGltfInstance instance = await VrmUtility.LoadBytesAsync("", vrmBytes, new RuntimeOnlyAwaitCaller());
    instance.ShowMeshes();
}
```

### RuntimeGltfInstance の解放

`RuntimeGltfInstance` はネイティブコレクションのため、アバターデータの使用後は `instance.Dispose()` で解放する必要がある。  
ただ、UniVRM 関連の処理をモジュールとして分離し、アバターの設定を扱うクラスから呼び出したかったので、アバター設定クラスが `UniGLTF` に依存しなくていいように、インスタンス解放用のクラス `AvatarInstance` を作った。

```cs
using System;
using UnityEngine;

public class AvatarInstance : IDisposable
{
    public GameObject Root { get; private set; }
    private Action onDispose;

    public AvatarInstance(GameObject instanceRoot, Action disposeAction)
    {
        Root = instanceRoot;
        this.onDispose = disposeAction;
    }

    public void Dispose()
    {
        onDispose?.Invoke();
        onDispose = null;
    }
}
```

AvatarInstance を利用して VRMLoad 処理をまとめる

```cs
using UnityEngine;
using VRM;
using UniGLTF;
using VRMShaders;
using Cysharp.Threading.Tasks;


public class VRMAvatarUtils
{

    public static async UniTask<AvatarInstance> LoadVRMFromBytesForURP (byte[] vrmBytes)
    {
        RuntimeGltfInstance instance = await VrmUtility.LoadBytesAsync("", vrmBytes, new RuntimeOnlyAwaitCaller());
        instance.ShowMeshes();

        return new AvatarInstance(instance.Root, () => instance.Dispose());
    }

    public static async UniTask<AvatarInstance> LoadVRMFromFileForURP(string path)
    {
        RuntimeGltfInstance instance = await VrmUtility.LoadAsync(path, new RuntimeOnlyAwaitCaller());
        instance.ShowMeshes();

        return new AvatarInstance(instance.Root, () => instance.Dispose());
    }
}
```

呼び出し元のクラスで、不要になったときに解放する

```cs
private AvatarInstance avatarInstance;

private void Start()
{
    avatarInstance = await VrmUtility.LoadVRMFromFile(GetVrmPath());
}


private void OnDestroy()
{
    avatarInstance.Dispose();
}
```

### URP で利用する

上記ロードでは、Shader が URP に対応していないものが設定されるため、正常に表示できない。  
URP で表示するために、Material を変更する処理を追加する必要がある。

```cs
using UnityEngine;
using VRM;
using UniGLTF;
using VRMShaders;
using Cysharp.Threading.Tasks;

public class VRMAvatarUtils
{
    public static async UniTask<AvatarInstance> LoadVRMFromFileForURP(string path)
    {
        RuntimeGltfInstance instance = await VrmUtility.LoadAsync(path, new RuntimeOnlyAwaitCaller());
        instance.ShowMeshes();
        ConvertMaterialsToURP(instance.Root);

        return new AvatarInstance(instance.Root, () => instance.Dispose());
    }

    private static void ConvertMaterialsToURP(GameObject model)
    {
        // ここにshader名を入れる
        string urpShaderName = "UniversalRenderingPipeline/Lit";

        Renderer[] renderers = model.GetComponentsInChildren<Renderer>();
        foreach (Renderer renderer in renderers)
        {
            Material[] materials = renderer.sharedMaterials;
            for (int i = 0; i < materials.Length; i++)
            {
                Material vrmMaterial = materials[i];
                Material unlitWFMaterial = new Material(Shader.Find(urpShaderName));
                unlitWFMaterial.color = vrmMaterial.color;

                // Copy main texture from the old material to the new one
                if (vrmMaterial.HasProperty("_MainTex"))
                {
                    Texture mainTexture = vrmMaterial.GetTexture("_MainTex");
                    unlitWFMaterial.SetTexture("_MainTex", mainTexture);
                }

                materials[i] = unlitWFMaterial;
            }
            renderer.sharedMaterials = materials;
        }
    }
}
```

### 参考

- [UniVRMPrograming ドキュメント - VRMUtility](https://vrm-c.github.io/UniVRM/ja/api/0_95_highlevel.html#id1)
