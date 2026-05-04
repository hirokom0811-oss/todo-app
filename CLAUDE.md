# CLAUDE.md — TODO & スケジューラー アプリ

# 自動更新ルール
このファイルは実装完了のたびに自動更新すること。
内容が実際のコードと常に一致している状態を保つこと。

# Git 自動コミットルール
実装完了のたびに以下を自動実行すること：
```
git add index.html
git commit -m "feat: [実装内容の日本語要約]"
git push origin main
```
- コミットメッセージは日本語で簡潔に（例：「feat: やること100に目標種類を追加」）
- CLAUDE.md を更新した場合は `git add CLAUDE.md` も含めること

# バージョン管理ルール

## バージョン番号の付け方（セマンティックバージョニング）

| 種別 | 変更内容の例 | バージョン操作 |
|------|-------------|---------------|
| major | 新タブ追加など大機能追加 | X.0.0 |
| minor | 機能追加・既存機能の改善 | x.Y.0 |
| patch | バグ修正 | x.y.Z |

## ⚠️ 実装完了時の必須手順（省略禁止）

実装が完了するたびに、以下を**必ずこの順番で**実行すること：

### ステップ1：APP_VERSION を更新する
`index.html` の `APP_VERSION` 定数を新しいバージョン番号に書き換える。

```js
const APP_VERSION = 'vX.Y.Z'; // ← 変更内容に応じて必ずインクリメント
```

- feat（機能追加）→ minor を +1（例: v1.2.1 → v1.3.0）
- fix（バグ修正）→ patch を +1（例: v1.2.1 → v1.2.2）
- 新タブ追加など大機能 → major を +1（例: v1.2.1 → v2.0.0）

### ステップ2：コミット＆プッシュ
```
git add index.html
git commit -m "feat/fix: [実装内容の日本語要約]"
git push origin main
```

### ステップ3：タグを付けてプッシュ
```
git tag vX.Y.Z
git push origin main --tags
```

- タグ名は `APP_VERSION` と**必ず一致**させること
- ステップ2のコミット後に実行すること

> **重要**: `APP_VERSION` の更新を忘れると過去の変更が積み残しになる。
> 1実装 = 1バージョンアップ を徹底すること。

## index.html への記載

`index.html` の `<script>` ブロック先頭付近に以下を定義すること：
```js
const APP_VERSION = 'v1.5.2';
```

バージョンはヘッダーUIに表示すること（例：アプリ名の右横に小さく表示）。

## タグ付けルール

実装完了時・バージョン更新時に以下を自動実行すること：
```
git tag vX.X.X
git push origin main --tags
```
- タグ名は `APP_VERSION` と必ず一致させること
- コミット → プッシュ → タグ付け → タグプッシュの順で実行すること

## 概要

仕事・個人両用のTODO管理 + カレンダースケジューラーを連携させたシングルファイルWebアプリ。
外部JSライブラリなし、LocalStorage永続化、PWA対応。


| 項目 | 内容 |
|------|------|
| ファイル | `E:\hiro\Claude\TODO\index.html`（約4800行、1ファイル完結） |
| 技術スタック | バニラ HTML5 / CSS3 / ES6+（外部JSライブラリなし） |
| 動作環境 | ブラウザで直接開く（`file://`）または簡易HTTPサーバー経由 |
| 永続化 | `localStorage` のみ |
| フォント | Google Fonts: Noto Sans JP (CDN) |
| PWA | `apple-mobile-web-app-capable` など iOS PWAメタタグ済み |

---



## ページ構成（4タブ）

| タブ | ID | 内容 |
|------|----|------|
| TODO | `page-todo` | タスク管理 + カレンダースケジューラー（左右分割） |
| やること | `page-goals` | 目標管理（グリッド or グループ表示、件数上限なし） |
| プロジェクト | `page-projects` | プロジェクト管理（カード一覧・フィルター・AI分解） |
| 旅モード | `page-travel` | 近くのスポット検索 + カレンダー登録 |

---

## LocalStorage キー一覧

| キー | 型 | 内容 |
|------|----|------|
| `todo_tasks` | `Task[]` | タスク一覧 |
| `todo_goals` | `Goal[]` | 目標一覧 |
| `todo_blocks` | `Block[]` | 作業枠一覧 |
| `todo_completions` | `{taskId,date}[]` | 繰り返しタスクの日別完了ログ |
| `todo_themes` | `{work:string[],personal:string[]}` | テーマ一覧 |
| `todo_block_templates` | `BlockTemplate[]` | 作業枠テンプレート |
| `todo_projects` | `Project[]` | プロジェクト一覧 |
| `todo_reviews` | `Review[]` | 振り返り一覧 |
| `todo_gist_token` | `string` | GitHub Personal Access Token |
| `todo_gist_id` | `string` | GitHub Gist ID（初回保存時に自動生成） |
| `todo_last_saved` | `string` | Gist最終保存日時（ISO8601） |
| `todo_gemini_api_key` | `string` | Gemini API キー |
| `todo_gemini_model` | `string` | Gemini モデル名（デフォルト: `gemini-2.0-flash`） |

---

## データモデル

### Task

```json
{
  "id": "uuid",
  "title": "タスク名",
  "dueDate": "2026-04-25 | null",
  "scheduledDate": "2026-04-25 | null",
  "scheduledTime": null,
  "scheduledEndTime": null,
  "blockIds": ["uuid"],
  "goalId": "uuid | null",
  "parentId": "uuid | null",
  "projectId": "uuid | null",
  "category": "work | personal",
  "priority": "high | medium | low",
  "scale": "large | medium | small",
  "theme": "設備計画 | 任意文字列 | null",
  "memo": "メモ | null",
  "completed": false,
  "recurrence": "daily | weekday | weekly | null",
  "recurrenceEnd": "2026-12-31 | null",
  "createdAt": "ISO8601"
}
```

**設計メモ:**
- `scheduledTime` / `scheduledEndTime` は廃止済み（フィールドは残存するが未使用）。時刻管理は Block で行う。
- `parentId` — AI分解で生成したサブタスクが親タスクのIDを持つ。`null` の場合は親タスク（TODOリストに表示）。
- `blockIds[]` — Task 側が Block への参照を持つ（Block 側は Task 参照を持たない）。
- `projectId` — 仕事カテゴリのタスクが紐づくプロジェクトID。`null` の場合は未紐づけ。既存データは `null` として後方互換。
- `memo` — タスクのメモ欄。AI分解のコメントも引き継ぐ。

### Block（作業枠）

```json
{
  "id": "uuid",
  "title": "作業枠名",
  "theme": "テーマ名",
  "date": "2026-04-25",
  "startTime": "10:00",
  "endTime": "11:30",
  "recurrence": "daily | weekday | weekly | null",
  "recurrenceEnd": "2026-12-31 | null",
  "createdAt": "ISO8601"
}
```

**設計メモ:**
- 繰り返しは1件の Block オブジェクトを `isBlockOnDate()` で展開して表示（インスタンスを生成しない）。
- タスクとの紐づけは Task 側の `blockIds[]` で管理する。

### Goal（目標）

```json
{
  "id": "uuid",
  "number": 1,
  "title": "目標名",
  "category": "仕事 | 個人 | その他",
  "theme": "テーマ名",
  "type": "achievement | habit | explore | null",
  "progressNote": "現在の状況メモ | null",
  "status": "notStarted | inProgress | done",
  "memo": "メモ",
  "doneAt": "ISO8601 | null",
  "createdAt": "ISO8601"
}
```

**設計メモ:**
- 件数上限なし（旧来の100件制限は撤廃）。
- `number` は一意な正整数。グリッド表示の位置を決める。
- グリッドは `Math.ceil(Math.max(maxNum+1, 10) / 10) * 10` セルに自動拡張。
- `type` — 目標の種類。AI分解プロンプトの切り替えに使用。既存データは `null` として後方互換。
  - `achievement`（達成型）: 明確なゴールがあり、達成したら完了するもの
  - `habit`（習慣型）: 繰り返しで積み上げていくもの。AI分解で12ヶ月分の繰り返しタスクを提案
  - `explore`（探索型）: やりながら進め方を決めていくもの。`progressNote` をAIへのヒントとして使用
- `progressNote` — 探索型で主に使用するが、モデルとしては全typeに持たせる。

### BlockTemplate（作業枠テンプレート）

```json
{
  "id": "uuid",
  "title": "朝のルーティン",
  "startTime": "07:00",
  "endTime": "08:00",
  "theme": "人づくり"
}
```

### Project（プロジェクト）

```json
{
  "id": "uuid",
  "title": "プロジェクト名",
  "theme": "設備計画 | 任意文字列 | null",
  "status": "active | waiting | done",
  "priority": "high | medium | low",
  "dueDate": "2026-05-15 | null",
  "memo": "",
  "createdAt": "ISO8601",
  "doneAt": "ISO8601 | null"
}
```

**設計メモ:**
- Projectは仕事専用。category は常に "work" として扱う（フィールドは持たない）。

### Review（振り返り）

```json
{
  "id": "uuid",
  "date": "2026-05-04",
  "items": [
    {
      "taskId": "uuid",
      "title": "タスク名",
      "parentTitle": "親タスク名 | null",
      "completed": true,
      "result": "good | bad | null",
      "comment": "選択コメント | null",
      "commentFree": "自由入力コメント | null"
    }
  ],
  "memo": "一言メモ",
  "aiAdvice": "AIアドバイス文 | null",
  "createdAt": "ISO8601"
}
```

**設計メモ:**
- 日付ごとに1件。同じ日に上書き保存した場合は既存レコードを更新する。
- `items` は振り返りダイアログを開いた時点の完了/未完了タスクのスナップショット。
- `aiAdvice` は Gemini API から取得したアドバイス文。保存時に記録される。

---

## グローバル状態（`S` オブジェクト）

```js
const S = {
  page: 'todo',           // 現在のページ
  tasks: [],              // Task[]
  goals: [],              // Goal[]
  blocks: [],             // Block[]
  blockTemplates: [],     // BlockTemplate[]
  completions: [],        // {taskId, date}[]
  themes: { work: ['設備計画','導入','活用','人づくり','コミュ・雑務'], personal: [] },
  projects: [],           // Project[]
  reviews:  [],           // Review[]
  filter: { status: 'active', category: 'all', date: 'today', goalId: null }, // null = Goal フィルター無効
  projectFilter: 'all',   // 'all' | 'active' | 'waiting' | 'done'
  projectSort: 'priority',// 'priority' | 'due'
  calView: 'day',         // 'day' | 'week' | 'month'
  calDate: new Date(),
  highlightId: null,      // ハイライト表示するタスクID
  goalView: 'grid',       // 'grid' | 'list'
};
```

グローバル変数（モーダル管理・旅モード・振り返り・未保存フラグ）:

```js
let _blockEditId        = null;  // 編集中Block ID（null = 新規）
let _blockPendingTaskId = null;  // Block作成後にリンクするTask ID
let _blockViewDate      = null;  // カレンダー上で「何日として開いたか」（繰り返しタスクの完了ログ日付に使用）
let _schedTaskId        = null;  // スケジュールモーダルの対象Task ID
let _projectEditId      = null;  // 編集中Project ID（null = 新規）
let _travelLat          = null;  // 旅モード: 検索緯度
let _travelLng          = null;  // 旅モード: 検索経度
let _travelSpots        = [];    // 旅モード: 最後の検索結果
let _reviewDate         = null;  // 振り返りモーダルの対象日付
let _reviewItems        = [];    // 振り返りモーダルのアイテム配列
let _reviewAiAdvice     = null;  // 振り返りモーダルのAIアドバイス文字列
let _localDirty         = false; // データ変更後Gist未保存フラグ
```

---

## 主要関数リスト

### ストレージ

| 関数 | 説明 |
|------|------|
| `loadAll()` | LocalStorage から全データを `S` に読み込む。旧 `blockId` → `blockIds[]` 移行処理も行う |
| `saveTasks()` / `saveGoals()` / `saveBlocks()` / `saveThemes()` / `saveCompletions()` / `saveBlockTemplates()` / `saveProjects()` / `saveReviews()` | 各エンティティを localStorage に書き込み、`markDirty()` を呼ぶ（アロー関数定数） |
| `markDirty()` | `_localDirty = true` にセット。Gistトークン・ID設定済みの場合のみ未保存バナーを表示 |

### Task CRUD

| 関数 | 説明 |
|------|------|
| `addTask(d)` | `S.tasks.unshift()` で先頭追加。`parentId: d.parentId \|\| null` を含む |
| `updateTask(id, data)` | スプレッド構文でフィールドを上書き |
| `deleteTask(id)` | ID でフィルタリングして削除 |
| `toggleComplete(id)` | `t.completed` を反転。繰り返し・通常タスク共通 |

### 繰り返しタスク

| 関数 | 説明 |
|------|------|
| `isOccurrenceOnDate(t, ds)` | 繰り返しタスクが指定日に発生するか判定（`scheduledDate` 必須） |
| `isRecurringDone(taskId, ds)` | `completions` ログに指定日の完了記録があるか確認 |
| `toggleCompletion(taskId, ds)` | `completions` ログの指定日をトグル |
| `getOccurrenceDates(task)` | 繰り返し発生日一覧を返す（上限 366 件） |
| `isRecurringFullyDone(task)` | 全発生日が完了済みか |
| `isTaskDone(t, ds?)` | 完了判定の統合関数。`t.completed`（強制完了）→ 日別ログ → 全日完了の順で判定 |
| `recurLabel(t)` | 繰り返し種別を日本語で返す |

### Block CRUD

| 関数 | 説明 |
|------|------|
| `addBlock(d)` | Block を追加して ID を返す |
| `updateBlock(id, data)` | スプレッド構文で上書き |
| `deleteBlock(id)` | Block 削除 + 紐づくタスクの `blockIds` からも除去 |
| `linkTaskToBlock(taskId, blockId)` | Task の `blockIds` に追加 |
| `unlinkTaskFromBlock(taskId, blockId)` | Task の `blockIds` から除去 |
| `isBlockOnDate(b, ds)` | Block が指定日に発生するか判定（繰り返し対応） |
| `getBlockById(id)` | ID で Block を取得 |
| `getBlockTasks(blockId)` | Block に紐づく Task 一覧を取得 |

### BlockTemplate

| 関数 | 説明 |
|------|------|
| `addBlockTemplate(d)` | テンプレートを追加 |
| `deleteBlockTemplate(id)` | テンプレートを削除 |
| `applyBlockTemplate(templateId)` | Block モーダルのフォームにテンプレート値を反映 |

### Goal CRUD

| 関数 | 説明 |
|------|------|
| `addGoal(d)` | Goal を追加 |
| `updateGoal(id, data)` | `status === 'done'` 時に `doneAt` を自動セット |
| `deleteGoal(id)` | Goal 削除 + 紐づくタスクの `goalId` を null に |
| `nextGoalNum()` | 使用されていない最小の正整数を返す（上限なし） |

### Project CRUD

| 関数 | 説明 |
|------|------|
| `addProject(d)` | Project を `S.projects` 先頭に追加。ID を返す |
| `updateProject(id, data)` | `status === 'done'` 時に `doneAt` を自動セット |
| `deleteProject(id)` | Project を削除 |
| `getProjectById(id)` | ID で Project を取得 |
| `getProjectTasks(projectId)` | プロジェクトに紐づく親タスク一覧 |
| `openProjectModal(id?)` | 追加/編集モーダルを開く。`id=null` で新規 |
| `saveProject()` | モーダルの入力値を保存 |
| `confirmDeleteProject(id)` | 確認ダイアログ → `deleteProject()` |
| `renderProjectList()` | プロジェクトカード一覧を描画 |
| `setProjectFilter(f)` | フィルターを切替えて再描画 |
| `setProjectSort(s)` | ソートを切替えて再描画 |
| `decomposeProject(projectId)` | AI でプロジェクトをタスクに分解（Gemini API） |
| `addSelectedAiTasksForProject(tasks, project)` | AI提案タスクを選択してTODOに追加 |
| `suggestProjects()` | 今週注力すべきプロジェクトをAIが提案 |

### 目標フィルター

| 関数 | 説明 |
|------|------|
| `buildGoalFilterButtons()` | タスクから参照されているGoalのみ抽出し、ボタン定義配列を返す |
| `renderGoalFilterButtons()` | `#filter-goals` に目標フィルターボタンを描画。Goalが1件以下なら非表示 |
| `setGoalFilter(goalId)` | `S.filter.goalId` をセットして再描画。`null` でフィルター解除 |

### テーマ管理

| 関数 | 説明 |
|------|------|
| `addTheme(cat, name)` | テーマを追加（重複チェックあり） |
| `removeTheme(cat, name)` | テーマを削除 |

### カレンダー描画

| 関数 | 説明 |
|------|------|
| `renderCalendar()` | `S.calView` に応じて月/週/日ビューを振り分け |
| `renderMonth(con)` | 6週×7日グリッド描画 |
| `renderWeek(con)` | 7列×時間軸グリッド描画 |
| `renderDay(con)` | 1日の時間軸 + 作業枠ブロック描画 |
| `calPrev()` / `calNext()` | カレンダーを前後へ移動 |
| `calToday()` | 今日へ移動 |
| `setCalView(v)` | ビューを切替 |
| `jumpToDay(ds)` | 指定日の日ビューへジャンプ |
| `tasksOnDate(ds)` | 指定日に関係する Task 一覧（期日 or スケジュール日） |
| `scheduledOnDate(ds)` | 指定日に予定されている Task 一覧（繰り返し含む） |

### モーダル

| 関数 | 説明 |
|------|------|
| `openTaskDetail(id)` | タスク詳細モーダルを開く |
| `openEditModal(id)` | タスク編集モーダルを開く。`renderEditSubtasks(id)` も呼ぶ |
| `saveEditTask()` | タスク編集を保存 |
| `renderEditSubtasks(parentId)` | 編集モーダル内のサブタスクセクションを描画 |
| `openBlockModal(id?, date?, start?, end?)` | 作業枠追加/編集モーダルを開く |
| `saveBlock()` | 作業枠を保存 |
| `openScheduleModal(id)` | スケジュール登録モーダルを開く（日付のみ設定） |
| `saveSchedule()` | `scheduledDate` を更新 |
| `clearSchedule()` | `scheduledDate` を null に |
| `openGoalDetail(id)` | 目標詳細モーダルを開く |
| `openAddGoalModal(number?)` | 目標追加モーダルを開く |
| `closeAllModals()` | 全モーダルの `.open` クラスを除去 |

### 振り返り機能

| 関数 | 説明 |
|------|------|
| `openReviewModal(ds)` | 指定日の振り返りモーダルを開く。完了/未完了タスクを収集し `_reviewItems` に格納 |
| `renderReviewModal()` | 振り返りモーダルのタスク一覧・AIアドバイスを描画 |
| `setReviewResult(idx, result)` | `_reviewItems[idx].result` を `good` / `bad` でトグル |
| `setReviewComment(idx, val)` | `_reviewItems[idx].comment` を更新 |
| `setReviewCommentFree(idx, val)` | `_reviewItems[idx].commentFree` を更新 |
| `buildReviewPrompt(date, items, memo)` | Gemini へ送るプロンプトを生成。Goal名・進捗・一言メモを含む |
| `getReviewAiAdvice()` | Gemini API を呼び出してAIアドバイスを取得・表示 |
| `saveReviewFromModal()` | `_reviewItems` と `_reviewAiAdvice` を `S.reviews` に保存 |
| `updateReviewBtn(ds)` | 日ビューの振り返りボタンの表示/非表示を更新 |

### AI 機能（Gemini API）

| 関数 | 説明 |
|------|------|
| `decomposeTask()` | タスク編集モーダルのフォーム値を使ってサブタスクを提案 |
| `decomposeGoal(goalId)` | 目標からタスクを提案。内部で `buildDecomposePrompt()` を呼ぶ |
| `buildDecomposePrompt(goal)` | `goal.type` に応じてAI分解プロンプトを切り替えて返す。`habit` 型は `month`（1〜12整数）を返させる |
| `suggestGoals()` | 「今月動けそうなGoal」をGemini APIに問い合わせ、結果モーダルを表示 |
| `buildSuggestPrompt(goals)` | 未着手/進行中ゴールを2セクションに分けてGeminiへのプロンプトを構築。`done` は除外 |
| `renderAiTaskSuggestions(tasks, goalId)` | `modal-ai-tasks` にチェックボックス一覧を描画 |
| `addSelectedAiTasks(tasks, goalId)` | 選択タスクをGoalのカテゴリ/テーマ・recurrence付きで追加。`task.month` がある場合はコード側で `scheduledDate`（月初）と `recurrenceEnd`（月末・うるう年対応）を計算 |
| `addSelectedAiTasksFromEdit(tasks, category, theme)` | 選択タスクを編集中タスクの `parentId` 付きで追加 |
| `toggleAiSelectAll(checked)` | AIタスク提案モーダルの全選択/解除 |
| `onGoalTypeChange(prefix)` | typeラジオ変更時に「状況メモ」欄の表示/非表示を切り替え |

### データ移行・修復

| 関数 | 説明 |
|------|------|
| `migrateHabitTaskDates(tasks, goals)` | 習慣型Goal紐づきタスクのタイトル `（N月）` パターンから現在年の `scheduledDate`（月初）・`recurrenceEnd`（月末）を再計算。修正件数を返す |
| `fixHabitTaskDatesInPlace()` | `migrateHabitTaskDates()` を現在の `S.tasks` / `S.goals` に対して実行し、`saveTasks()` → `renderAll()` |

### 旅モード

| 関数 | 説明 |
|------|------|
| `getGeoLocation()` | GPS で現在地を取得し `_travelLat/_travelLng` にセット |
| `searchByPlace()` | 地名→Nominatim でジオコーディング |
| `searchNearbySpots()` | Wikipedia Geosearch + Nominatim POI でスポット検索 |
| `fetchGourmetSpots(lat, lng, rangeM)` | Nominatim でグルメPOIを取得 |
| `renderSpotResult(spots, summary)` | スポットカード一覧を描画 |
| `showSpotDetail(idx)` | スポット詳細モーダルを開く |
| `toggleRegisterForm(idx)` | スポットカードのインライン登録フォームを開閉 |
| `registerSpotToBlock(idx)` | スポットを作業枠としてカレンダーに追加 |

### データ入出力

| 関数 | 説明 |
|------|------|
| `exportJSON()` | 全データを JSON でダウンロード（iOS Safari はテキスト表示フォールバック付き） |
| `exportGoals()` | 目標を UTF-8 BOM付き CSV でダウンロード |
| `parseCSV(text)` | CSV テキストを 2D 配列に変換（BOM・引用符・改行対応） |
| `exportDayICS()` | 日ビューの作業枠を iCalendar 形式でダウンロード |
| `gistPush()` | 全データを GitHub Gist に保存（初回 POST、以降 PATCH）。成功後に `_localDirty = false` |
| `gistPull()` | GitHub Gist からデータを読み込み上書き。成功後に `_localDirty = false` |
| `gistFindId()` | トークンから `todo_app_data.json` を持つ Gist を自動検索 |
| `saveGistToken()` | Gist トークン・ID を localStorage に保存 |

### その他ユーティリティ

| 関数 | 説明 |
|------|------|
| `renderAll()` | TODOリスト・カレンダー・目標グリッドを全再描画 |
| `renderTodoList()` | `parentId: null` の親タスクのみ表示。子タスク数バッジも描画 |
| `renderGoalGrid()` | 目標グリッドを描画。件数は `maxNum` から動的に算出 |
| `h(str)` | HTML エスケープ（XSS対策） |
| `dateStr(d)` | Date → `YYYY-MM-DD` 文字列 |
| `today()` | 今日の `YYYY-MM-DD` 文字列 |
| `pad(n)` | 2桁ゼロパディング |
| `showToast(msg)` | 2.5秒のトーストを表示 |
| `uuid()` | `crypto.randomUUID()` のラッパー |
| `buildThemeSelect(id, cat, selected)` | テーマ `<select>` を動的に構築 |
| `buildGoalSelect(id, selected)` | 目標 `<select>` を動的に構築 |
| `buildProjectSelect(id, selected)` | プロジェクト `<select>` を動的に構築（active のみ） |
| `onCategoryChange(prefix)` | カテゴリ変更時のテーマ・プロジェクト選択の表示/非表示切替 |

---

## 設計方針

### 1. シングルファイル原則
HTML / CSS / JS をすべて `index.html` 1ファイルにまとめる。ビルドツール・外部JSライブラリは使わない。

### 2. 状態は `S` オブジェクトに集約
グローバルな状態変数は `S` に統一する。モーダル管理のみ `_blockEditId` などのプレフィックス付き変数を使用。

### 3. 繰り返しはインスタンスを生成しない
Task・Block の繰り返しは、1件のオブジェクトを `isOccurrenceOnDate()` / `isBlockOnDate()` で展開して表示する。
Occurrence（発生日インスタンス）を別オブジェクトとして生成・保存しない。

### 4. 時刻管理は Block に統一
タスクの `scheduledTime` / `scheduledEndTime` は廃止済み。時刻はすべて Block の `startTime` / `endTime` で管理する。
タスクは「どの日に作業するか（`scheduledDate`）」のみ持つ。

### 5. 親子タスクの管理
- 子タスクは `parentId` フィールドで親を参照する（親側は子の参照を持たない）。
- TODOリストには `parentId: null` の親タスクのみ表示し、子タスク数は `▶ N件` バッジで示す。
- サブタスクは親タスクの編集モーダル下部セクションで管理する。

### 6. 完了判定の優先順位（`isTaskDone`）
1. `t.completed === true` → 強制完了（繰り返しタスクにも適用）
2. 繰り返しあり・日付指定あり → `completions` ログで判定
3. 繰り返しあり・日付指定なし → 全発生日が完了済みか判定
4. 繰り返しなし → `t.completed` のみ

### 7. レスポンシブ
- 800px 以上: TODO左右分割（TODOパネル 370px + カレンダー）
- 800px 未満: 縦積み + タブ切替
- `100dvh` で iOS Safari アドレスバー対応済み
- ホバーは `@media (hover: hover)` でマウス操作限定

### 8. モーダルの開閉
`.modal-overlay` に `.open` クラスを付与/除去で制御。`closeAllModals()` で全モーダルを一括閉鎖。

### 9. Goal の type システム
Goal には `type`（達成型 / 習慣型 / 探索型）を任意で設定できる。

- **UI**: 追加・編集モーダルのラジオボタンで選択。探索型のみ「状況メモ（`progressNote`）」テキストエリアを表示。
- **グリッドバッジ**: 各セルに種類バッジを表示（達成=青 / 習慣=緑 / 探索=橙）。`type: null` は非表示。
- **AI分解**: `buildDecomposePrompt(goal)` で type に応じたプロンプトを生成。
  - `achievement`: 具体的なアクション 5〜10件
  - `habit`: 12ヶ月分の繰り返しタスク。AIに `month`（1〜12整数）を返させ、コード側で `scheduledDate`（月初）と `recurrenceEnd`（月末・うるう年対応）を計算
  - `explore`: `progressNote` を踏まえた次のアクション 5〜10件
  - `null`: 汎用プロンプト
- **今月提案**: `suggestGoals()` → `buildSuggestPrompt()` で未着手を優先・進行中を補足として2セクションに分けてGeminiに送信。結果モーダルにステータス・種類バッジを表示し、「🤖 AI分解する」で即座にAI分解へ遷移。

### 10. 未保存通知（_localDirty フラグ）
- データ変更系の save 関数（`saveTasks()` 等）が呼ばれるたびに `markDirty()` が実行される。
- `markDirty()` は `_localDirty = true` にセットし、Gistトークン・ID設定済みの場合のみヘッダー下部に未保存バナー（`banner-push`）を表示する。
- `gistPush()` / `gistPull()` / JSONインポート の成功後に `_localDirty = false` にリセットしてバナーを非表示。
- `beforeunload` イベントで `_localDirty === true` のとき離脱確認ダイアログを表示。

### 11. 外部 API
| API | 用途 | キー |
|-----|------|------|
| Gemini API | AIタスク分解・Goal提案・振り返りアドバイス | `todo_gemini_api_key` |
| Wikipedia Geosearch (ja) | 旅モード: 観光スポット | 不要 |
| Nominatim (OpenStreetMap) | 旅モード: 地名→座標 / グルメPOI | 不要 |
| GitHub Gist API | データ同期 | `todo_gist_token` |

**Gemini APIキーは [AI Studio](https://aistudio.google.com) で作成したもののみ無料枠有効**（Cloud Console 発行のキーは quota=0）。

---

## CSV フォーマット（目標入出力）

```
番号,タイトル,カテゴリ,ステータス,テーマ,メモ
1,目標名,仕事,進行中,設備計画,メモ内容
```

- ステータス値: `未着手` / `進行中` / `達成`
- UTF-8 BOM付き（Excel 対応）
- 番号が重複・欠損の場合は自動で次の空き番号に割り当て
- `type` / `progressNote` は CSV 非対応（JSON バックアップで保持）

---

## JSON バックアップ構造

```json
{
  "tasks": [...],
  "goals": [...],
  "blocks": [...],
  "completions": [...],
  "themes": { "work": [...], "personal": [...] },
  "projects": [...],
  "reviews": [...],
  "exportedAt": "ISO8601"
}
```

---

## GitHub Gist 同期

- ファイル名: `todo_app_data.json`（非公開 Gist）
- 初回保存: POST → Gist ID を `todo_gist_id` に自動保存
- 以降: PATCH で更新
- iPhone での初回: 「🔍 検索」ボタンで既存 Gist ID を自動取得
- 起動時: `checkGistSync()` で Gist の最終更新日時とローカルの `todo_last_saved` を比較し、差異があればバナーで通知

---

## ICS エクスポート仕様

- 対象: 日ビュー表示中の日の **作業枠のみ**（タスク単体は出力しない）
- 1作業枠 = 1 VEVENT（タスク名は DESCRIPTION に改行区切りで列挙）
- タイムゾーン: `Asia/Tokyo`（VTIMEZONE セクション付き）
- ファイル名: `schedule_YYYYMMDD.ics`
