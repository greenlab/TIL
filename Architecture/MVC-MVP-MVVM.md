**MVC・MVP・MVVM 比較まとめ**（これまでのやり取りを反映）

壁打ち相手： Grok  
作成日： 2026/08/02  

---

これまでの議論を踏まえ、3つのパターンを整理して比較します。

---

### 1. 基本概要

| パターン | 構成要素 | 中核的な考え方 |
|----------|----------|----------------|
| **MVC** | Model - View - Controller | Controllerが入力を受け、Modelを操作し、Viewを更新する |
| **MVP** | Model - View - Presenter | PresenterがViewとModelを仲介。Viewは受動的 |
| **MVVM** | Model - View - ViewModel | ViewModelが画面用の状態を持ち、データバインディングでViewと自動同期 |

---

### 2. Viewに対するアプローチの違い

| 項目 | MVC | MVP | MVVM |
|------|-----|-----|------|
| **更新の仕方** | ControllerがViewを更新（明示的） | PresenterがViewのメソッドを呼び出して更新（明示的・命令的） | データバインディングにより自動同期（宣言的） |
| **主導権** | Controller | Presenter（Push型） | ViewModelの状態をViewが監視（Observe型） |
| **Viewとの関係** | ControllerがViewを知っていることが多い | PresenterがViewのインターフェースを保持 | ViewModelはViewを**知らない** |
| **ユーザー操作** | Controllerのアクション/メソッド | Viewのイベント → Presenterのメソッド | 主にCommand（ICommandなど） |

**これまでの議論でのポイント**
- MVPはイベントドリブンで、Presenterが「こう表示しろ」と指示する
- MVVMはデータバインディングで先に紐づけ、状態が変われば自動反映される
- MVVMの内部もイベント（PropertyChangedなど）を使っているが、開発者が書くコードのスタイルは宣言的

---

### 3. Model と ViewModel / Presenter の役割

| 役割 | Model | ViewModel（MVVM） / Presenter（MVP） |
|------|-------|--------------------------------------|
| **本質** | ビジネスデータとルール | 画面のための仲介・状態管理 |
| **UIへの依存** | なし | 画面の要求に応える（ただし具体的なViewは知らない） |
| **例** | `User`（氏名・生年月日など） | `FullName`や`AgeText`、`IsWarning`など画面用に整形したプロパティ |

**MVVMの重要なポイント（議論より）**
- ViewModelはViewの**実装を知らない**が、Viewの**要求には応える**（必要なプロパティやコマンドを用意する）
- Viewは極力ロジックを持たず、バインディングとスタイル・トリガーで記述するのが理想

---

### 4. 見た目の制御例（「注意」を赤＋太字で表示）

| パターン | 誰が判断・指示するか | 実際の処理 |
|----------|----------------------|------------|
| **MVP** | Presenterが指示 | Presenterが`SetColor(Red)`や`ShowAsWarning()`を呼ぶ |
| **MVVM** | ViewModelが状態を出す | ViewModelは`IsWarning = true`だけを公開。<br>赤・太字はView側のDataTriggerやStyleで処理 |

→ MVVMでは「状態（意味）」はViewModel、「見た目の適用」はView、と分離するのが基本。

---

### 5. 複雑なビジネスロジックが発生した場合

どのパターンでも**プレゼンテーション層（Controller / Presenter / ViewModel）に直接書かない**のが共通方針です。

```
[View]
   ↕
[Controller / Presenter / ViewModel]   ← 画面制御のみ（薄く保つ）
   ↓
[UseCase / Service / Interactor]       ← 複雑なビジネスロジックをここに抽出
   ↓
[Model / Domain]
   ↓
[Repository]
```

| パターン | プレゼンテーション層を薄くする方法 |
|----------|------------------------------------|
| MVC | Controller → Service / UseCase に委譲 |
| MVP | Presenter → Interactor / UseCase に委譲 |
| MVVM | ViewModel → UseCase / Service に委譲 |

---

### 6. 総合比較表

| 比較軸 | MVC | MVP | MVVM |
|--------|-----|-----|------|
| **テスタビリティ** | 普通 | 高い（Presenterを単体テストしやすい） | 高い（ViewModelを単体テストしやすい） |
| **Viewの薄さ** | 中程度 | 非常に薄い（Passive View） | 非常に薄い（ほぼバインディングのみ） |
| **更新スタイル** | 明示的 | 明示的（命令的） | 自動（宣言的） |
| **データバインディング** | 必須ではない | ほぼ使わない | **前提** |
| **向いている環境** | 伝統的Web（Rails, Django, ASP.NET MVCなど） | データバインディングが弱いUI（旧Androidなど） | データバインディングが強いUI（WPF, UWP, Android DataBinding, SwiftUI, MAUIなど） |
| **肥大化しやすい部分** | Controller | Presenter | ViewModel |
| **複雑なロジックの置き場所** | Service / Domain | Interactor / UseCase | UseCase / Service |

---

### 7. 選択の目安（まとめ）

- **シンプルさ・フレームワーク標準を優先** → MVC
- **テストしやすさ重視・データバインディングが弱い** → MVP
- **データバインディングが使えるモダンなUI** → MVVM
- **大規模・長期保守** → 上記のいずれか + UseCase/Service + Domain層（Clean Architecture的な構成）

これまでの議論で特に強調された点は次の通りです。

1. MVVMの核心は「データバインディングによる自動同期」と「ViewModelがViewを知らない」こと
2. それでもViewModelは画面の要求に合わせてプロパティを用意する
3. Viewはロジックを極力持たない
4. どのパターンでも複雑ロジックはプレゼンテーション層の外に出す

この理解で、3パターンの違いと使い分けが整理できていると思います。必要であれば特定の部分をさらに深掘りします。