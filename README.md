# Agent Skills

> Skills used within the Philtz Claude Team, published in `.skill` and `SKILL.md` formats.

PhiltzのClaude Team内で使用されているスキルを`.skill`と`SKILL.md`形式で公開するリポジトリです。

---

## Structure

```
├── raw/                          # スキル定義のソースファイル
│   └── <skill-name>/SKILL.md
└── zip/                          # 配布用 .skill パッケージ
    └── <skill-name>.skill
```

## Skills

| Skill | Description |
|---|---|
| Active Listening | Rogers（1951）来談者中心療法的傾聴。ユーザーが安全に暗黙知を語れる対話空間を作り、無条件の肯定的配慮で語りを受け止める。 |
| API Design | OpenAPI準拠、RFC 9457エラー構造、バージョニング、パス規約、フレームワーク選定などのAPI設計標準を強制する。 |
| Code Conventions | 命名規則、フォーマット、エラーハンドリング、型の整理、アーキテクチャ制約をすべてのコード変更に強制する。 |
| Cognitive Probing DICE | Robinson（2023）DICEプロービング。記述的・固有記憶・明確化・説明的の4種類のプローブで曖昧な回答を構造的な知識に変換する。 |
| Critical Decision Method | Klein et al.（1989）重要意思決定法（CDM）。過去の具体的な出来事を4フェーズで掘り下げ、意思決定の暗黙知を引き出すインタビュー手法。 |
| Devils Advocate | MacDougall & Baum（1997）悪魔の代弁者。ユーザーの知識を鍛えるスパーリングパートナーとして、前提への挑戦やスティールマン反論を行う。 |
| Dialectical Inquiry | Hegel / Mason（1969）弁証法的探究。テーゼ→アンチテーゼ→ジンテーゼの三段階で、ユーザーの知識内の矛盾や対立を建設的に統合する。 |
| Google Analytics | Google Analyticsの同意シグナルとCookieハンドリング規則を強制する。 |
| Keep Latest Remote | ローカルブランチをリモートと最新に保つ。git操作時に自動発動。 |
| Migration Procedure | 事前件数検証とドライランによる安全なデータ移行手順を定義する。 |
| Motivational Interviewing | Miller & Rollnick（1991）動機づけ面接 OARS技法。開かれた質問・是認・反映的傾聴・要約でユーザーの内発的動機を引き出す。 |
| Narrative Elicitation | ナラティブ引き出し法・IME。物語・比喩・仮想シナリオ・対比を通じて、ユーザーが言語化しにくい暗黙知を自然に引き出す。 |
| Operational Rules | データモデル、環境変数、ツール、バージョニング、アーキテクチャドキュメントの運用手順を強制する。 |
| Package Rules | パッケージインストール方法とフレームワーク/ライブラリ選定規則を強制する。 |
| Semantic Commit | gitコミット時にセマンティックコミット規約を強制する。 |
| Socratic Questioning | Paul & Elder（2007）× Chang（2023）ソクラテス式質問法。6つの質問カテゴリと産婆術で、ユーザーの論理構造・前提・根拠を可視化し思考を精緻化する。 |
| Test Responsibility | 新機能の実装後、agent-browserまたはPlaywrightでE2Eテストを自動作成・実行する。 |
| TS Monorepo Builder | Turborepoベストプラクティスに従いTypeScriptモノレポ構造をスキャフォールド・保守する。 |

## Connect with Us

私たちの活動に関するより詳しい情報は、公式サイトをご覧ください。

- **Official Website:** https://philtz.com
