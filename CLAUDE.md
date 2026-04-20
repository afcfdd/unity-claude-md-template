# CLAUDE.md

このファイルは、この Unity プロジェクトで作業する AI コーディングエージェント向けの実務ルールを定義する。
目的は「VContainer + UniTask + R3 前提で、長期運用しても崩れにくい構造を守ること」。

## 1. プロジェクトの基本方針

- このプロジェクトでは、責務の分離・依存方向・ライフタイムの明示を最優先にする。
- まず「どの層の責務か」を判断してから実装する。
- 迷ったら、薄い MonoBehaviour / 純 C# / 明示的 DI / 小さい変更 を優先する。
- 大規模な構造変更より、既存構造に沿った局所的で安全な変更を優先する。
- hidden global state, service locator, god object を増やさない。

## 2. 技術スタックの役割分担

- VContainer:
  - 依存解決、ライフタイム管理、composition root を担当する。
  - `LifetimeScope` / installer / bootstrap 以外で container に触れない。
  - `IObjectResolver` を業務ロジックに持ち込まない。
- UniTask:
  - one-shot async を担当する。
  - 例: 初期化、ロード、保存、通信、シーン遷移、待機。
- R3:
  - 状態変化・イベントストリーム・UI 反映を担当する。
  - 例: 画面状態、入力ストリーム、モデルの変化通知、一定時間ごとの更新。
- MonoBehaviour:
  - View / Adapter として使う。
  - Unity イベント、SerializeField、見た目、GameObject 参照に責務を限定する。

## 3. 推奨アーキテクチャ

基本は **feature-first + layer-within-feature** を採用する。

推奨ディレクトリ例:

```text
Assets/App/
  Bootstrap/
    ProjectLifetimeScope.cs
    SceneLifetimeScope/
    Installers/
  Shared/
    Domain/
    Application/
    Infrastructure/
    Presentation/
  Features/
    Player/
      Domain/
      Application/
      Infrastructure/
      Presentation/
      PlayerLifetimeScope.cs
    Battle/
      Domain/
      Application/
      Infrastructure/
      Presentation/
      BattleLifetimeScope.cs
  Scenes/
  Prefabs/
  ScriptableObjects/
  Tests/
    EditMode/
    PlayMode/