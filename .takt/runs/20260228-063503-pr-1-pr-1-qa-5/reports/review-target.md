# レビュー対象

## 概要
| 項目 | 内容 |
|------|------|
| モード | PR |
| ソース | PR #1 |
| タイトル | implement-rust-cli-npm-dist |
| ラベル | N/A |
| 状態 | OPEN |
| 著者 | taiga-tech |
| ベースブランチ | main ← takt/20260227T1629-implement-rust-cli-npm-dist |
| 変更量 | +26,123 / -1,686（357ファイル） |

## 目的・要件

Rust製CLIバイナリをnpm経由で配布するための基盤を実装する。

### 実装内容（PR説明より）
1. **Rust CLI crate の実装**（`crates/cli`、`crates/core`）
2. **プラットフォーム別 npm パッケージの追加**（darwin-arm64/x64、linux-x64-gnu/musl、win32-x64）
3. **メイン npm CLI パッケージ**（`packages/cli`）の実装（TypeScript、プラットフォーム検出）
4. **GitHub Actions リリースワークフロー**（`.github/workflows/release.yml`）の追加
5. **Changeset 設定**の追加

### テスト計画（著者記載）
- `[ ]` `packages/cli/src/platform.test.ts` のテストがパスすること
- `[ ]` ビルドが成功すること（`turbo build`）
- `[ ]` プラットフォーム検出ロジックの動作確認

> ⚠️ **全チェックボックスが未チェック**。著者による動作確認が未実施の可能性が高い。

## リンクされた Issue

N/A（PR説明に "Closes #N"、"Fixes #N"、"Resolves #N" の記載なし）

## コミット履歴

N/A（PRモード）

## 変更ファイル一覧

### ソースコード・設定ファイル（本質的な変更）

| ファイル | 種別 | 変更行数 |
|---------|------|---------|
| `Cargo.toml` | 追加 | +6 |
| `Cargo.lock` | 追加 | +14 |
| `crates/cli/Cargo.toml` | 追加 | +11 |
| `crates/cli/src/main.rs` | 追加 | +4 |
| `crates/core/Cargo.toml` | 追加 | +4 |
| `crates/core/src/lib.rs` | 追加 | +3 |
| `mise.toml` | 変更 | +1 |
| `.changeset/config.json` | 追加 | +20 |
| `.github/workflows/release.yml` | 追加 | +116 |
| `packages/cli-darwin-arm64/package.json` | 追加 | +11 |
| `packages/cli-darwin-arm64/vendor/aarch64-apple-darwin/sample-cli/.gitkeep` | 追加 | +0 |
| `packages/cli-darwin-x64/package.json` | 追加 | +11 |
| `packages/cli-darwin-x64/vendor/x86_64-apple-darwin/sample-cli/.gitkeep` | 追加 | +0 |
| `packages/cli-linux-x64-gnu/package.json` | 追加 | +11 |
| `packages/cli-linux-x64-gnu/vendor/x86_64-unknown-linux-gnu/sample-cli/.gitkeep` | 追加 | +0 |
| `packages/cli-linux-x64-musl/package.json` | 追加 | +11 |
| `packages/cli-linux-x64-musl/vendor/x86_64-unknown-linux-musl/sample-cli/.gitkeep` | 追加 | +0 |
| `packages/cli-win32-x64/package.json` | 追加 | +11 |
| `packages/cli-win32-x64/vendor/x86_64-pc-windows-msvc/sample-cli/.gitkeep` | 追加 | +0 |
| `packages/cli/package.json` | 追加 | +32 |
| `packages/cli/src/index.ts` | 追加 | +41 |
| `packages/cli/src/platform.ts` | 追加 | +43 |
| `packages/cli/src/platform.test.ts` | 追加 | +84 |
| `packages/cli/tsconfig.json` | 追加 | +15 |
| `packages/cli/tsup.config.ts` | 追加 | +9 |
| `pnpm-lock.yaml` | 変更 | +1816 -1685 |
| `turbo.json` | 変更 | +7 -1 |

### ⚠️ 本来コミットすべきでないファイル（重大問題）

| カテゴリ | ファイル数 | 代表例 |
|---------|----------|--------|
| `target/` Rust ビルド成果物 | **約64ファイル** | `target/release/sample-cli`（コンパイル済みバイナリ）、`target/release/deps/libsample_cli_core-*.rlib`、`.rmeta`、インクリメンタルビルドデータ |
| `.takt/.runtime/cache/pnpm/metadata*` | **約179ファイル** | `metadata-full-v1.3/registry.npmjs.org/@esbuild/darwin-arm64.json` など |
| `.takt/runs/` 過去の実行ログ | **約82ファイル** | 前回開発セッションのポリシー・レポート・AI レスポンス |
| `.takt/.runtime/` ランタイム状態 | **複数** | `env.sh`、`state/mise/tracked-configs/*`、`tmp/node-compile-cache/*` |

> ⚠️ `.gitignore` に `target/` が未追加。Rust ビルド成果物（コンパイル済みバイナリ含む）がそのままコミットされている。

## 差分

### `crates/core/src/lib.rs`（新規）
```rust
pub fn greet() -> String {
    "Hello from sample-cli-core!".to_string()
}
```

### `crates/cli/src/main.rs`（新規）
```rust
fn main() {
    let message = sample_cli_core::greet();
    println!("{}", message);
}
```

### `crates/cli/Cargo.toml`（新規）
```toml
[package]
name = "sample-cli"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "sample-cli"
path = "src/main.rs"

[dependencies]
sample-cli-core = { path = "../core" }
```

### `Cargo.toml`（新規）
```toml
[workspace]
resolver = "2"
members = [
    "crates/core",
    "crates/cli",
]
```

### `mise.toml`（変更）
```diff
 [tools]
 node = "24.14.0"
+rust = "latest"
```

### `packages/cli/src/platform.ts`（新規・43行）
```typescript
const SCOPE = '@taiga-tech' as const

const PLATFORM_ENTRIES: Readonly<Record<string, { pkg: string; triple: string }>> = {
  'darwin/arm64': { pkg: `${SCOPE}/cli-darwin-arm64`, triple: 'aarch64-apple-darwin' },
  'darwin/x64':   { pkg: `${SCOPE}/cli-darwin-x64`,  triple: 'x86_64-apple-darwin' },
  'win32/x64':    { pkg: `${SCOPE}/cli-win32-x64`,   triple: 'x86_64-pc-windows-msvc' },
  'linux/x64/gnu':  { pkg: `${SCOPE}/cli-linux-x64-gnu`,  triple: 'x86_64-unknown-linux-gnu' },
  'linux/x64/musl': { pkg: `${SCOPE}/cli-linux-x64-musl`, triple: 'x86_64-unknown-linux-musl' },
}

const SUPPORTED_PLATFORMS = new Set(['darwin', 'win32', 'linux'])

function buildKey(platform: string, arch: string, isGlibc: boolean): string {
  const libcSuffix = platform === 'linux' ? `/${isGlibc ? 'gnu' : 'musl'}` : ''
  return `${platform}/${arch}${libcSuffix}`
}

function resolveEntry(platform: string, arch: string, isGlibc: boolean): { pkg: string; triple: string } {
  if (!SUPPORTED_PLATFORMS.has(platform)) {
    throw new Error(`Unsupported platform: ${platform}`)
  }
  const key = buildKey(platform, arch, isGlibc)
  const entry = PLATFORM_ENTRIES[key]
  if (entry === undefined) {
    throw new Error(`Unsupported arch: ${platform}/${arch}`)
  }
  return entry
}

export function resolvePackageName(platform: string, arch: string, isGlibc: boolean): string {
  return resolveEntry(platform, arch, isGlibc).pkg
}

export function resolveTargetTriple(platform: string, arch: string, isGlibc: boolean): string {
  return resolveEntry(platform, arch, isGlibc).triple
}
```

### `packages/cli/src/index.ts`（新規・41行）
```typescript
import { spawnSync } from 'node:child_process'
import path from 'node:path'
import { familySync } from 'detect-libc'
import { resolvePackageName, resolveTargetTriple } from './platform'

const BINARY_NAME = 'sample-cli' as const

function main(): void {
  const { platform, arch } = process
  const isGlibc = familySync() === 'glibc'

  const pkg = resolvePackageName(platform, arch, isGlibc)
  const triple = resolveTargetTriple(platform, arch, isGlibc)

  let pkgDir: string
  try {
    const pkgJsonPath = require.resolve(`${pkg}/package.json`)
    pkgDir = path.dirname(pkgJsonPath)
  } catch (err) {
    process.stderr.write(
      `Failed to resolve package ${pkg}. Is the platform package installed?\n${err}\n`,
    )
    process.exit(1)
  }

  const binaryFilename = platform === 'win32' ? `${BINARY_NAME}.exe` : BINARY_NAME
  const binaryPath = path.join(pkgDir, 'vendor', triple, BINARY_NAME, binaryFilename)

  const result = spawnSync(binaryPath, process.argv.slice(2), { stdio: 'inherit' })

  if (result.error) {
    process.stderr.write(`Failed to execute ${binaryPath}: ${result.error.message}\n`)
    process.exit(1)
  }

  process.exit(result.status ?? 1)
}

main()
```

### `packages/cli/src/platform.test.ts`（新規・84行）
```typescript
import { describe, expect, it } from 'vitest'
import { resolvePackageName, resolveTargetTriple } from './platform'

describe('resolvePackageName', () => {
  it('darwin/arm64 → @taiga-tech/cli-darwin-arm64', () => {
    expect(resolvePackageName('darwin', 'arm64', false)).toBe('@taiga-tech/cli-darwin-arm64')
  })
  it('darwin/x64 → @taiga-tech/cli-darwin-x64', () => {
    expect(resolvePackageName('darwin', 'x64', false)).toBe('@taiga-tech/cli-darwin-x64')
  })
  it('win32/x64 → @taiga-tech/cli-win32-x64', () => {
    expect(resolvePackageName('win32', 'x64', false)).toBe('@taiga-tech/cli-win32-x64')
  })
  it('linux/x64 + glibc 検出 → @taiga-tech/cli-linux-x64-gnu', () => {
    expect(resolvePackageName('linux', 'x64', true)).toBe('@taiga-tech/cli-linux-x64-gnu')
  })
  it('linux/x64 + glibc 未検出 → @taiga-tech/cli-linux-x64-musl', () => {
    expect(resolvePackageName('linux', 'x64', false)).toBe('@taiga-tech/cli-linux-x64-musl')
  })
  it('darwin では isGlibc フラグを無視して arm64 パッケージを返す', () => {
    expect(resolvePackageName('darwin', 'arm64', true)).toBe('@taiga-tech/cli-darwin-arm64')
  })
  it('未対応 platform → "Unsupported platform: ..." をスロー', () => {
    expect(() => resolvePackageName('freebsd', 'x64', false)).toThrow('Unsupported platform: freebsd')
  })
  it('未対応 arch → "Unsupported arch: ..." をスロー', () => {
    expect(() => resolvePackageName('linux', 'arm64', false)).toThrow('Unsupported arch: linux/arm64')
  })
})

describe('resolveTargetTriple', () => {
  it('darwin/arm64 → aarch64-apple-darwin', () => {
    expect(resolveTargetTriple('darwin', 'arm64', false)).toBe('aarch64-apple-darwin')
  })
  it('darwin/x64 → x86_64-apple-darwin', () => {
    expect(resolveTargetTriple('darwin', 'x64', false)).toBe('x86_64-apple-darwin')
  })
  it('win32/x64 → x86_64-pc-windows-msvc', () => {
    expect(resolveTargetTriple('win32', 'x64', false)).toBe('x86_64-pc-windows-msvc')
  })
  it('linux/x64 + glibc 検出 → x86_64-unknown-linux-gnu', () => {
    expect(resolveTargetTriple('linux', 'x64', true)).toBe('x86_64-unknown-linux-gnu')
  })
  it('linux/x64 + glibc 未検出 → x86_64-unknown-linux-musl', () => {
    expect(resolveTargetTriple('linux', 'x64', false)).toBe('x86_64-unknown-linux-musl')
  })
})
```

### `.github/workflows/release.yml`（新規・116行）
```yaml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  build:
    name: Build ${{ matrix.target }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - target: aarch64-apple-darwin
            os: macos-latest
            platform_pkg: cli-darwin-arm64
          - target: x86_64-apple-darwin
            os: macos-latest
            platform_pkg: cli-darwin-x64
          - target: x86_64-pc-windows-msvc
            os: windows-latest
            platform_pkg: cli-win32-x64
          - target: x86_64-unknown-linux-gnu
            os: ubuntu-latest
            platform_pkg: cli-linux-x64-gnu
          - target: x86_64-unknown-linux-musl
            os: ubuntu-latest
            platform_pkg: cli-linux-x64-musl
    steps:
      - uses: actions/checkout@v4
      - name: Install Rust toolchain
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}
      - name: Install musl tools (musl target only)
        if: matrix.target == 'x86_64-unknown-linux-musl'
        run: sudo apt-get update && sudo apt-get install -y musl-tools
      - name: Build binary
        run: cargo build --release --target ${{ matrix.target }}
      - name: Copy binary to platform package (Unix)
        if: runner.os != 'Windows'
        run: |
          cp target/${{ matrix.target }}/release/sample-cli \
            packages/${{ matrix.platform_pkg }}/vendor/${{ matrix.target }}/sample-cli/sample-cli
      - name: Copy binary to platform package (Windows)
        if: runner.os == 'Windows'
        run: |
          copy target\${{ matrix.target }}\release\sample-cli.exe `
            packages\${{ matrix.platform_pkg }}\vendor\${{ matrix.target }}\sample-cli\sample-cli.exe
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: binary-${{ matrix.target }}
          path: packages/${{ matrix.platform_pkg }}/vendor/${{ matrix.target }}/sample-cli/

  publish:
    name: Publish to npm
    runs-on: ubuntu-latest
    needs: [build]
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "24"
          registry-url: "https://registry.npmjs.org"
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          pattern: binary-*
          merge-multiple: false
      - name: Place binaries into vendor directories
        run: |
          cp -r binary-aarch64-apple-darwin/. packages/cli-darwin-arm64/vendor/aarch64-apple-darwin/sample-cli/
          cp -r binary-x86_64-apple-darwin/. packages/cli-darwin-x64/vendor/x86_64-apple-darwin/sample-cli/
          cp -r binary-x86_64-pc-windows-msvc/. packages/cli-win32-x64/vendor/x86_64-pc-windows-msvc/sample-cli/
          cp -r binary-x86_64-unknown-linux-gnu/. packages/cli-linux-x64-gnu/vendor/x86_64-unknown-linux-gnu/sample-cli/
          cp -r binary-x86_64-unknown-linux-musl/. packages/cli-linux-x64-musl/vendor/x86_64-unknown-linux-musl/sample-cli/
      - name: Install dependencies
        run: pnpm install
      - name: Build launcher
        run: pnpm --filter @taiga-tech/cli build
      - name: Publish platform packages
        run: |
          pnpm --filter @taiga-tech/cli-darwin-arm64 publish --no-git-checks
          pnpm --filter @taiga-tech/cli-darwin-x64 publish --no-git-checks
          pnpm --filter @taiga-tech/cli-win32-x64 publish --no-git-checks
          pnpm --filter @taiga-tech/cli-linux-x64-gnu publish --no-git-checks
          pnpm --filter @taiga-tech/cli-linux-x64-musl publish --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      - name: Publish launcher
        run: pnpm --filter @taiga-tech/cli publish --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### `turbo.json`（変更）
```diff
-"outputs": [".next/**", "!.next/cache/**"]
+"outputs": [".next/**", "!.next/cache/**", "dist/**"]

+"test": {
+  "dependsOn": ["^build"]
+}
```

### `packages/cli/package.json`（新規）
```json
{
  "name": "@taiga-tech/cli",
  "version": "0.1.0",
  "description": "CLI launcher for sample-cli",
  "bin": {
    "sample-cli": "./dist/index.cjs"
  },
  "files": ["dist/**"],
  "scripts": {
    "build": "tsup",
    "test": "vitest run"
  },
  "dependencies": {
    "detect-libc": "^2.0.4"
  },
  "optionalDependencies": {
    "@taiga-tech/cli-darwin-arm64": "0.1.0",
    "@taiga-tech/cli-darwin-x64": "0.1.0",
    "@taiga-tech/cli-win32-x64": "0.1.0",
    "@taiga-tech/cli-linux-x64-gnu": "0.1.0",
    "@taiga-tech/cli-linux-x64-musl": "0.1.0"
  },
  "devDependencies": {
    "@types/node": "^24.0.0",
    "tsup": "^8.0.0",
    "typescript": "5.9.3",
    "vitest": "^3.0.0"
  },
  "publishConfig": {
    "access": "public"
  }
}
```

### プラットフォームパッケージ `package.json`（5種・共通構造）
```json
{
  "name": "@taiga-tech/cli-darwin-arm64",
  "version": "0.1.0",
  "description": "macOS arm64 binary for @taiga-tech/cli",
  "os": ["darwin"],
  "cpu": ["arm64"],
  "files": ["vendor/**"],
  "publishConfig": {
    "access": "public"
  }
}
```

---

## CI・テスト状況

| チェック | 結果 |
|---------|------|
| GitGuardian Security Checks | ✅ success |
| ビルド CI（turbo build） | ❌ 未実行（PR時に走る CI なし） |
| テスト CI（vitest） | ❌ 未実行（PR時に走る CI なし） |

> `release.yml` はタグ `v*` push のみで発火。PR に対して自動実行されるビルド・テスト CI が存在しない。

---

## レビュアー向け注目ポイント

### 🔴 最優先（動作確認・QA）
1. **テスト計画全未チェック**: 著者によるテスト・ビルド確認が行われていない（`[ ]` のまま）
2. **PR時 CI なし**: ビルド・テストを自動検証するワークフローがない。テストが通るかどうか未確認
3. **コンパイル済みバイナリのコミット**: `target/release/sample-cli` など Rust ビルド成果物（約64ファイル）が含まれている。`.gitignore` に `target/` が未追加

### 🟡 設計・アーキテクチャ
4. **キャッシュファイルのコミット**: `.takt/.runtime/cache/pnpm/` の pnpm メタデータキャッシュ（約179ファイル）が含まれる。`.gitignore` 未対応
5. **AI 実行ログのコミット**: `.takt/runs/` の前回開発セッションのログ（約82ファイル）が含まれる
6. **`rust = "latest"` の再現性問題**: `mise.toml` でピン留めなし。バージョンを固定すべき
7. **Changeset と release.yml の二重管理**: Changeset を設定したが `release.yml` は `changeset publish` を使わず直接 `pnpm publish` している。バージョン管理フローが統一されていない