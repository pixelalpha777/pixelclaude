# かぶミニ作戦ノート

AIを使った株取引の補助システムです。楽天証券のかぶミニ（単元未満株）をNISA成長投資枠で売買する前提です。

## 仕組み

```
Sol（材料・ニュース調査） → Opus（チャート採点・最終判断） → 本人が発注
```

- 報告は日曜〜木曜の22時台に出し、翌営業日の寄付に向けた判断を返します。
- 予算：27,000円（`data/settings/main.json`）
- 判断は「買い / 売り / 見送り / 保有継続」のどれかです。
- 保有銘柄を記録しておくと、毎晩の分析で「売るか・持ち続けるか」も判断します。

### チャート採点の観点（2026-09-25 の試運転報告より）

- 株価 > 25日線 > 75日線 の上昇トレンドか
- 25日線の向き（上向き / 下向き）
- RSI（過熱していないか）
- MACD（デッドクロスしていないか）
- 直近の急騰・高値圏（押し目待ちにするか）
- 52週高値など、目先の上値の壁

## 定期実行

Claude Code のルーティン「かぶミニ作戦ノート 夜間分析」（`trig_01Vt6BMEHMw4tdp7cki54Qxk`）が、
日曜〜木曜の 22:10（日本時間）に新しいセッションを起動します。起動したセッションが材料調査・チャート採点・判断を行い、
結果をダッシュボードの共有DB `reports/<日付>` に書き込みます。終わるとスマホに通知が届きます。

## ディレクトリ構成

| パス | 内容 |
| --- | --- |
| `dashboard/index.html` | ダッシュボード（claude.ai Artifact「かぶミニ作戦ノート」のページ本体） |
| `data/settings/main.json` | 設定（予算） |
| `data/reports/<日付>.json` | 各日の報告（判断・推奨銘柄・候補とチャート点） |
| `data/holdings/` | 保有銘柄の記録（現時点では空） |

## データ形式

### `reports/<日付>`

| フィールド | 内容 |
| --- | --- |
| `date` | 報告日 |
| `forDay` | どの日の寄付向けか |
| `decision` | 全体判断（買い / 売り / 見送り / 保有継続） |
| `market` | 市況のまとめ |
| `picks[]` | 推奨銘柄：`action` `code` `name` `order` `reason` `refPrice` `shares` `stop` `target` |
| `candidates[]` | Solの候補：`code` `name` `price` `conviction`（Sol確信度） `score`（チャート点） `note` `chosen` |
| `holdingsCheck[]` | 保有銘柄の判定：`code` `price` `action` `stop` `target` |
| `comment` | Opusのコメント |

### `holdings/<id>`

`id` `code` `name` `shares` `price`（取得単価） `date` `status`（open / closed） `stop` `target` `sellPrice` `sellDate`

## ダッシュボードについて

`dashboard/index.html` は claude.ai の Artifact ランタイム（`claude.use("db")`）の共有DBを読み書きします。
claude.ai 上の公開ページ https://claude.ai/artifact/FRRpksbBjXam6Qr8fyWW89 がそのまま稼働中で、
`data/` 以下は 2026-09-25 時点の DB の内容を書き出したものです。

---

この報告はAIによる参考情報で、投資助言ではありません。売買の判断と結果はご自身の責任でお願いします。
NISA成長投資枠は売却しても枠が戻るのは翌年です。
