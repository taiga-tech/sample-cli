# アーキテクチャレビュー

## 結果: REJECT

## サマリー
TypeScript ランチャー実装（`platform.ts`/`index.ts`）の設計品質は高いが、Rust ビルド成果物 64件・ツールキャッシュ 266件のコミット混入、Rust コアのテスト欠如、リリースパイプラインの信頼性問題という 5件のブロッキング指摘が存在する。これらを解消してから再レビューすること。

## 確認した観点
- [x] 構造・設計
- [x] コード品質
- [x] 変更スコープ
- [x] テストカバレッジ
- [x] デッドコード
- [x] 呼び出しチェーン検証

## 今回の指摘（new）

| # | finding_id | スコープ | 場所 | 問題 | 修正案 |
|---|------------|---------|------|------|--------|
| 1 | ARCH-001 | スコープ内 | `.gitignore` | Rust `target/` ビルド成果物 64件がコミットされている。`.gitignore` に `target/` エントリなし | `.gitignore` に `target/` を追加し、`git rm -r --cached target/` でトラッキング解除 |
| 2 | ARCH-002 | スコープ内 | `.takt/.runtime/`（266件） | pnpm レジストリメタデータ・mise キャッシュ等のマシンローカルファイルがコミットされている。`.takt/.gitignore` のホワイトリスト運用が機能していない | `git rm -r --cached .takt/.runtime/` を実行。ルート `.gitignore` に `.takt/.runtime/` を追加 |
| 3 | ARCH-003 | スコープ内 | `crates/core/src/lib.rs:1` | 新規公開関数 `pub fn greet()` にテストなし | `#[cfg(test)] mod tests` に `assert_eq!(greet(), "Hello from sample-cli-core!")` を追加 |
| 4 | ARCH-005 | スコープ内 | `.changeset/config.json:4` | `"changelog": "@changesets/cli/changelog"` を参照しているが `@changesets/cli` は未インストール。`release.yml` も changeset を使わず直接 `pnpm publish` しており、changeset 設定が完全なデッドインフラ | changeset を使わないなら `.changeset/` ディレクトリを削除。使うなら `devDependencies` に追加し `release.yml` に `changeset version/publish` を組み込む |
| 5 | ARCH-006 | スコープ内 | `.github/workflows/release.yml:97` | `run: pnpm install` に `--frozen-lockfile` がなく、リリース時に lockfile が暗黙更新される可能性がある | `run: pnpm install --frozen-lockfile` に変更 |
| 6 | ARCH-004 | スコープ内 | `packages/cli-linux-x64-gnu/package.json`, `packages/cli-linux-x64-musl/package.json` | `"libc"` フィールドなし。npm が glibc/musl を区別できず Linux x64 ホストに両パッケージがインストールされる（機能破壊なし・ダウンロード2倍） | 各ファイルに `"libc": ["glibc"]` / `"libc": ["musl"]` を追加（Warning 扱い） |
| 7 | ARCH-007 | スコープ内 | `mise.toml:3` | `rust = "latest"` が非再現性ビルドの原因。`release.yml` の `dtolnay/rust-toolchain@stable` も同様 | `rust = "1.85.0"` 等で固定（Warning 扱い） |
| 8 | ARCH-008 | スコープ内 | `.github/workflows/`（PR CI なし） | PR に対して自動テストを実行するワークフローが存在しない。PR 説明の全チェックボックスが未チェック | `.github/workflows/ci.yml` を追加し `pull_request` で `pnpm test` + `cargo test` を実行（Warning 扱い） |

## 継続指摘（persists）
なし（初回レビュー）

## 解消済み（resolved）
なし（初回レビュー）

## REJECT判定条件
ARCH-001〜003, ARCH-005, ARCH-006 の 5件が `new` ブロッキング指摘のため REJECT。