# HighLowRush 完全仕様書
version: 1.0-spec
target: implementation AI
priority: correctness > brevity > human readability
language: ja-JP only

---

## 0. APP_META

```yaml
app:
  name_ja: "ハイローラッシュ"
  name_en: "HighLowRush"
  platform: iPhone_only
  iPad: unsupported
  min_iOS: 17
  orientation: portrait_only
  theme: light_only
  dynamic_type: disabled
  offline_gameplay: true
  notifications: none
  monetization:
    price: free
    IAP: none
    ads: banner_only
  analytics: none
  crash_reporting: anonymous_only
  localization: ja-JP_only
```

Design target:
- 子どもでも理解しやすい。
- 学生・大人が友人同士で使っても幼すぎない。
- 明るい、ポップ、ゲームらしい。
- UI文言は基本ひらがな中心。
- HIGH/LOWという英語表示はゲーム操作に使わず、大きな↓/↑を使う。
- 背景は画面端まで描画可。主要操作UIはSafe Area内。
- 小さいiPhoneでもゲーム画面はスクロールさせない。
- 画面が狭い場合の縮小優先順位:
  1. 余白/間隔縮小
  2. 補助情報縮小
  3. 最後にカード/主要ボタンを最小限縮小
- 主要操作性とタップ領域を優先。
- iOS標準の左端スワイプ戻るは無効。戻る操作は明示ボタンのみ。
- アプリ内に「終了」ボタンは置かない。
- 起動時/画面遷移は軽量。不要な長い演出は避ける。

---

## 1. IMPLEMENTATION_POLICY

```yaml
implementation:
  architecture: "既存プロジェクト/既存アプリの流儀を優先"
  requirement:
    - UIとゲームロジックは分離
    - 保存/広告/共有/ゲームモードは疎結合
    - 将来の機能追加で大改修しにくい構造
    - 過剰な抽象化は禁止
  persistence_impl: "実装AIが既存構成に合わせて選定"
  image_generation: false
  image_assets: "初期プロトタイプでは仮UI/プレースホルダーを使用し、動作確認後に別途提供される完成アセットへ差し替える"
```

重要:
- UserDefaults/SwiftData等の具体技術は固定しない。
- ユーザーから見える挙動が本仕様通りであることを優先。
- 通常アップデートでは保存データを維持。
- 保存データ破損時:
  - クラッシュしない。
  - 読めない項目だけ初期値へフォールバック。
  - 正常な項目は保持。
  - ユーザーへの特別通知は不要。

---

## 2. GLOBAL_CARD_RULES

```yaml
rank_order: [A,2,3,4,5,6,7,8,9,10,J,Q,K]
strength:
  A: weakest
  K: strongest
suit_strength: none
```

Suit visual:
- ♥ ♦ = やわらかい赤
- ♠ ♣ = やわらかい濃いグレー
- 真紅/純黒は避ける。

Card face:
- 少しアイボリー寄りの白。
- 細いソフトな濃灰枠。
- やや角丸。
- 右下方向に非常に薄い影。
- フリップ中は影も少し変化。
- J/Q/Kに人物絵を使わない。
- 中央:
  - A: 大きいスート1個
  - 2-10: 大きい数字 + 下にスート1個
  - J/Q/K: 大きい文字 + 下にスート1個
- 中央数字/文字は大きく、太く、丸みのある高可読フォント。
- スートは中央文字より少し小さい。
- コーナーインデックスは左上/右下のみ、小さめ。
- 中央文字/スートにアウトラインなし。
- 表面パターンなし。

Card back:
- 両モード共通。
- アプリテーマ色を少し濃くした単色ベース。
- 中央に白抜きアプリロゴ/シンボル。
- ♠♥♦♣の小さな規則的反復パターン。
- 反復スートは全て同系色。赤黒分けしない。
- 細い二重枠。
- パターンは細かく控えめ。

---

## 3. CARD_ANIMATION_COMMON

```yaml
flip:
  axis: horizontal_visual_flip
  duration: 0.2-0.4s
  back_visible_briefly: true
  scale_animation: false
slide_from_deck:
  duration: 0.1-0.2s
  rotation: none
  easing: slight_deceleration_near_end
```

- 山札→現在カード位置へ軽くスライド後、フリップ。
- フリップ速度は両モード共通。
- 表示後のカードは常に正立、固定サイズ、ランダム傾きなし。
- フリップ完了後の不要な待機は入れない。
- ただし各モード固有のフィードバック表示中は入力ロック。

Single-player directional flip:
- ↑選択: 右方向へめくる印象
- ↓選択: 左方向へめくる印象
- DRAW時も選択した方向を維持

---

## 4. GLOBAL_AUDIO_HAPTICS

Settings:
```yaml
sound:
  label: "おと"
  initial: on
vibration:
  label: "しんどう"
  initial: on
toggle_ui:
  type: two_choice_button
  order: ["つける","けす"]
  equal_width: true
BGM: none
```

Rules:
- 設定切替時にテスト音/テスト振動は出さない。
- 実際の次操作から反映。
- 効果音はiPhoneの通常の音量/マナーモードに従う。
- 強制再生しない。
- ボタンタップ: 短い効果音 + 軽い振動（各設定ON時）。
- カードフリップ: 短い音 + 軽い振動（各設定ON時）。
- single正解/同じ/不正解は毎回音あり。ただし短く控えめ。
- 「おなじ！」は正解音と別のニュートラル専用音。
- ゲームオーバーは通常より少し強めの振動。
- 新記録も通常より少し強めの振動 + 専用短音。
- single再シャッフル開始時に短い専用シャッフル音。
- 「○しゅうめ！」専用音はなし。
- multiplayer最後の1枚は通常フリップと同じ振動。

---

## 5. HOME

Layout:
- 明るいポップな静止背景。
- 背景に薄い♠♥♦♣とカードシルエット。
- 背景アニメーションなし。
- 上中央: 大きいアプリ名/ロゴ。
- キャッチコピー:
  `つぎのカードは うえ？ した？`
- 大きい同サイズボタンを縦配置:
  1. `ひとりであそぶ`
  2. `みんなであそぶ`
- 各モードボタンに分かりやすいアイコン。
- 右上: 設定ギア。
- ホームに「遊び方」直接導線は置かない。
- 機能説明文はキャッチコピー以外置かない。
- ボタン押下時: 短い縮小/プレス反応。

Single best summary:
- `ひとりであそぶ`直下に小さく表示。
- トロフィーアイコン。
- `さいこうきろく ○れんしょう`
- 初期値も `さいこうきろく 0れんしょう`
- 999超は画面上 `999+れんしょう`

First launch:
- 初回起動直後はチュートリアルを出さずホームを表示。
- 初めて `ひとりであそぶ` を押した時のみsingle初回チュートリアル。
- multiplayer初回チュートリアルなし。

---

## 6. SINGLE_PLAYER_RULES

```yaml
deck:
  cards: 52
  joker: false
  depletion: physical_without_replacement
first_card:
  auto_draw: true
  random: true
  A_or_K_allowed: true
wrong_guess:
  result: immediate_game_over
draw_same_rank:
  streak_change: 0
  continue: true
correct:
  streak_increment: 1
manual_exit:
  updates_best: false
app_killed:
  session_lost: true
  updates_best: false
temporary_background:
  resume_session: true
```

- 初回カードは自動ドロー。
- 1枚目も履歴上「出たカード」として扱う。
- 1枚目表示後、残り `あと51まい`。
- 新しく引いたカードが次の基準カードになる。
- 同ランクは `おなじ！`、連勝据え置きで続行。
- 1回でも不正解で終了。
- 明示pauseボタンなし。
- 難易度なし。
- オンラインランキングなし。
- 統計は最高記録のみ。
- probability/history設定に関係なくBESTは1本。
- アプリ強制終了/クラッシュ後はホームから開始し途中復元しない。

---

## 7. SINGLE_DECK_CYCLE

When deck exhausted:
1. 最後のカードまで通常通り使用。
2. 山札表示を一度消す。
3. 0.5-0.8sの短いシャッフル演出。
4. 専用シャッフル音（sound ON時）。
5. `山札を まぜなおしたよ！` を0.6-0.8s表示。
6. 次に `○しゅうめ！` を短く強調表示。
7. 新しい山札を軽くフェードイン。
8. プレイ再開。

Cycle rules:
- streakは継続。
- 前周期の最後の現在カードを次周期の基準として引き継ぐ。
- 新周期の山札はその「同一物理カード」を除外した51枚で開始。
- probabilityは基準カード + 新しい51枚で即再計算。
- historyは周期ごとにリセット。
- 引き継いだ基準カードは新周期履歴上「現在カード」として扱う。
- `1しゅうめ`は表示しない。
- 2周期目以降のみ `2しゅうめ`, `3しゅうめ`...
- 3周期目以降も毎回同じ「まぜなおしたよ→○しゅうめ！」演出。
- 周回表示上限: 99。100以上は `99+しゅうめ`
- 内部値は正確に保持。

---

## 8. SINGLE_GAME_UI

Main hierarchy:
1. 現在連勝: 大きく `○れんしょう！`
2. `さいこうきろく ○れんしょう`
3. 2周期目以降: 小さく `○しゅうめ`
4. 現在カード
5. 山札
6. 残り枚数
7. ↓ / ↑ 操作

Streak:
- 初期 `0れんしょう！`
- テーマ色。
- 正解時、数字が軽くpop。
- DRAW時はアニメーションなし。
- 999超の画面表示は `999+れんしょう！`
- 内部値は正確に保持。

Best:
- 常時表示。
- ラベル `さいこうきろく`
- 現在連勝より小さい濃灰。
- 999超の画面表示は `さいこうきろく 999+れんしょう`

Deck:
- 現在カードの真下中央。
- 現在カードより少し小さい。
- タップ不可。
- タップ可能に見える押下表現を付けない。
- 複数枚束のレイヤー表現。
- レイヤーずれは右下方向。
- 残り枚数に応じて厚み変化。
- 実残り1枚でも最低2-3枚分の束感を維持。
- 実枚数はテキスト `あと ○まい` で正確表示。

Arrow buttons:
- 山札の下、横並び。
- 左: ↓
- 右: ↑
- 大きい横長角丸。
- visualよりtap targetを少し広く。
- 長押しも1入力。
- 同時に近いtapは最初の入力だけ受理、他方即無効。
- ↓: blue family
- ↑: pink-orange family
- probability OFF: 大きい矢印のみ。
- probability ON: 大矢印 + 下に小さい%。
- ON/OFFでボタンサイズ固定。
- 押下直後に選択側を短く強調。
- フリップ中/フィードバック中は無効。
- 初回カード完全表示までは無効。

Top navigation:
- 左上: `ホーム`
- 押すと確認:
  `ホームにもどると、いまのゲームは おわるよ`
  buttons: `ホームにもどる / やめる`
- 確認して戻ると手動終了。BEST更新なし。
- history画面から直接ホームには戻れない。

---

## 9. SINGLE_PROBABILITY

Setting:
- label: `かくりつをみる`
- 保存: true
- homeからsingle開始時、前回保存値を使用。
- ゲーム中変更不可。
- resultからsettings変更 → 次retryに反映。

Calculation:
- 実際の残り山札から計算。
- ↓ / same / ↑ の3分類。
- integer %。
- 表示値3つの合計は必ず100%になるよう丸め調整。
- より有利な選択肢を自動強調しない。
- 枚数内訳は出さない。

UI:
- ↓% = 左ボタン内。
- ↑% = 右ボタン内。
- center = `おなじ ○%`
- center色はneutral gray。
- probability OFF:
  - center probability areaは完全に畳む。
  - ボタンサイズは変えない。
- remaining countはprobability OFFでも常時表示。

---

## 10. SINGLE_HISTORY

Setting:
- label: `でたカードをみる`
- 保存: true

Game UI:
- ON時のみ `でたカード` button表示。
- OFF時は完全に非表示。
- 開くとゲーム入力停止。

History screen:
- title: `でたカード`
- top-left: `もどる`
- home shortcutなし。
- full-screen。
- current cycle only。
- 旧cycleへアクセス不可。
- probability表示なし。
- 残り `あと ○まい` 表示。

Matrix:
- columns exact: ♦ → ♥ → ♠ → ♣
- rows exact: A,2,3,4,5,6,7,8,9,10,J,Q,K
- 52セルを最初から全表示。
- 未出: 通常表示。
- 出たカード: 半透明。
- fadeはスート色も含む。
- checkmarkなし。
- 現在基準カード:
  - 通常opacity
  - 太いborder
  - `いま`ラベルなし。
- suit headerはシンボルのみ。

---

## 11. SINGLE_FEEDBACK

Correct:
- text: `せいかい！`
- bright green
- bold rounded
- iconなし
- shadowなし
- current card近辺
- duration: 0.4-0.6s
- 表示中操作無効

Draw:
- text: `おなじ！`
- gray
- same typography
- duration: 0.4-0.6s
- 表示中操作無効
- streak変化なし

Wrong:
- text: `ざんねん！`
- soft red
- bold rounded
- icon/shadowなし
- background dimなし
- losing cardは表示されたまま
- duration: 0.8-1.2s
- 全操作無効
- 通常より少し強いhaptic
- 後に短いfadeでresultへ

---

## 12. SINGLE_TUTORIAL

Trigger:
- app初回起動時ではなく、初めて `ひとりであそぶ` を押した時のみ。
- 1画面。
- title: `あそびかた`

Content:
- ↑ = 次のカードが大きい
- ↓ = 次のカードが小さい
- rank visual:
  `A → 2 → 3 → … → Q → K`
- draw example:
  `7 → 7 は おなじ！ そのまま つづくよ`
- end/goal:
  `ちがったら そこで おわり！ なんれんしょう できるかな？`
- auto-shuffle説明なし。
- probability/history説明なし。
- button: `あそぶ！`

In-game:
- rule/help buttonなし。

Settings > 遊び方:
- 初回tutorialより詳しい。
- 上部tab:
  `ひとりであそぶ | みんなであそぶ`
- single詳細にはprobability/history/山札使い切り後も補足。
- multiplayer説明も含む。
- homeからの直接導線なし。
- banner広告あり。

---

## 13. SINGLE_RESULT

Do NOT show:
- losing card
- `ざんねん！`
- cycle count
- date

Show:
- main large: `○れんしょう！`
- below: `さいこうきろく ○れんしょう`
- 0でも `0れんしょう！`

New record:
- immediately above main:
  `きろくこうしん！`
- gold
- tiny sparkle
- slightly stronger haptic
- dedicated short celebration sound

Buttons:
- primary large: `もういちど`
- below, 3 smaller horizontal:
  - icon + `シェア`
  - gear + `せってい`
  - home + `ホーム`

Behavior:
- retry = 即新規game開始。現在保存settings使用。
- result settings = resultへ戻る。変更は次retryから。
- banner広告あり。

---

## 14. SINGLE_SHARE_RESULT

Type:
- image + text
- image dynamic generated in app
- 1080x1080
- one fixed design template

Background:
- bright pop app-theme
- faint ♠♥♦♣ pattern
- card illustrationなし

Content:
- main centered actual result.
- BEST below, smaller.
- new record時のみ上に `きろくこうしん！` gold.
- app name/logo bottom-left.
- QR bottom-right.
- QR target = App Store app page.
- QR自身は角丸にしない。
- white rounded plate + generous quiet zone。
- near QR:
  `アプリストアであそべるよ`

Do NOT include:
- losing card
- title/rank/mode name
- date
- cycle count
- catchphrase

999+:
- app UIは999+表示だが、share imageは実数表示。
- 桁が長い場合のみ文字サイズを自動縮小。

---

## 15. MULTIPLAYER_CONCEPT

```yaml
mode: shared_one_device
app_judgement: none
score: none
win_loss: none
answer_input: none
first_card_auto: false
draw_trigger: tap_deck_only
orientation: portrait
keep_screen_awake_during_game: true
```

- 参加者は口頭で上/下を言う。
- アプリはカードをめくるだけ。
- 正解/不正解/DRAW判定表示なし。
- ↑/↓操作UIなし。
- 最初のカードも山札タップで引く。
- 最初のtap前説明文なし。
- 全画面tapではなく山札tapのみ。
- flip後0.3-0.5s入力ロック。
- 最新1枚のみ表示。
- 前カードは完全に消える。
- current card中央大。
- deck下。
- cardサイズ固定・正立。
- game titleなし。
- game中は自動ロック防止。
- game終了時はシステム自動ロック挙動へ戻す。
- 一時backgroundは状態保持。
- app kill/closeでsession終了、次回新deck。
- home returnでsession保存しない。
- multiplayer初回tutorialなし。

---

## 16. MULTIPLAYER_PRESET_SCREEN

Home `みんなであそぶ` → 必ず事前設定画面。

title:
`あそびかたをきめる`

Saved across sessions:
- エンドレス
- ジョーカー
- でたカードをみる
- かくりつをみる

Initial defaults:
```yaml
endless: off
joker: off
history: off
probability: off
```

Start button:
`これであそぶ`

Game中にもsettingsへ入れる。

Mid-game settings confirmation:
Normal:
`せっていをかえると、いまの山札はさいしょからになるよ`

Endless:
`せっていをかえると、いままでのカードはきえるよ`

buttons:
`せっていをかえる / やめる`

設定変更後:
- current deck/history/current cardを全reset。
- `これであそぶ`で新規開始。
- first cardは自動で引かない。

---

## 17. MULTIPLAYER_NORMAL

Deck:
- joker OFF: 52
- joker ON: 54
- without replacement
- first tap後:
  - joker OFF: `あと51まい`
  - joker ON: `あと53まい`

Deck visual:
- 残りに応じ厚み減少。
- 最後1枚でもtapしやすい最低厚みを維持。

At 0:
- deck消える。
- `あと0まい`は表示しない。
- probability消える。
- current/last cardは中央に残す。
- `山札をまぜる` button表示。
- history ONなら `でたカード` は使える。
- home usable.
- 0枚時homeは確認なし。
- 特別な `ぜんぶめくった` messageなし。

Before 0 home:
`ホームにもどると、いまの山札はなくなるよ`

Shuffle:
- normal modeではいつでも `山札をまぜる` を使える。
- top area。
- shuffle icon + text。
- midway confirmation:
  `山札をまぜなおす？`
  buttons: `まぜなおす / やめる`
- 設定保持。
- current card削除。
- history即reset。
- count即52/54へreset。
- 0.5-0.8s shuffle animation。
- animation中全入力無効。
- shuffle後auto flipなし。
- fresh deckを表示し、tapでfirst draw。
- deck0後shuffleも同じ処理。

---

## 18. MULTIPLAYER_ENDLESS

```yaml
draw_model: independent_full_pool_each_flip
same_physical_card_repeat: allowed
immediate_repeat: allowed
joker_repeat: allowed_if_enabled
remaining_count: none
deck_visual_thickness: constant
```

UI:
- deck近くに大きい `∞`
- historyにも `∞`
- `山札をまぜる`なし。
- separate history resetなし。
- historyはsession中累積。
- home確認:
  `ホームにもどると、いままでのカードはきえるよ`

Probability:
- full pool基準。
- 過去drawで変化しない。
- joker ON時はnormal rankについて↓/same/↑ + joker probability。

---

## 19. MULTIPLAYER_JOKER

Setting label:
`ジョーカー`

Rules:
- rank/strengthなし。
- social interpretationはユーザーに委ねる。
- center cardと同サイズ。
- 特別animation/soundなし。
- Joker後の次tapは普通に進む。

Probability:
- current cardがJoker:
  - probability area一時完全非表示。
  - 次のnormal cardで復帰。
- current normal card + joker ON:
  - ↓ / same / ↑ + separate Joker probability。

History:
- 52 matrixとは別。
- matrix下にgrouped Joker box。
- normal:
  `ジョーカー 0/2`, `1/2`, `2/2`
- endless:
  `ジョーカー ×N`
- Joker個体1/2の区別なし。

---

## 20. MULTIPLAYER_PROBABILITY

Setting:
`かくりつをみる`

OFF:
- probability area completely hidden。
- hintなし。

Before first card:
- ONでも何も表示しない。

Normal:
- actual remaining deckから計算。
- last 1 cardでも表示。
- 0 cardsで消す。

Endless:
- full pool + current rankから計算。
- historyに依存しない。

Display concept:
`↓ 35%   おなじ 8%   ↑ 57%`
+ Joker ONならseparate Joker %

Rounding:
- single-playerと統一。
- integer %。
- 表示総和100%になるよう調整。

History:
- probability表示なし。

---

## 21. MULTIPLAYER_HISTORY

Setting:
`でたカードをみる`

OFF:
- history button完全非表示。

ON:
- first card前でも開ける。
- full-screen。
- game interaction停止。
- top-left `もどる`
- home shortcutなし。
- title `でたカード`

Base matrix:
- columns: ♦ ♥ ♠ ♣
- rows: A...K
- suit headers symbol only。
- red/gray suit color rulesは共通カード仕様準拠。

Normal:
- 52セル最初から通常表示。
- drawn = semi-transparent。
- current = normal opacity + thick border。
- checkmarkなし。
- `いま`なし。
- remaining `あと ○まい`
- deck0時:
  - remaining count表示なし。
  - 全drawn状態可視。
  - 戻るとdeck0 stateへ。
  - 自動resetなし。

Endless:
- cellにoccurrence `×N`
- unseen = faint
- current = normal + thick border + count
- frequencyによる色強調なし
- per-card count cap display: `×99+`
- total draw count表示なし
- `∞`表示

Joker:
- matrix下に前節仕様で表示。

No direct shuffle button in history.

---

## 22. MULTIPLAYER_TOP_NAV

Left→right order:
1. home icon + `ホーム`
2. card/history icon + `でたカード` (history ON時のみ)
3. shuffle icon + `山札をまぜる` (normal only)
4. gear icon + `せってい`

Rules:
- always icon + text。
- narrow deviceでも文字を残す。
- unavailable button = disabledではなくhidden。
- hidden後は残りが自然に詰める。
- home左、settings右。
- mode titleなし。
- light press feedback。
- sound/haptic settingsに従う。

---

## 23. SETTINGS_GLOBAL

Title:
`せってい`

Top:
- titleのみ。説明文なし。
- top-left `← もどる`
- opened-from画面へ戻る。
- home shortcutなし。
- autosave。
- save buttonなし。
- `保存しました` feedbackなし。

Background:
- 明るいアイボリー無地。

Layout:
- category cards。
- かなり薄い色。
- rounded。
- very subtle shadow。
- 幅は画面ほぼいっぱい + side margins。
- 内部余白ゆったり。
- 各setting rowをさらに小カードで囲わない。
- 項目間はdivider lineなし、spacingのみ。

Category header:
- bold, slightly larger。
- colored icon。
- card/backgroundでカテゴリまとまりを明示。

Category structure:
- ゲーム
  - `ひとりであそぶ`
  - `みんなであそぶ`
- `おと・しんどう`
- `そのほか`

Single/Multiplayer game categories:
- accordion。
- 初期は両方closed。
- 同時に1つだけopen。
- 前回open状態は保存しない。

Setting item:
- name。
- 常時1行補足説明。
- 補足は名前より一段小さい。
- slightly lighter text。
- toggle = `つける | けす` two-choice。
- 選択側背景色あり。
- 同一幅。
- 順序固定 `つける | けす`

Single game settings:
- `かくりつをみる`
- `でたカードをみる`
- `さいこうきろくをけす`
  - single section内。
  - soft red text。
  - trash icon。
  - destructive visual。
  - confirmation:
    `さいこうきろくを けしてもいい？`
    buttons: `けす / やめる`
  - reset後:
    `きろくをけしたよ`
  - 2段階confirmなし。

Sound:
- `おと`
Vibration:
- `しんどう`

Other:
- `遊び方`
- `このアプリについて`

---

## 24. ABOUT_SCREEN

Entry:
Settings → `このアプリについて`

Back:
- `← もどる` → 必ずSettingsへ。

No banner ad on this screen.

Order:
1. `このアプリをシェア`
2. `レビューする`
3. `お問い合わせ`
4. `プライバシーポリシー`
5. `利用規約`
6. copyright
7. version

Version:
`バージョン 1.0.0`
- build number非表示。

Copyright:
`© 2026 <DEVELOPER_NAME>`

Review:
- App Store review screenへ直接。
- 自動レビュー依頼popupは一切出さない。

Privacy / Terms:
- app内Web view。
- top-left `← もどる`。
- 戻り先 = About。
- そのページ内の外部リンクはSafari。
- 通信失敗:
  `つうしんできなかったよ。もういちどためしてね`

Contact:
- app内標準メールcomposer。
- 宛先自動入力。
- 件名にapp name。
- 本文上部は自由記入。
- 下部に自動付与:
  - app version
  - iOS version
  - iPhone model
- 問い合わせカテゴリ事前選択なし。
- mail composer不可時:
  `メールを おくれるように せっていしてね`

Placeholders:
```yaml
developer_name: "<TBD>"
support_email: "<TBD>"
privacy_url: "<TBD>"
terms_url: "<TBD>"
app_store_url: "<TBD>"
```

---

## 25. APP_SHARE

About → `このアプリをシェア`

Share payload:
- fixed intro image
- editable default intro text
- App Store link

Default text style:
- やさしい日本語。
- 例:
  `つぎのカードは うえ？ した？ ひとりでも みんなでもあそべるハイローゲームだよ！`

Intro image:
- externally supplied fixed image file。
- app内動的生成しない。
- size 1080x1080。

Design:
- homeと同系色のbright pop background。
- faint fixed ♠♥♦♣ / card silhouettes。
- exact placementはsquare専用。
- app name/logo top center。
- logoはhomeと同デザイン。
- app icon large, center-upper。
- iconとlogoは近め。
- app icon = App Store風rounded icon, no extra stroke, very subtle shadow。
- catchphrase 2 lines:
  1. `つぎのカードは`
  2. `↓ した？        ↑ うえ？`
- line2はline1より少し大きい。
- ↓ left, ↑ right。
- ↓ blue。
- ↑ pink-orange。
- arrowsは文字より少し大きく太い。
- button-like backgroundなし。
- same rounded bold app font。
- supporting line:
  `ひとりでも みんなでも！`
  - theme color
  - normal weight
  - understated
- QR lower-right。
- QR+guideまとめてwhite rounded plate。
- plate subtle shadow。
- plate corners moderately rounded。
- guide above QR:
  `アプリストアであそべるよ`
  - dark navy。
- QR points App Store。
- screenshotは使わない。

---

## 26. APP_ICON

Externally generated asset; implementation AIは生成しない。

Spec:
- no text。
- background: pink→purple→blue gradient。
- central: 1 playing card, slightly tilted。
- card = 7♥ fixed。
- left large ↓。
- right large ↑。
- ↓ blue family。
- ↑ pink-orange family。
- arrows textなし。
- bright/pop/game-like。
- 子ども向けすぎず学生にも自然。
- small-size recognition priority。

---

## 27. LAUNCH_SCREEN

- background: light ivory solid。
- center:
  - main `ハイローラッシュ`
  - small `HighLowRush`
- logo only。カード/矢印なし。
- extra animationなし。
- homeに入った後の追加intro animationなし。
- 起動後すぐ操作可能。

---

## 28. ADS / ADMOB

SDK:
Google AdMob

IDs:
```yaml
admob:
  app_id_ios: "ca-app-pub-8174756915786797~3111467770"
  banner_unit_name: "highlowrush_ios_home_banner"
  banner_unit_id: "ca-app-pub-8174756915786797/4454826263"
```

Use:
- 同一banner unitを全banner対象画面で共通利用。
- Adaptive Banner。
- banner only。
- interstitialなし。
- rewardedなし。
- personalized tracking adsなし。
- IDFA trackingなし。
- ATT promptなし。
- SDK/地域要件上必要なprivacy consentのみ別途適切に実装。

Placement:
- all major eligible screens bottom fixed。
- operationsから十分離す。
- bottom safe areaを確保。
- layout内に広告スペースを事前予約。
- load中/failed時もreserved areaを維持。
- loading indicator/messageなし。
- load failure時は自動retry。
- retry timingは実装AIに委任。
- layout jump禁止。
- accidental tap誘発禁止。

Banner ON screens:
- Home
- Single game
- Single result
- Multiplayer setup
- Multiplayer game
- Settings
- 遊び方
- その他「主要画面」で本仕様と矛盾しないもの

Banner OFF:
- `このアプリについて`
- Privacy/Terms WebView
- system mail composer等

Development:
- 開発/テスト中はtest adsを使用し、本番広告の不正表示/クリックを避ける。

---

## 29. PRIVACY / DATA

```yaml
tracking: false
ATT_prompt: false
usage_analytics: false
gameplay_telemetry: false
personalized_ads: false
crash_reports:
  enabled: true
  anonymous: true
```

Crash report may include:
- app version
- iOS version
- iPhone model
- crash/diagnostic data

Do NOT send:
- play count
- streak
- selected mode
- gameplay history
- user behavior analytics

Crash reporting:
- 個別consent popupなし。
- privacy policyに明記。

Privacy/Terms:
- 初回起動時に同意popupなし。
- Aboutからいつでも確認。

---

## 30. NETWORK / ERROR

Game:
- 基本offline完全動作。
- 起動時offline警告なし。

Network-required functions failure:
- common message:
  `つうしんできなかったよ。もういちどためしてね`

Unexpected recoverable error:
- crashを避ける。
- game stateだけ安全終了しhomeへ戻す等のfallback。
- homeへ戻した場合:
  `うまくよみこめなかったので ホームにもどったよ`

App crash/force quit:
- next launch = Home。
- active game復元なし。
- persistent settings/BEST等のみ維持。

Temporary background:
- current screen/session維持。
- multiplayerでは復帰後もgame中のみkeep-awake再適用。

---

## 31. NAVIGATION

Global:
- left-edge swipe back disabled。
- explicit button navigation only。
- standard short fade/slide transitions。
- custom card-flip screen transitionsなし。

Home → Single:
- first ever only tutorial。
- subsequent direct game start using saved settings。

Home → Multiplayer:
- always preset screen first。

Single game → Home:
- confirm, manual exit, BEST no update。

Single history:
- `もどる` only。

Multiplayer history:
- `もどる` only。

Settings:
- `← もどる` only。
- return to opener。

About:
- back → Settings。

Privacy/Terms:
- back → About。

No screen should add an app-quit action。

---

## 32. BEST / PERSISTENCE

Persistent data:
- single BEST exact integer。
- sound。
- vibration。
- single probability。
- single history。
- multiplayer endless。
- multiplayer joker。
- multiplayer history。
- multiplayer probability。
- single tutorial completed flag。
- other user-facing settings needed by spec。

Initial:
- BEST = 0
- sound = ON
- vibration = ON
- single probability = OFF initially; later persist chosen state.
- single history = OFF initially; later persist chosen state.
- multiplayer all four = OFF.

Important:
- app reinstall = settings/BEST reset。
- app update = keep。
- manually exiting single game = no BEST update。
- app kill mid-single = no BEST update。
- game over after actual wrong guess = evaluate/update BEST。
- reset BEST action explicit only。

---

## 33. UI_COPY_CANONICAL

Use exactly unless localization/layout needs punctuation spacing:

```text
ハイローラッシュ
HighLowRush
つぎのカードは うえ？ した？

ひとりであそぶ
みんなであそぶ

せってい
おと
しんどう
つける
けす

かくりつをみる
でたカードをみる
でたカード

○れんしょう！
さいこうきろく ○れんしょう
○しゅうめ
あと ○まい

せいかい！
おなじ！
ざんねん！
きろくこうしん！

もういちど
シェア
ホーム
もどる

あそびかた
あそぶ！

あそびかたをきめる
これであそぶ
エンドレス
ジョーカー
山札をまぜる

山札を まぜなおしたよ！

さいこうきろくを けしてもいい？
けす
やめる
きろくをけしたよ

ホームにもどると、いまのゲームは おわるよ
ホームにもどる
ホームにもどると、いまの山札はなくなるよ
ホームにもどると、いままでのカードはきえるよ

山札をまぜなおす？
まぜなおす

せっていをかえると、いまの山札はさいしょからになるよ
せっていをかえると、いままでのカードはきえるよ
せっていをかえる

このアプリをシェア
レビューする
お問い合わせ
プライバシーポリシー
利用規約

アプリストアであそべるよ

つうしんできなかったよ。もういちどためしてね
メールを おくれるように せっていしてね
うまくよみこめなかったので ホームにもどったよ
```

---

## 34. APP_STORE_META

Confirmed:
```yaml
app_name: "ハイローラッシュ"
english_name: "HighLowRush"
subtitle: "めくって当てる！かんたんハイローゲーム"
```

Do NOT generate here:
- promotional text
- full App Store description
- screenshots copy
Those are delegated to another AI later.

---

## 35. PROTOTYPE_AND_ASSET_WORKFLOW

Priority order:
1. まずゲームロジック/画面遷移/設定/保存/広告/各種挙動が仕様通り動くプロトタイプを完成。
2. 初期プロトタイプではUIを過度に作り込まない。
3. 本番用の凝った画像アセットはプロトタイプ動作確認後に別途作成・提供。
4. 提供後、本番アセットへ差し替えてUIを仕上げる。

Prototype phase:
- 未提供の画像箇所は標準UI、単純な図形、仮画像、プレースホルダーで代用可。
- 本番画像をAI側で無理に生成する必要はない。
- 仮表示でもレイアウト/画面遷移/タップ領域/状態変化は確認可能にする。
- 後から画像を容易に差し替えられる構造にする。
- 仮アセットに本番デザイン上の意味を持たせすぎない。

Final external assets expected later:
1. App icon
2. Launch logo
3. Fixed app-intro/share image (1080x1080)
4. Decorative background assets if needed
5. App/logo symbols used on card back if provided
6. その他、プロトタイプ確認後にChatGPT側で作成するデザインアセット

Can be code-drawn both prototype/final where appropriate:
- simple arrows
- standard card outlines
- basic suit symbols
- simple shadows/rounded rectangles
- simple background fills/patterns

---

## 36. SCREEN_LIST

```text
LaunchScreen
Home
SingleTutorial(first single only)
SingleGame
SingleHistory
SingleResult
MultiplayerSetup
MultiplayerGame
MultiplayerHistory
Settings
HowToPlay(tabbed single/multi)
About
PrivacyWebView
TermsWebView
SystemMailComposer
SystemShareSheet
AppStoreReviewRoute
```

---

## 37. ACCEPTANCE_CRITERIA

Implementation is acceptable only if all true:

1. iPhone only / iOS17+ / portrait only.
2. iPad target removed/not offered.
3. Game UI never requires scrolling.
4. Rank rule exactly A<2<...<K.
5. Single uses physical 52-card depletion + cyclic reshuffle rules exactly.
6. Single equal rank never increments streak and never ends game.
7. Single wrong guess ends game.
8. Manual single exit never updates BEST.
9. Single probability uses actual remaining deck.
10. Single history matrix/current-card semantics match spec.
11. Multiplayer never judges win/loss.
12. Multiplayer first card requires deck tap.
13. Normal multiplayer supports optional 2 Jokers.
14. Endless multiplayer is independent full-pool draw.
15. Multiplayer probability/history/Joker behavior match spec.
16. All saved settings persist through normal update.
17. Reinstall resets local data.
18. Offline gameplay remains fully usable.
19. No IAP/interstitial/rewarded ads.
20. AdMob banner adaptive, bottom fixed, reserved space, no layout jump.
21. No tracking/ATT/usage analytics.
22. Anonymous crash diagnostics only.
23. About screen has no banner.
24. Swipe-back disabled.
25. App kill/crash resumes at Home, not mid-game.
26. All user-visible Japanese copy is gentle and primarily hiragana-centric.
27. Prototype phase may use simple placeholders/standard UI for unprovided complex images; final phase replaces them with externally supplied assets.
28. UI remains stable on small and large supported iPhones.
29. Major controls respect Safe Area and remain clearly separated from banner ad.
30. No feature not defined here should be added without explicit approval.

---

## 38. PLACEHOLDERS_TO_FILL_BEFORE_RELEASE

```yaml
developer_name: TBD
support_email: TBD
privacy_url: TBD
terms_url: TBD
app_store_url: TBD
final_asset_filenames: TBD
crash_reporting_service: implementation_choice
```

---

## 39. DO_NOT_INFER

Implementation AI must NOT invent:
- accounts/login
- cloud sync
- online ranking
- multiplayer networking
- real-time rooms
- statistics beyond BEST
- difficulty levels
- BGM
- achievements
- push notifications
- IAP
- subscriptions
- interstitial/rewarded ads
- user tracking
- analytics
- extra tutorial popups
- extra confirmation dialogs
- custom back-swipe navigation
- extra game modes
- card strength by suit
- Joker strength
- multiplayer answer buttons
- auto first draw in multiplayer
- auto reshuffle in multiplayer normal mode
- auto-reset history on opening history
- share image fields not explicitly specified
