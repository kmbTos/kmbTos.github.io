# 幻奏弾幕譚 〜 Phantasmal Resonance

音楽解析駆動の東方風二次創作・ボス戦専用縦スクロール弾幕STG。
単一HTMLファイル(index.html)で動作。ビルド不要。

## 遊び方

`index.html` をブラウザ(Chrome/Edge推奨)で直接開くだけで動きます(file://可)。
サーバ経由でも可:

```
python -m http.server 8124 --directory music-danmaku
→ http://localhost:8124/
```

古い版がキャッシュされていると動かないことがあるので、
タイトル画面に「build v1.2」が出ていなければ Ctrl+F5 で再読込。

1. MP3(ogg/wav/m4aも可)をドラッグ&ドロップ、または「内蔵デモ曲でプレイ」
2. 曲を自動解析(ビート検出・BPM推定・盛り上がりセクション分割)
3. スタート。曲が終わるまで生き残ればクリア

## 操作

| キー | 動作 |
|---|---|
| ↑↓←→ / WASD | 移動 |
| Shift | 低速移動(当たり判定表示) |
| X / Space | ボム(弾消し+ボスにダメージ) |
| Esc | ポーズ |

ショットは自動連射(最初からフル装備・オプション4基はボスを緩追尾)。

## 音楽→弾幕の対応

- **ビート同期**: 事前解析したビートグリッドに合わせて発射
- **重低音キック** → 全方位リング弾
- **ビート** → 自機狙い扇弾(kunai)
- **高音アクセント** → 星屑弾
- **曲のセクション**(エネルギー量子化+メディアンフィルタで分割) → スペルカード切替
  - tier 0〜3(静→サビ)で密度・速度・パターン族(spiral/flower/wall/rain/stars)が変化
  - スペル名は tier に応じた符名+語彙から自動生成
- ボスHPはセクション長に比例。時間内に削り切るとスペルカード取得ボーナス

## 実装メモ

- 解析: 自前radix-2 FFT(win1024/hop512)でスペクトラルフラックス→
  自己相関でBPM、位相探索でビートグリッド、4秒窓RMS量子化でセクション
- 再生中はAnalyserNodeでBASS/MID/TREBLEをライブ取得(演出・弾色・補助パターン)
- 内蔵デモ曲はFloat32Arrayへの直接サンプル合成・モノラル22.05kHz
  (OfflineAudioContextの大量ノードは環境によって極端に遅いため不使用)
- 長い処理のyieldはsetTimeoutでなくMessageChannel
  (非表示ページではsetTimeoutが1秒に間引かれ解析が固まって見えるため)
- ページ非表示中はrAFが止まるので自動ポーズ(音楽だけ進む desync 防止)
- デバッグ: `window.G` (state/music/bullets/boss/player/spells/songPos)
