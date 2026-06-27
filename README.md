# 上富田AIナビ

上富田町専属のAIアシスタントアプリ。毎日、上富田町に関する情報をAIが自動で収集・整理し、利用者一人ひとりに必要な情報だけを分かりやすく届けます。

---

## 技術スタック

- **React 18 + TypeScript**
- **Vite** — 高速開発環境
- **Tailwind CSS** — スタイリング
- **vite-plugin-pwa** — PWA対応
- **Lucide React** — アイコン
- **Anthropic Claude API** — AI要約・コメント生成

---

## 開発環境の起動方法

### 1. 依存パッケージのインストール

```bash
npm install
```

### 2. 環境変数の設定（不要）

このアプリはAnthropic APIキーをクライアントに持たず、APIプロキシ経由で動作します。
`src/services/aiService.ts` の `API_ENDPOINT` を必要に応じて変更してください。

### 3. 開発サーバーの起動

```bash
npm run dev
```

ブラウザで http://localhost:5173 を開いてください。

---

## GitHubへのPush方法

```bash
# 初回
git init
git add .
git commit -m "feat: 上富田AIナビ 初期実装"
git remote add origin https://github.com/<あなたのユーザー名>/kami-navi.git
git branch -M main
git push -u origin main

# 2回目以降
git add .
git commit -m "fix: ○○を修正"
git push
```

---

## Vercelへの公開方法

### 方法A：Vercel CLIを使う

```bash
npm install -g vercel
vercel login
vercel
```

### 方法B：GitHubと連携する（推奨）

1. [vercel.com](https://vercel.com) にアクセスしてログイン
2. 「Add New Project」→ GitHubリポジトリを選択
3. Framework Preset: **Vite** を選択
4. 「Deploy」ボタンを押すだけ

以降、`main` ブランチへの push で自動デプロイされます。

---

## ニュース取得元の追加方法

`src/services/newsService.ts` にプラグイン形式で追加できます。

### ステップ1：Fetcherクラスを作成

```typescript
class NhkWakayamaFetcher implements NewsFetcher {
  source: NewsSource = 'nhk-wakayama'
  sourceName = 'NHK和歌山'

  async fetch(): Promise<RawArticle[]> {
    // RSSフィードを取得してパース
    const res = await fetch('https://www3.nhk.or.jp/lnews/wakayama/rss/news.xml')
    const text = await res.text()
    const parser = new DOMParser()
    const xml = parser.parseFromString(text, 'application/xml')
    const items = xml.querySelectorAll('item')

    return Array.from(items).map(item => ({
      title: item.querySelector('title')?.textContent ?? '',
      summary: item.querySelector('description')?.textContent ?? '',
      url: item.querySelector('link')?.textContent ?? '',
      publishedAt: new Date(item.querySelector('pubDate')?.textContent ?? ''),
      category: 'other' as NewsCategory,   // AIが後で分類
      importance: 2,
    }))
  }
}
```

### ステップ2：`NEWS_FETCHERS` 配列に追加

```typescript
const NEWS_FETCHERS: NewsFetcher[] = [
  new KamitondaOfficialFetcher(),
  new DisasterInfoFetcher(),
  new EventInfoFetcher(),
  new NhkWakayamaFetcher(), // ← 追加するだけ
]
```

### ステップ3：型定義に新ソースを追加（必要な場合）

`src/types/index.ts` の `NewsSource` 型に追加：

```typescript
export type NewsSource =
  | 'kamitonda-official'
  | 'wakayama-pref'
  | 'nhk-wakayama'   // ← 追加
  | ...
```

`src/utils/constants.ts` の `SOURCE_LABELS` にも表示名を追加。

---

## プロジェクト構造

```
src/
├── types/          # TypeScript型定義
│   └── index.ts
├── services/       # データ取得・AI・ストレージ
│   ├── aiService.ts      # Claude API連携
│   ├── newsService.ts    # ニュース取得（追加しやすい構造）
│   └── storageService.ts # localStorage操作
├── hooks/          # カスタムフック
│   └── useApp.ts         # アプリ全体の状態管理
├── utils/          # ユーティリティ
│   ├── constants.ts      # ラベル・色定義
│   └── relevance.ts      # 関連度スコア計算
├── components/     # 再利用UIコンポーネント
│   ├── ui.tsx            # 汎用コンポーネント群
│   ├── NewsCard.tsx      # ニュースカード
│   ├── DailySummaryCard.tsx
│   └── BottomNavBar.tsx
└── pages/          # 画面
    ├── OnboardingScreen.tsx
    ├── HomeScreen.tsx
    ├── SearchScreen.tsx
    ├── FavoritesScreen.tsx
    └── SettingsScreen.tsx
```

---

## 今後追加予定の機能

アプリは拡張しやすい構造で作られています。以下の機能を追加予定：

- 🌤 今日何着ていく（天気連携）
- 📅 イベントカレンダー
- 🗑 ごみの日リマインダー
- 🌊 河川水位情報
- 🐻 熊目撃情報マップ
- 🗺 観光マップ
- 🏪 町内店舗検索
- 💬 AIチャット相談
