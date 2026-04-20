# Unity 向け CLAUDE.md テンプレート

VContainer + UniTask + R3 を技術スタックとする Unity プロジェクトで、Claude Code（AI コーディングエージェント）に一貫した設計ルールを守らせるための `CLAUDE.md` 雛形です。

## このリポジトリについて

`CLAUDE.md` はプロジェクトルートに置くだけで、Claude Code がそのプロジェクト内で作業する際に自動的に読み込みます。  
このファイルに設計方針・技術スタックの役割分担・アーキテクチャ規約を書いておくことで、AIへの指示を毎回書かずに済みます。

## 対象スタック

| ライブラリ | 役割 |
|-----------|------|
| [VContainer](https://github.com/hadashiA/VContainer) | DI・ライフタイム管理・Composition Root |
| [UniTask](https://github.com/Cysharp/UniTask) | one-shot 非同期処理 |
| [R3](https://github.com/Cysharp/R3) | 状態変化・イベントストリーム・UI 反映 |

## 使い方

1. このリポジトリの `CLAUDE.md` を自分のプロジェクトルートにコピーする
2. プロジェクト固有の方針があれば追記・修正する
3. Claude Code で作業を開始するだけ

```
your-unity-project/
  CLAUDE.md   ← ここに置く
  Assets/
  Packages/
  ...
```

## CLAUDE.md の内容

### 1. プロジェクトの基本方針

- 責務の分離・依存方向・ライフタイムの明示を最優先
- まず「どの層の責務か」を判断してから実装
- 迷ったら「薄い MonoBehaviour / 純 C# / 明示的 DI / 小さい変更」を優先
- hidden global state / service locator / god object を増やさない

### 2. 推奨ディレクトリ構成

feature-first + layer-within-feature を採用。

```
Assets/App/
  Bootstrap/
  Shared/
    Domain/ Application/ Infrastructure/ Presentation/
  Features/
    Player/
      Domain/ Application/ Infrastructure/ Presentation/
      PlayerLifetimeScope.cs
    Battle/
      ...
  Tests/
    EditMode/ PlayMode/
```

## カスタマイズ例

プロジェクト固有のルールを末尾に追加するのが推奨です。

```markdown
## 4. このプロジェクト固有のルール

- セーブデータの読み書きは SaveRepository 経由に統一する。
- Scene 遷移は SceneTransitionService 以外から行わない。
```

## ライセンス

MIT
