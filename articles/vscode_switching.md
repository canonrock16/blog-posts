---
title: "vscodeのワークスペースを色分けしてスイッチできる拡張が便利"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode"]
published: false
---

# はじめに
vscodeで複数のワークスペースを切り替えながら作業する時、今自分がどのワークスペースを開いているのかわからなくなったり、いちいちミッションコントロールを起動して画面を切り替えるのが面倒だったりしないでしょうか。
[vscode-workspace-switcher](https://marketplace.visualstudio.com/items?itemName=sadesyllas.vscode-workspace-switcher)を導入すれば、こんな感じで解決します。
![switching.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/233586/4271af8e-93b3-aef3-7c02-953d23657944.gif)
各ワークスペースの左端のバーの色を替えてワークスペースを識別しやすくし、ショートカットキーでワークスペースを切り替えられるようになります。

# 導入と設定
1.[vscode-workspace-switcher](https://marketplace.visualstudio.com/items?itemName=sadesyllas.vscode-workspace-switcher)をインストール
2.ターミナルからvscodeを起動するコマンド`code`をインストール。[参考記事](https://qiita.com/naru0504/items/c2ed8869ffbf7682cf5c)
3.切り替えたいフォルダの`.code-workspace`ファイルを作成。vscode上でワークスペースに追加したいフォルダを開いた状態で「メニュー→ファイル→名前を付けてワークスペースを保存」を選び、任意のフォルダに保存する。
4.vscodeの設定ファイル`setting.json`を開き、以下を追加

```setting.json
  "vscodeWorkspaceSwitcher.paths": ["/path/to/3で.code-workspaceファイルを保存したフォルダ/"],
```
この時点で、vscode左端のバーのvscode-workspace-switcherのアイコンや、`Ctrl-w`で各ワークスペースを切り替えられるようになっているはずです。
5.各`.code-workspace`ファイルに以下を追記。色の指定を変更すれば任意の色にできます。

```.code-workspace
{
  "settings": {
    "workbench.colorCustomizations": {
      "activityBarBadge.background": "#4aa1c4",
      "activityBar.background": "#4aa1c4",
      "activityBar.foreground": "#000000"
    }
  }
}
```
