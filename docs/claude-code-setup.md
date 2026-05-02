# Claude Code セットアップ手順書（Mac用）

> CAREER EDIT プロジェクトを Claude Code で続けるための環境構築手順書。  
> **対象**：Mac ユーザー（macOS 12 Monterey 以降）  
> **所要時間**：約30〜45分（Node.js が既にインストール済みなら15分）  
> **最終更新**：2026年4月27日

---

## なぜ Claude Code を使うのか

CAREER EDIT プロジェクトでは「公式HPを必ず参照する」ことが編集メディアの根本ルール。しかし、現在のチャット環境（Claude.ai）では以下の制約がある：

- robots.txt のブロックにより一部の公式HPが取れない
- ブラウザ自動操作ができない
- スクリーンショット機能がない

**Claude Code に切り替えると**：

- ✅ 公式HPを `curl` で直接取得（80%のサイトはこれで十分）
- ✅ Playwright でブラウザ自動操作（残り20%のサイトに対応）
- ✅ スクリーンショット撮影
- ✅ ローカルファイル操作（Excel生成、PDF処理等）
- ✅ GitHub への直接コミット

---

## 全体の流れ

```
1. Node.js をインストール（必須）
   ↓
2. Claude Code をインストール
   ↓
3. プロジェクトをローカルにクローン
   ↓
4. AI_HANDOFF_PROMPT.md を読ませて文脈引き継ぎ
   ↓
5. （オプション）Playwright/curl で公式HP自動取得を試す
   ↓
完了：CAREER EDIT 作業を Claude Code で継続できる状態
```

---

## ステップ1：Node.js をインストール（10分）

### 既にインストール済みか確認

ターミナルを開いて：

```bash
node --version
```

`v18.0.0` 以上が表示されれば OK。スキップしてステップ2へ。

それ以外の場合は以下のいずれかでインストール：

### 方法A：公式サイトからダウンロード（簡単）

1. https://nodejs.org/ja を開く
2. 「LTS」版のmacOS Installerをダウンロード
3. インストーラーを実行

### 方法B：Homebrew でインストール（推奨・後で更新が楽）

Homebrew が未インストールの場合、まずこれをインストール：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

その後：

```bash
brew install node
```

### 確認

```bash
node --version  # v20.x.x など
npm --version   # 10.x.x など
```

---

## ステップ2：Claude Code をインストール（5分）

### インストール

```bash
npm install -g @anthropic-ai/claude-code
```

### 確認

```bash
claude --version
```

バージョン番号が表示されればOK。

### 初回起動と認証

```bash
claude
```

ブラウザが開いて Anthropic アカウントへのログインを求められる。Claude Pro / Team / Max のアカウントでログイン（既にClaude.aiで使っているアカウントと同じ）。

API キーは不要（Claude Pro等のサブスクリプションがあれば認証で済む）。

---

## ステップ3：プロジェクトをローカルにクローン（5分）

### Git が入っているか確認

```bash
git --version
```

入っていなければ Mac に Xcode Command Line Tools が必要：

```bash
xcode-select --install
```

### CAREER EDIT リポジトリをクローン

```bash
# 適当な作業フォルダに移動
cd ~/Documents

# CAREER EDIT リポジトリをクローン
git clone https://github.com/marumoto1984/career-edit.git

# プロジェクトフォルダに入る
cd career-edit
```

### 確認

```bash
ls -la
```

以下のような構造が見えればOK：

```
career-edit/
├── README.md
├── docs/
│   ├── AI_HANDOFF_PROMPT.md
│   ├── PROJECT_OVERVIEW.md
│   ├── editorial-policy.md
│   ├── company-profile-process.md
│   ├── research-task-manual.md
│   └── ...
├── index.html
└── logo/
```

---

## ステップ4：Claude Code を起動して文脈を引き継ぐ（5分）

### Claude Code を起動

プロジェクトフォルダ内で：

```bash
claude
```

対話モードが起動する。

### 初回プロンプト（コピペで使える）

以下をそのままコピペ：

```
このプロジェクトは「CAREER EDIT」という広告・マーケ・IT領域のキャリアメディアの立ち上げです。
編集長：丸本翔一（マルサン代表）。

まず、以下のドキュメントを順に読み込んで、プロジェクトの文脈を把握してください：

1. docs/AI_HANDOFF_PROMPT.md（最重要・これだけで全体方針がわかる）
2. docs/PROJECT_OVERVIEW.md（プロジェクト全体像）
3. docs/editorial-policy.md（編集姿勢8原則）
4. docs/company-profile-process.md（企業情報作成プロセス・公式HP参照ルール含む）
5. docs/content-taxonomy.md（業態分類）

読み終えたら、以下を確認してから「読みました」と答えてください：

- 編集姿勢8原則のうち、原則1と原則8の核心は何か
- 「公式HP参照ルール」に違反する例は何か
- 業態分類の8カテゴリで、CAREER EDITが新設したカテゴリは何か

これから、index.html に統合されている6社プロト（電通・博報堂・ADK・東急エージェンシー・電通デジタル・オプト）に続いて、残り4社（サイバーエージェント・セプテーニ・GO・TUGBOAT）のプロトを作成していきます。
```

Claude Code が docs/ 配下を読み込み、プロジェクトの文脈を把握する。

---

## ステップ5（オプション）：公式HP自動取得を試す（10分）

### 第1段階：curl のみで試す

最もトークン消費が少ない方法。Mac には curl が標準装備されているので追加インストール不要。

Claude Code への指示例：

```
東急エージェンシーの公式HPから会社概要を取得してください。

手順：
1. curl で https://www.tokyu-agc.co.jp/company/about.html にアクセス
2. HTMLから会社概要の情報（企業名、設立、代表者、本社住所、資本金、従業員数、売上）を抽出
3. 結果を構造化して表示

User-Agent には適切なブラウザ識別子を使ってください：
curl -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36" [URL]
```

これで公式HPの内容が取れれば、第1段階で完結。

### 第2段階：Playwright CLI を試す（curl で取れない場合のみ）

Playwright CLI のインストール：

```bash
# CAREER EDITプロジェクトフォルダ内で
npm init -y
npm install -D @playwright/test
npx playwright install chromium
```

Claude Code への指示例：

```
東急エージェンシーの公式HPは curl では取れませんでした。
Playwright を使って取得してください。

手順：
1. Node.js スクリプトを書く
2. Playwright で https://www.tokyu-agc.co.jp/company/about.html にアクセス
3. ページの本文テキストを抽出
4. 必要な情報を構造化して表示

スクリプトは tools/fetch-company.js として保存してください。
```

### Playwright のトークン消費を抑える設定

Playwright MCP を使うとトークン消費が激しいので、**Playwright CLI**（コマンドラインで直接実行）を推奨。MCPの約1/4のトークンで済む。

---

## ステップ6：実際の作業フロー

### 例：サイバーエージェントのプロトを作成する

```
サイバーエージェントのプロトを作成してください。

【手順】
1. 公式HPから情報を取得：
   - https://www.cyberagent.co.jp/about/profile/
   - https://www.cyberagent.co.jp/about/officer/
   - https://www.cyberagent.co.jp/ir/
   curl で取れない場合は Playwright を使う

2. 取得した情報を docs/research/cyberagent.md にメモとして保存

3. index.html 内の電通プロト（id="page-company-dentsu"）の構造を読み取り、それをマスターとして使う

4. サイバーエージェント用のプロトを作成（11カテゴリ・85項目構造）
   - 情報がない項目は隠す
   - 出典の3階層ルールを守る
   - 編集姿勢8原則を守る

5. 完成したプロトを index.html に統合（電通デジタルプロトの直後に挿入）

6. 左ナビゲーションにも追加

7. 検証：HTMLの構造が壊れていないかチェック

8. 終わったら git add . && git commit -m "Add CyberAgent prototype" で記録
```

Claude Code はこれを自律的に実行してくれる。

---

## トラブルシューティング

### Q. `node: command not found` エラー

A. Node.js がインストールされていない、またはパスが通っていない。ステップ1からやり直す。

### Q. `claude: command not found` エラー

A. Claude Code のインストールが完了していない。

```bash
npm install -g @anthropic-ai/claude-code
```

を再実行。それでもダメなら、npm のグローバルパスを確認：

```bash
npm config get prefix
```

このパスの `bin/` がPATHに含まれているか確認。

### Q. Claude Code 起動時に認証エラー

A. ブラウザが自動で開かない場合、表示されるURLをコピーして手動でブラウザに貼り付けてログイン。

### Q. curl で robots.txt エラー

A. 一部のサイトは User-Agent をチェックしている。

```bash
curl -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" https://...
```

これでも取れない場合は Playwright を使う。

### Q. Playwright インストールでエラー

A. Mac の場合、Apple Silicon（M1/M2/M3）でアーキテクチャ問題が起きることがある。

```bash
# Rosetta 2 を経由する場合
arch -x86_64 npx playwright install chromium
```

### Q. トークン消費が予想以上に多い

A. 以下を確認：

1. Playwright MCP ではなく **Playwright CLI** を使っているか（CLIの方が約1/4）
2. 不要なファイルを Claude に読ませていないか（プロジェクトルートに大きなログがあると全部読まれる）
3. 一度に複数社を処理せず、1社ずつ完結させる

### Q. GitHub にプッシュできない

A. SSH キーが設定されていない可能性。

```bash
# SSH キー生成（既にあればスキップ）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 公開キーを表示
cat ~/.ssh/id_ed25519.pub

# これを GitHub の Settings > SSH and GPG keys に登録
```

---

## トークン消費の目安（参考）

| 作業 | 推定トークン消費（往復合計） |
|---|---|
| 公式HPをcurlで取得＋情報抽出（1社） | 5,000〜15,000 |
| Playwright CLIで取得（1社） | 10,000〜30,000 |
| 1社のプロト作成（公式HP取得込み） | 30,000〜80,000 |
| index.htmlへの統合・検証 | 10,000〜30,000 |
| 全体（残り4社プロト作成） | 200,000〜500,000 |

Claude Pro（月$20）の使用枠で十分カバーできる範囲。

---

## 推奨される作業の進め方

### パターン1：1社ずつ丁寧に

```
Day 1: サイバーエージェントのプロト作成
Day 2: セプテーニのプロト作成
Day 3: GOのプロト作成
Day 4: TUGBOATのプロト作成
Day 5: 全体のレビュー＋デザイン調整
```

各日のClaude Code利用時間：1〜2時間程度。

### パターン2：一気に進める（半日コース）

```
午前：4社の公式HP情報を一括取得（curl + Playwright）
   ↓
午後：4社のプロトを順番に生成
   ↓
夕方：index.htmlに統合、検証、コミット
```

トークン消費は多いが、流れが途切れない利点がある。

### パターン3：バイトとの分業（推奨）

```
バイトのフェーズ（research-task-manual.md に従って）：
- 4社の公式HPスクショ取得
- データシート（Excel）に転記
- Googleドライブで共有

   ↓ 編集長レビュー

Claude Code のフェーズ：
- スクショ＋データシートを読み込む
- プロトを生成
- index.htmlに統合
```

---

## 補足：Claude Code でできる Claude.ai との違い

| 項目 | Claude.ai（このチャット） | Claude Code |
|---|---|---|
| ローカルファイル読み書き | ❌ | ✅ |
| `curl` 等のコマンド実行 | ❌ | ✅ |
| Git 操作 | ❌ | ✅ GitHubに直接プッシュ |
| ブラウザ自動操作 | ❌ | ✅ Playwright |
| 長時間の自律作業 | △ | ✅ 安定 |
| プロジェクト全体の文脈把握 | △ コンパクションあり | ✅ ファイル直接参照 |
| マルチファイル編集 | ❌ | ✅ |
| **編集視点・対話** | ✅ Claude.ai が自然 | △ コーディング寄り |

**結論**：作業の大部分（公式HP取得、プロト生成、index.html統合、Git操作）は Claude Code が圧倒的に効率的。一方、**編集方針の議論・キャリア相談・編集視点の壁打ち**は Claude.ai の方が向いている。

両方を使い分けるのがベスト。

---

## 改訂履歴

- 2026.04.27：初版作成
