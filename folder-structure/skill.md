---
name: folder-structure
description: コロケーション原則に基づくフォルダ構造規約をrulesに作成し、既存コードの移行計画MDを生成する。新規プロジェクト立ち上げ時や構造整理の議論をしたいときに使う。
version: 2.0.0
---

# Folder Structure Skill

対応フレームワーク: **Next.js App Router** / **Vite + React**

起動すると以下を順番に行う：

1. `.claude/rules/folder-structure.md`（規約ルール）を作成・更新する
2. プロジェクトをスキャンしてルールと照合する
3. `docs/folder-structure-migration.md`（移行計画）を作成する

---

## Workflow

### Step 1: プロジェクトを把握する

- `package.json` を Read してフレームワークを判定する
  - `"next"` が dependencies にある → **Next.js App Router**
  - `"vite"` が devDependencies にある → **Vite + React**
  - 判定できない場合はユーザーに確認する
- `CLAUDE.md` があれば Read してプロジェクト固有の注記を確認する

### Step 2: .claude/rules/folder-structure.md を作成・更新する

`.claude/rules/` ディレクトリがなければ作成し、`folder-structure.md` を書く。
既に存在する場合は内容を確認し、プロジェクトに合わせて更新する。

記載する内容（後述の「規約テンプレート」を参照）：
- 基本方針
- フレームワーク別のフォルダ構造
- フィーチャー内ファイルの配置判断基準
- コンポーネントフォルダの構成
- バレルファイルの書き方
- フックの切り出し基準
- 子コンポーネントの置き場所
- 共通コンポーネントの置き場所
- テストの配置

### Step 3: プロジェクト構造をスキャンする

フレームワークに応じてスキャン対象を変える：

**Next.js App Router**
```bash
find app -name "*.tsx" -o -name "*.ts" | grep -v node_modules | grep -v ".next" | sort
find components -name "*.tsx" -o -name "*.ts" | grep -v node_modules | sort
```

**Vite + React**
```bash
find src -name "*.tsx" -o -name "*.ts" | grep -v node_modules | sort
```

スキャン結果とルールを照合して以下を検出する：

**対象: `app/`（Next.js）または `src/pages/`, `src/features/`（Vite）配下**
- コンポーネントが独立フォルダになっていない
- バレルファイル（`index.ts`）が抜けている
- フックがコンポーネントと別フォルダにある（例: `hooks/useFoo.ts` → コンポーネントと同フォルダにすべき）
- テストが `__tests__/` 以外の場所にある
- 単一コンポーネントだけが使うファイルがコンポーネントフォルダの外に置かれている

**対象外: `components/`（共通コンポーネント）**
- 複数箇所から使われる共通コンポーネントはフラット配置のままでよい（サブフォルダ規約を適用しない）

### Step 4: docs/folder-structure-migration.md を作成する

`docs/` ディレクトリに `folder-structure-migration.md` を作成する。以下の構成で書く：

---

**1. 移行対応表（全ファイル網羅）**

| 現状 | 移行後 | 作業内容 |
|---|---|---|
| `app/foo/components/FooTable.tsx` | `app/foo/FooTable/FooTable.tsx` | フォルダ作成・移動 |
| （新規）| `app/foo/FooTable/index.ts` | バレルファイル作成 |
| `app/foo/hooks/useFooTable.ts` | `app/foo/FooTable/useFooTable.ts` | 移動 |

作業内容の種類: `フォルダ作成・移動` / `バレルファイル作成` / `import パス修正` / `削除`

**2. 移行の進め方**

- ページ単位で移行する（複数ページを同時に触らない）
- 移動後は import パスを合わせて修正する
- バレルファイル（`index.ts`）経由で import されていれば影響範囲は最小限

**3. 未着手チェックリスト**

```markdown
- [ ] FooTable（app/foo/）
- [ ] AddFooForm（app/foo/）
- [ ] BarTable（app/bar/）
```

---

## 規約テンプレート（.claude/rules/folder-structure.md の内容）

### 基本方針

- **コロケーション**: 関連ファイルは同じフォルダに置く
- **機能単位**: コンポーネント・フック・テストをまとめて1フォルダに
- **バレルファイル**: `index.ts` で外部からの import を一本化

### フィーチャー内ファイルの配置判断

| 配置場所 | 基準 |
|---|---|
| フィーチャールート直下 | フィーチャー内の**複数コンポーネント**から参照される共有ファイル |
| コンポーネントフォルダ内 | **そのコンポーネントだけ**が使うファイル |

### フレームワーク別フォルダ構造

**Next.js App Router**

```
app/
  [feature]/
    page.tsx
    schema.ts             ← 複数コンポーネントが参照 → フィーチャールートに
    action.ts             ← 同上
    FeatureTable/
      FeatureTable.tsx
      columns.tsx         ← このコンポーネントだけが使う → フォルダ内に
      useFeatureTable.ts
      index.ts
    AddFeatureForm/
      AddFeatureForm.tsx
      useAddFeatureForm.ts
      index.ts
```

**Vite + React**

```
src/
  pages/
    FeaturePage/
      FeaturePage.tsx
      useFeaturePage.ts
      index.ts
  components/
    common/
    form/
    table/
  features/
    [feature]/
      FeatureTable/
        FeatureTable.tsx
        columns.tsx
        useFeatureTable.ts
        index.ts
```

### コンポーネントフォルダの構成

```
ComponentName/
  ComponentName.tsx        ← UI
  useComponentName.ts      ← ロジック（状態・ハンドラーがある場合のみ）
  ComponentName.module.css ← スタイル（CSS Modules の場合のみ）
  index.ts                 ← バレルファイル
  __tests__/
    ComponentName.test.tsx
```

### バレルファイル

```ts
// index.ts
export { ComponentName } from './ComponentName';
// 型は外部で必要な場合のみ追加
// export type { ComponentNameProps } from './ComponentName';
```

外部からの import は必ず `index.ts` 経由：

```ts
// Good
import { FeatureTable } from '@/app/feature/FeatureTable';

// Bad
import { FeatureTable } from '@/app/feature/FeatureTable/FeatureTable';
```

### フック切り出し基準

| 切り出す | 切り出さない |
|---|---|
| `useState` が複数ある | 状態なし・JSX のみ |
| データ取得（fetch / useQuery） | 単純な表示コンポーネント |
| `onSubmit` などのハンドラー | props を受け取って描画するだけ |
| 外部 API・ライブラリとの連携 | |

### 子コンポーネントの置き場所

- **そのコンポーネント内でのみ使う** → 同じフォルダ内に置く
- **複数箇所から使う** → `components/` 配下に移動

### 共通コンポーネントの分類（`components/`）

複数のページ・コンポーネントから使われる共通コンポーネントは `components/` 配下にフラットに置く。
サブフォルダ規約（`ComponentName/ComponentName.tsx`）は **適用しない**。

```
components/
  common/   ← プロジェクト全体で使う汎用コンポーネント
  form/     ← フォーム関連
  table/    ← テーブル関連
  ui/       ← shadcn/ui などの自動生成（基本触らない）
```

### テストの配置

```
ComponentName/
  __tests__/
    ComponentName.test.tsx
```

### 移行方針

1. **新規作成** は必ず新規則で作る
2. **既存コードに修正・追加が発生したとき** は、そのページ・コンポーネントを合わせて移行する
3. 触らない既存コードも移行対象だが優先度は低い

---

## Notes

- `.claude/rules/folder-structure.md` の生成時は**プロジェクトの実際のファイル名・パス**を例として使うこと
- 移行対応表は**全ファイルを網羅**して移行漏れを防ぐ
- フレームワークが判定できない場合はユーザーに確認してから進める