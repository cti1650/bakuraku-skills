# bakuraku-skills

バクラク（https://workflow.layerx.jp ）の申請作業を支援するスキル集。

> [!NOTE]
> **個人が検証目的で作成しているリポジトリ。** 特定の組織の公式ツールではなく、
> 作者の所属先の見解を代表するものでもない。無保証で提供する。
> 業務で使う場合は、各自の組織のルールに従って判断すること。

> [!IMPORTANT]
> **組織固有の情報はこのリポジトリに含めない。**
> ベンダー名・サービス名・適格請求書番号・カード利用名・稟議名・内訳は、
> `plugins/bakuraku-skills/skills/bakuraku-apply/references/vendors.local.md` に置く（`.gitignore` 済み）。
>
> リポジトリ内のファイルは判定の考え方と画面操作手順のみを持つ。

## 収録スキル

| スキル | 内容 |
|---|---|
| `bakuraku-apply` | 請求書PDFからカード利用報告の下書きを作成する。ベンダー判定表と画面操作手順を提供する |

## 何のためのものか

バクラクの申請をエージェントに任せる際、手順が無いと毎回

- 画面のどこを押せばよいか、スナップショットを繰り返し取って探す
- 自動入力やファイル添付の挙動を知らずに二重入力してしまう

ことになり、トークンを浪費し事故も起きる。あらかじめ「どこを押すか」「何に気をつけるか」を与えることで、この往復を減らす。

ベンダー判定の実データは `vendors.local.md` に置く。これを用意すると、ベンダー特定にかかる往復も削減できる。

## 使うには

`vendors.local.md` の準備を推奨する。無くても動作するが、その場合はベンダーごとに利用者への確認が発生する。書式は `references/vendors.md` を参照。

## 導入

### Claude Code

```
/plugin marketplace add <このリポジトリのURL>
/plugin install bakuraku-skills@bakuraku-skills
```

導入後、「この請求書をバクラクに申請して」等で自動的に呼び出される。

ブラウザ操作用の Playwright MCP はプラグインに同梱している（`plugins/bakuraku-skills/.mcp.json`）。
別途インストールする必要はなく、プラグインを有効にすると `npx` 経由で起動する。

**初回だけ、立ち上がったブラウザで利用者自身がバクラクにログインする。**
プロファイルは `${CLAUDE_PLUGIN_DATA}/chrome-profile` に永続化されるので、以降のセッションではログイン状態が引き継がれる。
エージェントは認証情報を受け取らない。

### Codex

リポジトリをクローンし、作業ディレクトリとして開く。ルートの `AGENTS.md` が参照先を案内する。

```
git clone <このリポジトリのURL>
cd bakuraku-skills
codex
```

Codex はプラグインの `.mcp.json` を読まないため、Playwright MCP は `~/.codex/config.toml` に自分で登録する。

```toml
[mcp_servers.playwright]
command = "npx"
args = ["-y", "@playwright/mcp@latest", "--user-data-dir", "/absolute/path/to/chrome-profile"]
```

`--user-data-dir` を省略すると毎回一時プロファイルになり、そのつどログインし直しになる。

## 前提

- **ログインは利用者自身が行う。** このスキルは認証情報を扱わない（[バクラク共通利用規約](https://bakuraku.jp/terms/common/) 第12条第4項・第17条第10号）
- **下書き保存までで止まる。** 申請の提出は人が内容を確認してから行う
- ブラウザ操作には Playwright MCP を使う。Claude Code では同梱済み、Codex では利用者が登録する
- `npx` が動く Node.js 環境が必要

## 構成

```
.
├── .claude-plugin/marketplace.json     マーケットプレイス定義
├── AGENTS.md                           Codex 用の入口
└── plugins/bakuraku-skills/
    ├── .claude-plugin/plugin.json      プラグイン定義
    ├── .mcp.json                       同梱する MCP サーバ（Playwright）
    └── skills/bakuraku-apply/
        ├── SKILL.md                    手順と制約
        └── references/
            ├── vendors.md              ベンダー対応表
            └── ui-flow.md              画面操作手順
```

## 更新

対応表や画面手順を変えたら `plugins/bakuraku-skills/.claude-plugin/plugin.json` の `version` を上げる。

## ライセンス

MIT License. 詳細は [LICENSE](LICENSE) を参照。
