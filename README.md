# akashic-hover-plugin

<p align="center">
<img src="https://github.com/akashic-games/akashic-hover-plugin/blob/master/img/akashic.png"/>
</p>

**akashic-hover-plugin**は Akashic でマウスホバー可能なエンティティを利用することができるプラグインです。

実装例は `sample` ディレクトリ以下にあるサンプルプロジェクトを参照してください。

## 利用方法

[akashic-cli](https://github.com/akashic-games/akashic-cli)をインストールした後、

```sh
akashic install --plugin 5 @asmka/akashic-hover-plugin
```

でインストールできます。

上記の例では `--plugin` に `5` を指定していますが、これは任意の値で問題ありません。

本プラグインは、エントリポイント(`lib/index`)とプラグイン本体のスクリプトファイルが異なるため、
`game.json` を以下のように書き換える必要があります。
(相対パスとして認識させるため `./` が先頭に必要です。)

```javascript
...
	"assets": {
		...
		// スクリプトアセットとして追加
		"hover_plugin": {
			"type": "script",
			"path": "node_modules/@asmka/akashic-hover-plugin/lib/HoverPlugin.js",
			"global": true
		}
	},
...
	"operationPlugins": [
		{
			"code": 5,
			"script": "./node_modules/@asmka/akashic-hover-plugin/lib/HoverPlugin.js", // HoverPlugin.js のパスに書き換え
			"option": {
				"cursor": "pointer" // ホバー時のcursorを指定。省略時は "pointer"
			}
		}
	],
...
```

### コンテンツへの適用

`E#touchable`, `E#hoverable` プロパティが `true` を返すエンティティに対して `E#hovered`, `E#hovering`, `E#unhovered` トリガを発火させます。
このインタフェースは `HoverableE` として `src/HoverableE` に定義されています。

```typescript
export interface HoverableE extends g.E {
  hoverable: boolean;
  hovered: g.Trigger<HoveredEvent>;
  hovering: g.Trigger<HoveringEvent>;
  unhovered: g.Trigger<UnhoveredEvent>;
  cursor?: string;
}
```

`E#hovered` はホバー時、`E#hovering` はホバー移動時、`E#unhovered` はホバーが外れた時に発火されます。

### 既存のエンティティへの適用

`g.FilledRect` など既存のエンティティをホバー可能にするには以下のようにします。

```javascript
import * as hover from "@asmka/akashic-hover-plugin";

...
const rect = new g.FilledRect(...);
const hoveredRect = hover.Converter.asHoverable(rect);
hoveredRect.hovered.add((e) => {
	rect.cssColor = "#f00";
	rect.modified();
});
hoveredRect.hovering.add((e) => {
  rect.x -= e.prevDelta.x;
  rect.y -= e.prevDelta.y;
  rect.modified();
});
hoveredRect.unhovered.add((e) => {
	rect.cssColor = "#000";
	rect.modified();
});
```

## 仕様

### option

`game.json` の `operationPlugins` の節で `option` プロパティにオブジェクトを記述することで、プラグインのオプションを指定できます。
`option` は次の名前のプロパティ名と対応する値を持つオブジェクトです。

- cursor
  - 文字列 (省略可能。省略された場合 `"pointer"`)
  - ホバー時の cursor を指定。CSS に準拠。
- showTooltip
  - 真偽値 (省略可能。省略された場合 `false`)
  - ホバー時に tooltip を表示されるかどうか。
  - 表示内容は `HoverableE#title` 。

## ライセンス

本リポジトリは MIT License の元で公開されています。
詳しくは [LICENSE](https://github.com/akashic-games/akashic-hover-plugin/blob/master/LICENSE) をご覧ください。

ただし、画像ファイルおよび音声ファイルは
[CC BY 2.1 JP](https://creativecommons.org/licenses/by/2.1/jp/) の元で公開されています。
