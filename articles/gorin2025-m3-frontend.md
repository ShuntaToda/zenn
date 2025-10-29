---
title: "第63回 技能五輪全国大会ウェブデザイン職種のM3フロントエンド課題を解説してみる"
emoji: "🧑‍💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react"]
published: true
---

こんにちは！[戸田](https://x.com/shuntemskills)です。

前回に引き続き第63回 技能五輪全国大会ウェブデザイン職種のM3フロントエンド課題を解説していきます。

**前回のM2バックエンド課題の解説はこちら↓**

https://zenn.dev/shun_tem/articles/gorin2025-m3-backend

**第63回 技能五輪全国大会ウェブデザイン職種の課題一覧はこちら↓**
https://www.javada.or.jp/jigyou/gino/zenkoku/n_63/kadai/39.html


**技能五輪全国大会公式サイト**

https://worldskills.jp/nationalskills/

## 課題の概要

M3課題はReactを使用したフロントエンドをメインとしたウェブアプリケーションを作成する課題です。

写真スライドショーを表示し、複数のテーマ（基本・フェード・ぼかし）から選択可能で、キーボード操作、ドラッグ&ドロップ、設定パネルなどの機能を実装します。


## 主に利用する技術

- React 19
- React Router 7（ページ遷移）
- Tailwind CSS 4（スタイリング）

## 実装の解説

ここからはReactを利用した実装の解説をします。

解説に利用する参考リポジトリ

https://github.com/ShuntaToda/gorin2025-m3-frontend

:::message
今回解説する実装だけが正解ではありません。様々な実装方法の中から適切な実装を選択できる力が技能五輪では求められます。
:::

### プロジェクト構成

今回作成したプロジェクトの構成は以下の通りです。

```
src/
├── api/                          # API通信管理
│   └── photoApi.js
├── components/                   # Reactコンポーネント
│   ├── Slideshow.jsx             # スライドショー表示
│   ├── SettingsPanel.jsx         # 設定パネル
│   └── PhotoDetail.jsx           # 写真詳細
├── hooks/                        # カスタムフック
│   ├── usePhotos.js              # 写真データ管理
│   ├── useSettings.js            # 設定管理
│   └── useKeyboardShortcuts.js   # キーボード操作
├── App.jsx                       # ルーティング管理
├── main.jsx                      # エントリーポイント
└── index.css                     # グローバルスタイル
```

### ルーティング

ページ遷移はReact Routerで管理します。
React Routerでは色々なルーティング方法がありますが、シンプルなDeclarative Modeを使用しました。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/App.jsx#L121-L124

メインページのURL `/` では写真スライドショーを表示し、写真詳細ページのURL `/photo/:id` では選択した写真の詳細情報を表示します。

`BrowserRouter`でReact Routerを初期化し、ページ遷移機能を有効化します。

https://reactrouter.com/start/declarative/routing#configuring-routes


### API通信の実装

API通信の管理は `photoApi.js` で行います。
大会でも実務でもAPI通信は重要な役割を担っています。
必ず理解しておきましょう。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/api/photoApi.js#L1-L137

#### 主な機能

- `fetchApi()` - 汎用的なfetchラッパー関数
  - エラーハンドリングを統一的に行う
  - Content-Typeヘッダーを自動設定
  - HTTPエラーをチェック

- `resolveImageUrl()` - 画像URLを完全なURLに変換
  - 相対パスの場合はバックエンドのベースURLを付与
  - すでに完全なURLの場合はそのまま返す

- `getPhotos()` - 写真一覧を取得
- `getPhotoById()` - 写真詳細を取得
- `getThemes()` - テーマ一覧を取得
- `getSettings()` - ユーザー設定を取得
- `saveSettings()` - ユーザー設定を保存

https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch


### コンポーネントとカスタムフック

本来の手順では複数のコンポーネントを作りつつ機能を段階的に実装していきますが、解説としては複雑になってしまうので1つのコンポーネントごとに解説します。

https://ja.react.dev/learn/your-first-component

https://ja.react.dev/learn/reusing-logic-with-custom-hooks

### 状態管理とコンポーネント階層

M3フロントエンドの状態管理とコンポーネント構造は以下のようになっています。

```
App（ルーティング管理）
│
├── MainPage（親コンポーネント）
│   │
│   ├── usePhotos フック
│   │   ├── photos（配列）
│   │   ├── currentPhotoIndex（インデックス）
│   │   ├── loading（読み込み状態）
│   │   ├── error（エラー情報）
│   │   ├── addPhoto()（ドラッグ&ドロップで使用）
│   │   └── shufflePhotos()（ランダム再生時）
│   │
│   ├── useSettings フック
│   │   ├── settings（テーマ、スライド間隔、再生モード）
│   │   ├── themes（テーマ一覧）
│   │   ├── loading（読み込み状態）
│   │   ├── error（エラー情報）
│   │   └── updateSettings()（API保存）
│   │
│   ├── Slideshow コンポーネント
│   │   ├── Props受け取り
│   │   │   ├── photos（配列）
│   │   │   ├── settings（設定）
│   │   │   └── currentPhotoIndex（現在のインデックス）
│   │   ├── 内部状態
│   │   │   ├── animationPhase（アニメーション段階：exiting/entering/null）
│   │   │   └── isDragOver（ドラッグオーバー状態）
│   │   └── useKeyboardShortcuts フック
│   │       └── 左右矢印キー操作
│   │
│   └── SettingsPanel コンポーネント
│       ├── Props受け取り
│       │   ├── themes（テーマ一覧）
│       │   └── settings（現在の設定）
│       └── 内部状態
│           ├── intervalInput（スライド間隔入力値）
│           └── intervalError（バリデーションエラー）
│
└── PhotoDetail コンポーネント（ルート: /photo/:id）
    ├── Props取得
    │   └── useParams() で photo ID を取得
    └── 内部状態
        ├── photo（取得した写真詳細データ）
        ├── loading（読み込み状態）
        ├── failed（失敗状態）
        └── navigate（React Router）
```

このような構成により、MainPageで一元管理されたデータが、各子コンポーネントにPropsとして流れます。各コンポーネントは必要に応じて内部状態を持ち、UI操作を管理します。PhotoDetailコンポーネントはルーティングにより別画面に表示されます。


### カスタムフックについて

- `usePhotos`、`useSettings`、`useKeyboardShortcuts`は、データ取得とロジックをコンポーネントから分離したカスタムフックです。
- コンポーネント内に直接APIコールやstate管理を書くのではなく、hooksに記述することで、コードの再利用性と保守性が向上します。

### usePhotos カスタムフック

写真データと操作ロジックを管理するカスタムフックです。以下の状態と関数を返します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/hooks/usePhotos.js#L15-L67


#### 返却値

- `photos` 写真の配列
- `loading` 読み込み中フラグ
- `error` エラー情報
- `currentPhotoIndex` 現在の写真インデックス
- `addPhoto()` 写真を追加する関数
- `shufflePhotos()` 写真をシャッフルする関数

### `usePhotos`の内部で使用しているReactのhooks

#### useState

複数の状態を管理（photos、loading、error、currentPhotoIndex）
アプリケーション全体で使用される最重要なhooksです。

https://ja.react.dev/reference/react/useState

##### usePhotosで利用しているuseStateの使い方
- `const [photos, setPhotos] = useState([])`
  - 取得した写真データの配列を保持します
  - 初期値は空の配列`[]`で、APIからデータを取得後に更新されます
- `const [loading, setLoading] = useState(true)`
  - データを読み込み中かどうかを示すboolean値です
  - 初期値は`true`で、APIコール中は読み込み中状態を表します
- `const [error, setError] = useState(null)`
  - APIコール時にエラーが発生した場合、エラー情報を保持します
  - 初期値は`null`で、エラーが発生した時にエラーオブジェクトが設定されます
- `const [currentPhotoIndex, setCurrentPhotoIndex] = useState(0)`
  - 現在表示している写真の配列インデックスを保持します
  - 初期値は`0`で、最初の写真から始まります
  - ユーザーが矢印キーで操作する、またはスライドショーが自動切り替えする際に更新されます

#### useEffect

コンポーネント初期化時に写真データをAPIから取得します。

https://ja.react.dev/reference/react/useEffect

##### usePhotosで利用しているuseEffectの使い方

- 依存配列が空`[]`のため、マウント時に1回だけ実行されます
- 実行順序は以下の通りです
  1. 最初に`loading`を`true`に設定して、読み込み中状態を示す
  2. `getPhotos()` APIを呼び出して、サーバーから写真データを取得（後で解説）
  3. 取得に成功したら`setPhotos(data)`で状態を更新し、`loading`を`false`に変更
  4. 取得に失敗した場合は`error`にエラー情報を保存し、`loading`を`false`に変更
- このパターンにより、初回レンダリング時に自動的にデータが読み込まれ、その後コンポーネントに反映される仕組みになっています

### useSettings カスタムフック

設定とテーマを管理するカスタムフックです。以下の状態と関数を返します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/hooks/useSettings.js#L14-L66


#### 返却値
- `settings` 設定オブジェクト（テーマID、スライド間隔、再生モード）
- `themes` テーマ一覧の配列
- `loading` 読み込み中フラグ
- `error` エラー情報
- `updateSettings()` 設定を更新してサーバーに保存する関数

### useSettingsで利用しているhooksの使い方

#### useState
- `const [settings, setSettings] = useState({ themeId: "A", slideInterval: 500, playMode: "auto" })`
  - ユーザーの設定情報を保持するオブジェクトです
  - 初期値には、テーマID、スライド間隔、再生モードが含まれます
  - ユーザーが設定を変更すると、このオブジェクトが更新され、アプリ全体に反映されます

- `const [themes, setThemes] = useState([])`
  - テーマ一覧を保持する配列です
  - 初期値は空の配列`[]`で、APIからデータを取得後に更新されます
  - SettingsPanelコンポーネントでテーマ選択肢を表示するのに使用されます

- `const [loading, setLoading] = useState(true)`
  - 設定とテーマデータの読み込み中かどうかを示すboolean値です
  - 初期値は`true`で、APIコール中は読み込み中状態を表します

- `const [error, setError] = useState(null)`
  - APIコール時にエラーが発生した場合、エラー情報を保持します
  - 初期値は`null`で、エラーが発生した時にエラーオブジェクトが設定されます

#### useEffect

- 依存配列が空`[]`のため、マウント時に1回だけ実行されます
- 実行順序は以下の通りです
  1. `Promise.all()`を使用して`getSettings()`と`getThemes()`を同時に実行
     - 複数のAPI呼び出しを並行実行することで、処理時間を短縮できます
     - 両方のAPIが完了するまで次の処理に進みません
  2. 取得に成功したら`setSettings(settingsData)`と`setThemes(themesData)`で状態を更新し、`loading`を`false`に変更
  3. 取得に失敗した場合は`error`にエラー情報を保存し、`loading`を`false`に変更
- このパターンにより、初回レンダリング時に自動的に設定とテーマデータが読み込まれ、その後UIが更新される仕組みになっています


### useKeyboardShortcuts カスタムフック

キーボードイベントの登録・削除を管理するカスタムフックです。このフックは2つの場所で異なる目的で使用されます。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/hooks/useKeyboardShortcuts.js

#### 返却値

このフックは値を返さず、引数として受け取ったハンドラ関数を管理して、グローバルキーボードイベントを監視します。

- `handler` - キーボードイベント（keydown）のハンドラ関数を引数として受け取る

### useKeyboardShortcutsで利用しているhooksの使い方

#### useEffect

- 依存配列が空`[]`のため、マウント時に1回だけ実行されます
- 実行順序は以下の通りです
  1. `window.addEventListener("keydown", handler)`でグローバルキーボードイベントを監視
  2. コンポーネントアンマウント時に`window.removeEventListener("keydown", handler)`でイベントリスナーを削除
- このパターンにより、メモリリークを防ぎながら、ハンドラ関数の更新に対応できます。


### MainPage コンポーネント

メイン画面のコンポーネントで、スライドショーと設定パネルを統合して管理。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/App.jsx#L13-L112

#### 主な役割

- 複数のカスタムフック（usePhotos、useSettings）を使用してデータを管理
- SlideshowコンポーネントとSettingsPanelコンポーネントをレンダリング
- キーボード操作（数字キー1-9でテーマ切り替え）を処理
- 写真をクリックして詳細画面に遷移

##### 使用しているReactの主要なhooks

- `useNavigate`
  - React Routerで提供されるhookで、プログラムでページ遷移を行います
  - 写真をクリックして詳細画面に遷移する際に使用されます
  - `navigate('/photo/:id')` の形式で使用します
  - 参考ドキュメント: https://reactrouter.com/api/hooks/useNavigate
- `useCallback`
  - （正直この大会においてはレンダリングの効率を求められていないので必要ないですが一応実装してみました）
  - 関数をメモ化（保存）し、不必要な再レンダリングを防ぎます
  - 子コンポーネントに[コールバック関数](https://qiita.com/nakajima417/items/4d0c2d46ff82351549e6)を渡す際に使用すると、親がレンダリングされても同じ関数参照を渡すことができるため、子コンポーネントの無駄なレンダリングを防ぎます
  - 参考ドキュメント: https://ja.react.dev/reference/react/useCallback


#### URLパラメータからIDを取得

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/PhotoDetail.jsx#L10-L11

### Slideshow コンポーネント

スライドショーを表示し、写真の切り替え、キーボード操作、ドラッグ&ドロップを管理します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/Slideshow.jsx#L8-L321

#### 主な機能

- 写真の表示と切り替え
- テーマに応じたアニメーション効果（フェード、ぼかし）
- キーボードの左右矢印キーでの手動制御
- ドラッグ&ドロップによる写真追加
- 自動再生とランダム再生

#### 使用しているReactの主要なhooks

- `useEffect`
- `useState`
- `useRef`
  - 再レンダリング時に値を保持し続けることができる
  - stateとは異なり、値の変更で再レンダリングがトリガーされないため、パフォーマンスが向上します
  - 参考ドキュメント: https://ja.react.dev/reference/react/useRef

#### `timerRef` の解説

このコンポーネントでは、自動再生タイマーのリファレンスを`useRef`で管理しています。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/Slideshow.jsx#L18-L18


##### useRefを使用する理由
- 再レンダリング時に値が保持される（タイマーIDを保持し続けられる）
- 値が変わっても再レンダリングをトリガーしない（パフォーマンス向上）
- `clearInterval`で同じタイマーを参照できる（正確なタイマー制御）
- stateと違い同期的にアクセスできる（即座に値を取得・更新可能）
- 一度useStateを利用して同じ実装をしてみるとuseRefを使用する理由がわかりやすいと思います。


#### アニメーション制御（`getAnimationClass()`）

`getAnimationClass()`メソッドは現在の状態に応じて適切なCSSアニメーションクラスを返します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/Slideshow.jsx#L241-L269

##### getAnimationClass()の処理の流れ

1. **テーマA（基本テーマ）**
   - アニメーションなし（空文字を返す）

2. **テーマB（フェード）**
   - `animationPhase === 'exiting'` 時に `animate-fade-out` クラスを適用
   - `animationPhase === 'entering'` 時に `animate-fade-in` クラスを適用
   - CSSで定義されたアニメーション（500ms）が実行

3. **テーマC（ぼかし）**
   - `animationPhase === 'exiting'` 時に `animate-blur-out` クラスを適用
   - `animationPhase === 'entering'` 時に `animate-blur-in` クラスを適用
   - CSSで定義されたアニメーション（500ms）が実行


https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/index.css#L5-L34

#### 写真切り替えメソッド（`goToNext()`）

`goToNext()`メソッドについて解説します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/Slideshow.jsx#L42-L83

##### goToNext()の引数
- `withAnimation` - アニメーション効果を使用するかどうか
- `resetTimer` - 自動再生タイマーをリセットするかどうか

##### 処理の流れ
1. 画像が存在するかチェック
2. タイマーリセット
3. 画面切り替えの実行
   1. アニメーションなしの場合は即座に切り替え
      1. 手動で実行した際にアニメーションがあると不自然に見えるため、アニメーションなしで即座に切り替えられるようにしています
      2. 仕様としては求められていないので、実装しなくてもいいです
   2. アニメーション付き切り替え
      - `setAnimationPhase('exiting')`で現在の写真の退場アニメーション（フェード・ぼかし）を開始
        - animationPhaseに'exiting'を設定することで、CSSのClass名が付与され、CSSアニメーションが適用されます
      - 500msの退場アニメーション時間待つ
        - CSSでは500msと設定していますが、Reactのレンダリングのずれが発生すると一瞬画像が表示されてしまうことがあったので、DOMの処理は490msで実行されるように修正しています
      - インデックスを更新して次の写真に切り替え
      - `setAnimationPhase('entering')`で入場アニメーションを開始
      - アニメーション完了後に`setAnimationPhase(null)`でリセット
4. 次の写真のインデックスを計算
   1. `calculateNextPhotoIndex()`で次の写真のインデックスを計算
   2. インデックスを更新して次の写真に切り替え

#### ドラッグ&ドロップ（`handleDragOver()`、`handleDragLeave()`、`handleDrop()`）

`handleDragOver()`、`handleDragLeave()`、`handleDrop()`メソッドについて解説します。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/Slideshow.jsx#L196-L226

##### handleDragOver() - ドラッグ中の処理

**役割**
- ユーザーがファイルをドラッグしながらスライドショー領域の上に重ねている状態を検出

**処理内容**
1. `e.preventDefault()` - デフォルト動作をキャンセル
   - ブラウザのデフォルト動作では、ドラッグ中のファイルはドロップできない仕様(コメントアウトなどして試してみてください)
   - `preventDefault()`を呼ぶことで、ブラウザ側のデフォルト処理をキャンセルし、ドロップを許可する

2. `setIsDragOver(true)` - ドラッグ状態を有効化
   - UIで「ここにドロップできます」という視覚的フィードバックを表示するための状態管理

##### handleDragLeave() - ドラッグを離れた時の処理

**役割**
- ユーザーがファイルをドラッグして領域外に出た場合を検出

**処理内容**

1. `setIsDragOver(false)` - ドラッグ状態を無効化
   - UIをもとの状態に戻す
   - ボーダーと背景色が元に戻る

##### handleDrop() - ドロップ完了時の処理

**ファイル取得と検証**

- `e.dataTransfer.files` - ドロップされたファイル一覧を取得
- 複数ファイルをドロップされても、最初のファイル（`files[0]`）のみ使用
- `file.type.startsWith('image/')` - MIME typeで画像ファイルかチェック
  - `image/jpeg`、`image/png`など、`image/`で始まるファイルのみ許可
  - テキストファイルなど画像以外は拒否

**ファイル読み込み**
- `FileReader()` FileReaderオブジェクトを作成し、ブラウザ内でファイルを非同期に読み込む
  - 参考ドキュメント: https://developer.mozilla.org/ja/docs/Web/API/FileReader
- `reader.onload` - ファイル読み込み完了後に実行するコールバック
  - `event.target.result` - 変換されたData URL
- `reader.readAsDataURL(file)` - ファイルを読み込むためのメソッドを呼び出す
- 新しい写真オブジェクトを作成して、親コンポーネントに渡す

- ドロップ完了後、ドラッグ状態UIをリセット

##### ドラッグ&ドロップの全体フロー

```
ユーザーがファイルをドラッグ
    ↓
handleDragOver() - isDragOver = true（視覚的フィードバック開始）
    ↓
領域内で継続ドラッグ
    ↓
handleDragOver() が連続発火（isDragOver = true を維持）
    ↓
ファイルをドロップ
    ↓
handleDrop() - ファイル読み込み → 新規写真追加
    ↓
setIsDragOver(false)（UI状態リセット）
```

または

```
ユーザーがファイルをドラッグして領域外に移動
    ↓
handleDragLeave() - isDragOver = false（視覚的フィードバック終了）
```

#### SettingsPanel コンポーネント

設定パネルを表示し、テーマ、スライド間隔、再生モードを管理。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/SettingsPanel.jsx#L7-L173

#### 主な機能

- テーマ選択（ラジオボタン）
- スライド間隔設定（テキストボックス）
- 再生モード選択（自動再生 / ランダム再生）
- キーボード操作ヒント表示

#### テーマ変更ハンドラー（`handleThemeChange()`）

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/SettingsPanel.jsx#L16-L22

##### 処理の流れ

1. **イベント取得**
   - `e.target.value` - ラジオボタンで選択されたテーマID（'A'、'B'、'C'）を取得

2. **スプレッド演算子で既存設定を保持**
   - `...settings` - 現在の設定オブジェクト全体をコピー
   - これにより、テーマIDのみ変更し、他の設定（スライド間隔、再生モード）は保持

3. **親コンポーネントに即座に通知**
   - `onSettingsChange()`を呼び出して変更を通知
   - 親コンポーネント（MainPage）がこれを受け取り、APIに保存

##### Enterキー押下時の処理

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/SettingsPanel.jsx#L16-L22

- テキストボックスで`Enter`キーが押された場合、`handleIntervalSubmit()`を呼び出します。
- これにより、ユーザーがEnterキーで設定を確定できるようにしています。

#### PhotoDetail コンポーネント

写真の詳細情報を表示するモーダルコンポーネント。

https://github.com/ShuntaToda/gorin2025-m3-frontend/blob/90575934e126ddc7f44fbc454e729490a0f446e8/frontend/src/components/PhotoDetail.jsx#L7-L143

#### 主な機能

- 写真詳細情報の表示（ファイル名、ファイルサイズ、作成日時、キャプション）
- ESCキーまたは背景クリックでモーダルを閉じる
- 詳細表示中でもブラウザ履歴が正常に動作

#### 使用しているReactの主要なhooks

- `useParams`
  - React Routerで提供されるhookで、URLのパラメータを取得します
  - `const { id } = useParams()` の形式で、ルート定義時の `:id` パラメータにアクセスできます
  - このコンポーネントではURL `/photo/:id` から写真IDを取得し、詳細情報を取得するのに使用されます。


## まとめ

今回は第63回 技能五輪全国大会ウェブデザイン職種のM3フロントエンド課題を解説しました。

この記事を見て大会でも業務でも役立つ知識を身につけられたら幸いです。
次の大会ではいい成績を残せるように頑張ってください！
