# AzMythic 戦闘Telemetry規約

## 目的

- 手動検証した戦闘結果を、次の調整に戻せる形で残す。
- `MythicMobs` の実機挙動と、プレイヤーが感じたプレイフィールを分けて記録する。
- 後から `Codex` や補助スクリプトが読みやすい構造を固定する。

## 基本方針

- 1 戦闘につき 1 JSON を作る。
- JSON は `telemetry/schema/encounter-report.schema.json` に従う。
- プラグインの聞き取り結果だけを切り出す場合は `telemetry/schema/encounter-survey.schema.json` に従う。
- 実測値と主観評価は混ぜず、`server_metrics` と `player_observation` を分ける。
- 動画、ログ、スクリーンショットは JSON に埋め込まず、`artifacts` にパスだけを書く。
- 取れない値は `null` または空配列でよい。推測値を実測値として書かない。

## 配置

- `telemetry/schema/encounter-report.schema.json`
  正式な JSON Schema
- `telemetry/schema/encounter-survey.schema.json`
  プラグインのアンケート出力用 Schema
- `telemetry/templates/encounter-report.template.json`
  新規検証のひな形
- `telemetry/templates/encounter-survey.template.json`
  アンケート出力のひな形
- `telemetry/examples/*.json`
  記入例
- `telemetry/runs/*.json`
  実戦検証ログ

`telemetry/runs` は検証のたびに増えるので、基本は Git 管理しない。

## 1 ファイル 1 戦闘

1 つの JSON は、1 回の戦闘セッションだけを表す。

例:

- ウィデラーを近接ビルドで 1 回討伐
- フェルドスを遠距離ビルドで 1 回途中撤退
- 同じボスでも、装備や立ち回りが違うなら別ファイル

## 必須セクション

### `encounter`

戦闘の対象と条件を表す。

- `encounter_id`
  一意な記録 id
- `boss_id`
  対象ボスや mob の内部識別子
- `boss_name`
  表示名
- `server_id`
  `azup1` など
- `scenario`
  どんな検証か
- `tester`
  実際に触った人
- `played_at`
  検証日時

### `loadout`

検証時のプレイヤー条件を表す。

- `weapon_id`
- `armor_set`
- `role`
  `melee`, `ranged`, `tank`, `hybrid`
- `notes`

### `server_metrics`

実機挙動として観測できた事実を書く。

- `result`
  `victory`, `wipe`, `retreat`, `abort`
- `combat_time_sec`
- `boss_hp_end_ratio`
  0.0 から 1.0
- `player_deaths`
- `player_damage_taken`
- `player_damage_dealt`
- `peak_dps_window`
  短時間火力のメモ
- `phase_reached`
- `phase_transitions`
- `skill_events`
- `cc_events`
- `death_causes`

### `player_observation`

人間が感じたプレイフィールを書く。

- `telegraph`
  予兆の見やすさ
- `fairness`
  理不尽さの少なさ
- `pressure`
  圧の適正さ
- `readability`
  情報の読みやすさ
- `role_balance`
  近接、遠距離、立て直し余地の偏り
- `highlights`
  良かった点
- `pain_points`
  問題点

### `artifacts`

外部ファイルへの参照を書く。

- `latest_log_path`
- `video_path`
- `replay_path`
- `screenshots`

### `score`

評価器や手動採点結果を書く。

- `telegraph_score`
- `fairness_score`
- `pressure_score`
- `readability_score`
- `role_balance_score`
- `overall_score`
- `summary`

## スコア基準

各スコアは `0` から `100` を使う。

- `90-100`
  かなり良い。修正は微調整でよい
- `70-89`
  実用域。改善余地あり
- `50-69`
  問題が見える。次回修正候補
- `0-49`
  明確な欠陥あり。優先対応

## `skill_events` の使い方

`skill_events` は完全な全記録でなくてよい。調整対象の攻撃だけでも価値がある。

優先して残すもの:

- 初見殺しになりやすい攻撃
- フェーズ移行技
- 被弾理由が分かりにくい攻撃
- 強すぎる、弱すぎると感じた攻撃

## `player_observation` の書き方

主観は主観として、そのまま残してよい。ただし曖昧な言葉だけで終わらせない。

悪い例:

- `理不尽だった`

良い例:

- `Attack4 は予兆開始から着弾までが短く、近接距離では 2 回中 2 回とも視認回避できなかった`

## 推奨運用

1. 戦闘前に `telemetry/templates/encounter-report.template.json` を複製する。
2. 戦闘後すぐに `encounter`, `loadout`, `server_metrics` を埋める。
3. 必要に応じてプラグインの `/encountersurvey start <bossId> [focus...]` で聞き取り JSON を出す。
4. 記憶が新しいうちに `player_observation` と `score` を埋める。
5. その JSON を `Codex` に渡して再調整案を作る。

## アンケート出力の位置付け

`encounter-survey` は、完全な戦闘記録ではなく「主観評価を取りこぼさず残す補助記録」である。

- 実測値が揃っているなら `encounter-report` を正本にする
- プラグインアンケートは `player_observation` と `focus` を強化する補助情報として使う
- 後で統合する時は、`boss_id`, `played_at`, `tester` をキーに寄せる

## Codex に渡す時の最小セット

最低でも次の 3 つがあれば、再調整案をかなり返しやすい。

- `server_metrics.skill_events`
- `player_observation.pain_points`
- `score`

## 将来拡張

将来的に補助プラグインや bot を入れる場合も、この schema を壊さずに項目を増やす。

優先候補:

- tick 単位の `timeline`
- `position_trace`
- `avoidability_window_ms`
- `stacked_hazard_windows`
- `phase_failure_reason`
