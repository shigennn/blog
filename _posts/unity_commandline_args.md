---
title: '【Unity】 コマンドライン引数を扱う'
excerpt: 'アプリ実行時とEditorでコマンドライン引数を扱う方法'
coverImage: '/assets/blog/dynamic-routing/cover.jpg'
date: '2023-06-19'
ogImage:
  url: '/assets/blog/dynamic-routing/cover.jpg'
tags:
  - 'Unity'
---

## 背景

Unity を使って簡単な Windows デスクトップアプリを作っていたとき、アプリ実行時にコマンドライン引き数で userId などの情報を受け取る必要があった。  
また、開発時にビルド前に Unity Editor でテストする際の方法も調べたので、自分メモ用にまとめておく。

## 要約

- `System.Environment.GetCommandLineArgs();` でスペース区切りで入力したコマンドライン引数の文字列配列を取得できる
- 引き数で指定する記述方法に応じて、リストからどのように取得するか処理をつくる
- Editor で実行する場合、Unity Hub の `…` から設定できる

## 方法

### UnityEditor でコマンドライン引き数を設定する

まず、テストできるように Editor でコマンドライン引き数を設定をする。  
Project を起動するとき、あらかじめ`Unity Hub` の `プロジェクト` タブで起動するプロジェクトの `…` マークから、`コマンドライン引き数を加える` をクリックすると入力 window が開くので、ここに入力する  
![](/assets/blog/dynamic-routing/unity_commandline_args/1.png)1

### スクリプトからコマンドライン引き数を受け取る

`Environment.GetCommandLineArgs()` で全てのコマンドライン引数を受け取ることができる。

```cs
using System;

string[] args = Environment.GetCommandLineArgs();
Debug.Log(string.Join(", ", args));

// Editor側で a b c d と設定していた場合
// Console: a, b, c, d
```

自分の作っていたデスクトップアプリでは、コマンドライン引数の順に依存せず、引数名に対する値を受け取りたかったので、下記のようにした。

- コマンドライン引数

```bash
-userId xxxxx -password abcDEF1223
```

- スクリプト側

```cs
private string GetCommandLineArg(string arg)
{
    string[] args = Environment.GetCommandLineArgs();
    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == arg && args.Length > i + 1)
        {
            return args[i + 1];
        }
    }
    return null;
}


string userId = GetCommandLineArgs("-userId");
string password = GetCommandLineArgs("-password");
```

## 参考

[https://learn.microsoft.com/ja-jp/dotnet/api/system.environment.getcommandlineargs?view=net-7.0](https://learn.microsoft.com/ja-jp/dotnet/api/system.environment.getcommandlineargs?view=net-7.0)
