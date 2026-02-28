# QAレビュー

## 結果: REJECT

## サマリー
`index.ts` の `main()` 関数および `crates/core/src/lib.rs` の `greet()` 関数が完全未テストであり、Rust ビルド成果物（64ファイル）とランタイムキャッシュ（266ファイル）が `.gitignore` 未追加のままコミットされているため、4件のブロッキング問題により差し戻す。

## 確認した観点
| 観点 | 結果 | 備考 |
|------|------|------|
| テストカバレッジ | ❌ | `index.ts` main() 完全未テスト、Rust `greet()` 未テスト |
| テスト品質 | ✅ | `platform.test.ts` は13件・正常/異常/エッジケースを網羅、設計良好 |
| エラーハンドリング | ✅ | `index.ts` のエラーパスは stderr + exit 1 で適切に実装 |
| 保守性 | ❌ | ビルド成果物・ランタイムキャッシュ計330ファイルがリポジトリに混入 |
| 技術的負債 | ❌ | `.gitignore` に `target/`・`.takt/` の追加漏れ |

## 今回の指摘（new）
| # | finding_id | カテゴリ | 場所 | 問題 | 修正案 |
|---|------------|---------|------|------|--------|
| 1 | QA-001 | テストカバレッジ | `packages/cli/src/index.ts` (全41行) | `main()` 関数が完全未テスト。`require.resolve` によるパッケージ解決・バイナリパス構築・`spawnSync` 実行・Win32 分岐・エラーハンドリング（exit 1）がいずれも単体テスト未対応 | 内部ロジックを純粋関数として抽出し、`require.resolve`/`spawnSync`/`process.exit` をモック可能にした上で、パッケージ解決失敗・バイナリ実行失敗・Win32パス・signal kill（`status: null → exit 1`）の各パスをテストする |
| 2 | QA-002 | 保守性 | `.gitignore`（未変更）/ `target/`（64ファイル） | このPRで `Cargo.toml`（Rust workspace）を追加したにもかかわらず `.gitignore` に `target/` を追加していない。コンパイル済みバイナリ `target/release/sample-cli` を含む64ファイルがコミットされている | `.gitignore` に `# Rust\ntarget/` を追加し、既存の `target/` ファイルを全て `git rm -r --cached target/` で追跡除外した上で再コミットする |
| 3 | QA-003 | 保守性 | `.gitignore`（未変更）/ `.takt/`（266ファイル） | `.takt/.runtime/cache/pnpm/`（npmメタデータキャッシュ約179件）および `.takt/runs/`（AI実行ログ約87件）が `.gitignore` 未追加のままコミットされている。リポジトリ汚染・信号対雑音比の著しい悪化 | `.gitignore` に `.takt/.runtime/` と `.takt/runs/` を追加（または `.takt/` 全体を除外）し、既存ファイルを `git rm -r --cached` で除去する |
| 4 | QA-004 | テストカバレッジ | `crates/core/src/lib.rs`（全3行） | 公開関数 `greet()` に `#[cfg(test)]` ブロックが存在しない。Rust 標準の同一ファイルインラインテストすら未追加 | `lib.rs` に以下を追加する：`#[cfg(test)] mod tests { use super::*; #[test] fn greet_returns_expected_message() { assert_eq!(greet(), "Hello from sample-cli-core!"); } }` |

## 継続指摘（persists）
| # | finding_id | 前回根拠 | 今回根拠 | 問題 | 修正案 |
|---|------------|----------|----------|------|--------|
| — | — | — | — | 前回QAレビュー実行なし（初回） | — |

## 解消済み（resolved）
| finding_id | 解消根拠 |
|------------|----------|
| — | 前回QAレビュー実行なし（初回）。なお、PRブランチ内の `.takt/runs/` に格納された過去の testing-review（TAKT開発フェーズ）では TEST-W01/W02/W03（全Warning）が記録されており、本レビューでは参考情報として継承する |

## Warning（非ブロッキング）
| # | finding_id | 場所 | 問題 | 推奨対応 |
|---|------------|------|------|----------|
| 1 | QA-W01 | `.github/workflows/`（CI不在） | PRトリガーのビルド・テスト自動実行ワークフローが存在しない。`release.yml` はタグ `v*` push のみ発火。テスト計画チェックボックスも全て未チェック | `.github/workflows/ci.yml` を追加し、PRで `pnpm install && pnpm --filter @taiga-tech/cli test` を自動実行する |
| 2 | QA-W02 | `packages/cli/src/platform.test.ts:64-84` | `resolveTargetTriple` に異常系テスト（未対応 platform・arch）がない。`resolveEntry()` 経由で間接的には保護済み | `resolveTargetTriple` describeブロックに `expect(() => resolveTargetTriple('freebsd', 'x64', false)).toThrow(...)` を追加 |
| 3 | QA-W03 | `packages/cli/src/platform.test.ts`（全体） | テスト名が `darwin/arm64 → @taiga-tech/cli-darwin-arm64` の矢印表記。意図は読み取れるが `should return X when Y` 形式の方が標準的 | 必須ではないが、命名規約統一の観点で改善を推奨 |

## REJECT判定条件
- QA-001（new）: `packages/cli/src/index.ts` main() 完全未テスト → **REJECT**
- QA-002（new）: `target/` ビルド成果物 64ファイルコミット → **REJECT**
- QA-003（new）: `.takt/` ランタイムキャッシュ 266ファイルコミット → **REJECT**
- QA-004（new）: `crates/core/src/lib.rs` greet() Rustテストなし → **REJECT**