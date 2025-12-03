# リッチUML実装ガイド

## 技術スタック概要
- **Next.js 14 + React 18**: App Router構成でインタラクティブなダイアグラムページをSSR/ISR両対応させる。
- **React Flow**: ノード/エッジモデルでUMLライクなステート遷移を表現。ズームやパン、ミニマップを備えたリッチな操作体験を提供。
- **Tailwind CSS + shadcn/ui**: レイアウトとテーマを柔軟に調整し、ライト/ダーク両モードでの閲覧性を担保する。
- **Mermaid CLI (ドキュメント同期用)**: Markdownに残す静的図版とReact Flow構成を併存させ、設計ドキュメントとUIの差分を減らす。
- **Storybook 8**: コンポーネント単位の振る舞い確認とデザイナー/エンジニア間のコミュニケーション向上。

## ディレクトリ提案
```
apps/web/app/(ux)/user-journey/page.tsx     # Next.jsページ
apps/web/components/diagrams/UserJourneyFlow.tsx
apps/web/styles/globals.css                # Tailwindベース
packages/ui/storybook/ux/UserJourneyFlow.stories.tsx
```

## 主要コンポーネント
- `UserJourneyFlow`（`output/タマネギアプリ/UX/ユーザー体験フローReact.tsx`）  
  - React Flowの`Node`と`Edge`を用いてアプリの状態遷移を構成。  
  - `MiniMap`/`Controls`/`Background`を有効化し、ズーム・パン・グリッド表示を提供。  
  - Tailwindクラスでレスポンシブな表示とダークモードサポート。

- `UxDiagramLayout`  
  - shadcn/uiの`Card`や`Tabs`を活用し、テキスト解説・研究根拠リスト・図の切り替えをまとめるレイアウト。

## 実装ステップ
1. Next.jsプロジェクトに`reactflow`, `tailwindcss`, `@tanstack/react-query`（データ取得用）を導入。
2. `UserJourneyFlow`コンポーネントを`app/(ux)/user-journey/page.tsx`で読み込み、UXセクションに配置。
3. Mermaidで記述した静的図（`ユーザー体験フローUML.md`）とReact Flowのノード/エッジ定義を同じソースから生成できるよう、JSONスキーマ化を検討。
4. Storybookに図の状態（フィルタ済み・警告表示・別候補リクエストなど）を分岐させたバリエーションを追加。
5. E2Eテスト（Playwright）でパン・ズーム・エッジクリックのアクセシビリティを確認し、キーボード操作フォールバックも検証。

## アクセシビリティ
- ノードに`aria-label`を付与し、スクリーンリーダーにステート遷移を読み上げさせる。
- キーボード操作でミニマップのフォーカス移動、エッジ選択・解除が可能か確認。
- 色覚多様性を考慮したカラーリング（Tailwindの`slate`系＋アクセント色）を使用。

## 拡張アイデア
- `reactflow`の`useNodesState`を活用し、フィルタリングに応じて表示ノードを切り替え（例: 育てるフローのみ表示）。
- `zustand`で表示モードやハイライト状態を管理し、リフレクション結果からノードを発光させるインジケーターを追加。
- WebGLベースのアニメーション（`react-spring`や`framer-motion`）でエッジ遷移を強調。

## ビルド/デプロイ考慮
- Mermaid CLIをCIに組み込み、Markdown図とReact Flowデータの差異を検出。
- React FlowはSVGベースなのでISR/SSGで初期HTMLを生成した後、クライアントでインタラクティブ化する。  
- 図の更新頻度が高い場合はCMS（Contentlayerなど）からノード/エッジ定義を取得する仕組みを検討。
