# コード提示前の必須チェック項目

コードを提示する前に、以下の項目を必ず自己チェックし、エラーがないことを確認してください。

## Swift 6 / Strict Concurrency Checks

1. **actor境界の確認**
   - actorの外から`@MainActor`や他のactor-isolatedなinitやメソッドを呼んでいないか？
   - `Sendable`な構造体のinitは`nonisolated`にすべきではないか？
   - `await`が必要な箇所で抜けていないか？逆に不要な`await`を付けていないか？

2. **MainActor隔離の確認**
   - UI関連でない構造体（DTO、Snapshot等）に`@MainActor`が付いていないか？
   - `@MainActor`なinitを、actor内やバックグラウンドスレッドから呼んでいないか？

3. **Sendableの確認**
   - actor境界を越えて渡す型（DTO、イベント、状態等）に`Sendable`が付いているか？
   - `Sendable`な型のプロパティも全て`Sendable`か？

4. **actor init中の制約**
   - actorのinit内で、actor-isolatedなメソッドを呼んでいないか？
   - stored propertiesを全て初期化してから、selfを使っているか？

## SwiftUI / ObservableObject

5. **ViewModel初期化**
   - `@StateObject`の初期化は`_vm = StateObject(wrappedValue: ...)`の形式か？
   - init内で全stored propertiesを初期化してから、selfを使った購読処理を行っているか？

6. **二重状態管理の回避**
   - 同じインスタンスを`@StateObject`と`@ObservedObject`で重複管理していないか？
   - 所有元は`@StateObject`、子Viewは`@ObservedObject`で参照のみか？

7. **UI更新のスレッド**
   - `@Published`の更新は`@MainActor`コンテキストか、`receive(on: DispatchQueue.main)`を通しているか？

## SwiftUI構文

8. **文字列補間のエスケープ**
   - `Text("\(variable)")`が`Text("\\(variable)")`になっていないか？
   - バックスラッシュのエスケープミスがないか？

9. **ScrollViewReaderのジェネリクス**
   - `ScrollViewReader { proxy in ... }`のようにクロージャ引数を受けているか？
   - 型引数`<...>`を不要に指定していないか？

10. **onScrollGeometryChangeの引数**
    - iOS 18の新APIを使う場合、`for:`と`action:`の両方を指定しているか？

## Combine

11. **AnyCancellableの管理**
    - 全ての購読を`Set<AnyCancellable>`に`store(in: &cancellables)`しているか？
    - 一時変数に格納して即座に解放されていないか？

12. **スレッド切り替え**
    - UI更新前に`receive(on: DispatchQueue.main)`を入れているか？

## Watch Connectivity

13. **replyHandlerの実装**
    - `sendMessage`で`replyHandler`を使う場合、受信側で`didReceiveMessage:replyHandler:`を実装しているか？

## コード品質

14. **バージョン表記**
    - コード全体を提示する場合、冒頭に`// バージョン: vX.X`を付けているか？

15. **変更点の明示**
    - 修正・追加箇所に`// 🔧`や`// 🆕`などのコメントを付けているか？
    - 変更点を箇条書きで明示しているか？

16. **コメントの適切性**
    - 難易度に応じて、理解を助けるコメントを付けているか？
    - 過度なコメントで可読性を下げていないか？

17. **実践的なコード**
    - 学習用と明示されていない限り、実践的で本番投入可能なコードを提示しているか？
    - エラーハンドリング、エッジケース、キャンセル処理を考慮しているか？

## 提示前の最終確認

上記の全項目をチェックし、問題がある場合は修正してから提示してください。
特に以下の頻出エラーに注意：
- `Main actor-isolated initializer cannot be called from outside of the actor`
- `Reference to generic type 'ScrollViewReader' requires arguments in <...>`
- `Missing arguments for parameters 'for', 'action' in call`
- `'self' used before all stored properties are initialized`
- actor init中のactor-isolatedメソッド呼び出し

これらのエラーが出ないよう、事前に設計を見直してください。

---

# コード提示プロセス（必須）

コードを提示する際は、以下の3ステップを必ず実行してください：

## Step 1: 設計レビュー（内部思考）
- actor境界、MainActor隔離、Sendable制約を確認
- 競合条件、デッドロック、メモリリークの可能性を検討
- SwiftUI/Combineのアンチパターンがないか確認

## Step 2: コード生成
- 上記の自己チェック項目を全て満たすコードを生成
- バージョン番号と変更点コメントを付与

## Step 3: 提示前の最終検証
以下の質問に全て「はい」と答えられるか確認：
1. このコードはSwift 6 strict concurrency checksをパスするか？
2. actor境界を越えた不正な呼び出しはないか？
3. UI更新は全てMainActorで行われるか？
4. 二重状態管理やメモリリークの可能性はないか？
5. エラーハンドリングとキャンセル処理は適切か？
6. ユーザーがコピー&ペーストで即座に使えるか？

全て「はい」の場合のみ、コードを提示してください。
「いいえ」がある場合は、Step 1に戻って設計を見直してください。

## 提示形式
1. バージョン番号（例：v1.0）
2. 目的・変更点の要約
3. コード全体（変更箇所にコメント）
4. 変更点の明示（箇条書き）
5. 注意点・よくある落とし穴

---

# 過去の成功パターン

## EngineSnapshot（Sendable構造体）
- `nonisolated init`を使用
- `@MainActor`を付けない
- 全プロパティが`Sendable`
- デフォルト引数に`Date()`を使用可能（nonisolatedコンテキスト）

## TimerEngine（actor）
- init中にactor-isolatedメソッドを呼ばない
- AsyncStreamでイベントを配信
- 時刻ベースの堅牢な計算（バックグラウンド/一時停止対応）
- 排他制御の中枢化（ExclusiveCoordinator）

## ViewModel（@MainActor ObservableObject）
- 全stored propertiesを初期化してからselfを使用
- AsyncStreamをCombineにブリッジ（toPublisher拡張）
- `receive(on: DispatchQueue.main)`でUI更新を保証

## View（SwiftUI）
- `@StateObject`で所有、`@ObservedObject`で参照
- `_vm = StateObject(wrappedValue: ...)`の形式で初期化
- `Text("\(variable)")`のバックスラッシュエスケープに注意

---

# 頻出エラーと解決策

## `Main actor-isolated initializer cannot be called from outside of the actor`
**原因**: `Sendable`な構造体のinitが暗黙的に`@MainActor`に隔離されている
**解決**: `nonisolated init`を明示

## `Reference to generic type 'ScrollViewReader' requires arguments in <...>`
**原因**: 型引数を不要に指定している
**解決**: `ScrollViewReader { proxy in ... }`のようにクロージャで使用

## `'self' used before all stored properties are initialized`
**原因**: init内でstored properties初期化前にselfを使用
**解決**: 全プロパティ初期化後にselfを使った処理を行う

## actor init中のactor-isolatedメソッド呼び出し
**原因**: Swift 6ではactor init中はselfがまだactor隔離されていない
**解決**: setupメソッドを分離し、init後に外部から呼ぶ

## エラー/よくある落とし穴対策
- ViewModel初期化の順序に注意（self初期化前のメソッド呼び出し禁止）
- Swift 6: actor init中にactor-isolatedメソッド呼び出し禁止
- .onScrollGeometryChange の Missing arguments エラー回避（適切な引数を指定）
- Text("(Int(minutes))分") のエスケープミス回避
- sendMessageのreplyHandlerを使う場合、受信側で didReceiveMessage:replyHandler: を必ず実装
- ObservableObjectのアンチパターン回避（@StateObject/@ObservedObjectの混在禁止、過度な階層化NG）
- Invalid redeclaration of synthesized propertyエラーがViewModelで発生する場合はAppStrageからUserDefaultへ変更
- Type 'ShapeStyle' has no member 'accentColor' ''SwiftUIの .accentColor は iOS 15以降非推奨で、代わりに .tint(_:) もしくは Color.accentColor を明示的に色として使う必要があります。''
- Initializer 'init(wrappedValue:)' is not available due to missing import of defining module 'Combine'がViewModelで良く発生する
