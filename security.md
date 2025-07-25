# セキュリティ概念 - 謎解き活用ガイド

## 1. 脆弱性の種類

### SQLインジェクション
- **概要**: データベースへの不正なSQLクエリの挿入
- **謎解き活用**:
  - ログインフォームに特殊文字を入力して情報取得
  - `' OR '1'='1` のような古典的パターンを使用
  - エラーメッセージから情報を推測
- **実装例**: 意図的に脆弱なフォームを作成し、正しいペイロードで次のヒントを表示

### XSS（クロスサイトスクリプティング）
- **概要**: 悪意のあるスクリプトの挿入と実行
- **謎解き活用**:
  - 反射型XSS: URLパラメータに隠しメッセージ
  - 格納型XSS: 掲示板やコメント欄にスクリプトを仕込む
  - DOM-based XSS: JavaScriptの動的処理を利用
- **実装例**: `<script>alert(atob('SGludA=='))</script>`

### CSRF（クロスサイトリクエストフォージェリ）
- **概要**: ユーザーの意図しないリクエストの送信
- **謎解き活用**:
  - 特定のリファラーからのアクセスのみ許可
  - CSRFトークンが次のヒントになる
- **実装例**: 正しいトークンを含むPOSTリクエストで情報開示

### バッファオーバーフロー
- **概要**: メモリ領域の境界を超えた書き込み
- **謎解き活用**:
  - 特定の長さの入力で隠しメッセージ表示
  - 16進数やシェルコードの理解が必要な問題
- **実装例**: 入力長が特定の値の時にフラグを表示

## 2. 認証・認可

### 多要素認証（MFA）
- **概要**: 複数の認証要素の組み合わせ
- **謎解き活用**:
  - 知識要素: パスワード（暗号解読で取得）
  - 所持要素: QRコードやトークン
  - 生体要素: 架空の指紋パターンなど
- **実装例**: 3つの異なる謎を解いて認証を通過

### JWT（JSON Web Token）
- **概要**: トークンベースの認証
- **謎解き活用**:
  - ヘッダーとペイロードの改ざん
  - 署名の検証をバイパス
  - 秘密鍵の推測
- **実装例**: `{"alg":"none"}`で署名検証を無効化

### OAuth 2.0
- **概要**: 認可のためのプロトコル
- **謎解き活用**:
  - リダイレクトURIの操作
  - stateパラメータにヒントを埋め込む
  - 認可コードの傍受
- **実装例**: 特定のscopeでのみアクセス可能な情報

### セッション管理
- **概要**: ユーザーの状態管理
- **謎解き活用**:
  - セッションIDの推測
  - セッションフィクセーション
  - Cookieの属性（HttpOnly, Secure）を利用
- **実装例**: 特定のセッションIDでログインすると特別な情報を表示

## 3. ネットワークセキュリティ

### ポートスキャン
- **概要**: 開いているポートの探索
- **謎解き活用**:
  - 非標準ポートにサービスを配置
  - ポート番号自体が暗号の一部
  - ノックシーケンス（特定の順序でアクセス）
- **実装例**: nmapで特定のポートを発見

### 中間者攻撃（MITM）
- **概要**: 通信の傍受と改ざん
- **謎解き活用**:
  - HTTPSでない通信に情報を隠す
  - 証明書のCNやSANにヒント
  - プロキシ経由でのみ見える情報
- **実装例**: Burp Suiteで傍受すると隠しヘッダーが見える

### DNSセキュリティ
- **概要**: ドメイン名システムの保護
- **謎解き活用**:
  - DNSSECレコードに情報
  - サブドメイン列挙で発見
  - TXTレコードに暗号化データ
- **実装例**: `dig ANY hidden.example.com`

### ファイアウォール/IDS回避
- **概要**: セキュリティ機器のバイパス
- **謎解き活用**:
  - フラグメンテーション
  - エンコーディングの変更
  - タイミング攻撃
- **実装例**: 特定のUser-Agentでのみ通過可能

## 4. 暗号とハッシュ

### ハッシュ関数の脆弱性
- **概要**: 衝突や弱いハッシュアルゴリズム
- **謎解き活用**:
  - MD5の衝突を利用
  - レインボーテーブルでの解読
  - ソルトなしハッシュの脆弱性
- **実装例**: 同じMD5ハッシュを持つ異なるファイル

### 暗号の実装ミス
- **概要**: 暗号アルゴリズムの誤った使用
- **謎解き活用**:
  - ECBモードの脆弱性
  - 初期化ベクトル（IV）の再利用
  - 弱い鍵の使用
- **実装例**: 同じ平文ブロックが同じ暗号文になるECBモード

### サイドチャネル攻撃
- **概要**: 実装から漏れる情報を利用
- **謎解き活用**:
  - タイミング攻撃（処理時間の差）
  - エラーメッセージの違い
  - キャッシュの挙動
- **実装例**: パスワード検証の応答時間から正解を推測

## 5. アプリケーションセキュリティ

### ディレクトリトラバーサル
- **概要**: 想定外のファイルへのアクセス
- **謎解き活用**:
  - `../../../etc/passwd`のようなパス操作
  - 隠しファイルの発見
  - 設定ファイルから情報取得
- **実装例**: 特定のパスで秘密のファイルにアクセス

### コマンドインジェクション
- **概要**: OSコマンドの不正実行
- **謎解き活用**:
  - パイプやセミコロンを使った複数コマンド実行
  - 環境変数の読み取り
  - システム情報の取得
- **実装例**: `; cat /flag.txt`で情報表示

### XXE（XML外部実体参照）
- **概要**: XMLパーサーの脆弱性
- **謎解き活用**:
  - ローカルファイルの読み取り
  - 内部ネットワークへのアクセス
  - DoS攻撃のシミュレーション
- **実装例**: DTDを使ってファイル内容を取得

### デシリアライゼーション
- **概要**: 信頼できないデータの復元
- **謎解き活用**:
  - Pickleやserializeの悪用
  - オブジェクトインジェクション
  - リモートコード実行
- **実装例**: 特殊なシリアライズデータで隠し機能を実行

## 6. 物理セキュリティ概念

### ソーシャルエンジニアリング
- **概要**: 人間の心理を利用した攻撃
- **謎解き活用**:
  - 架空の人物のSNSプロフィールから情報収集
  - フィッシングメールのシミュレーション
  - 電話番号から情報を聞き出す
- **実装例**: LinkedInプロフィールにヒント

### 物理アクセス
- **概要**: 施設やデバイスへの物理的侵入
- **謎解き活用**:
  - QRコードでドアロック解除
  - USBデバイスのシミュレーション
  - バッジのクローニング
- **実装例**: 特定の画像がアクセスカード代わり

## 7. フォレンジック

### ログ解析
- **概要**: システムログからの情報抽出
- **謎解き活用**:
  - アクセスログのパターン分析
  - エラーログから情報推測
  - タイムスタンプの異常
- **実装例**: 特定時刻のログエントリーが暗号化されたメッセージ

### メモリフォレンジック
- **概要**: メモリダンプの解析
- **謎解き活用**:
  - プロセスリストから情報取得
  - 文字列検索でヒント発見
  - 暗号鍵の抽出
- **実装例**: メモリダンプファイルに隠されたフラグ

### ファイルカービング
- **概要**: 削除されたファイルの復元
- **謎解き活用**:
  - 画像の一部から全体を復元
  - ファイルヘッダーの識別
  - スラック領域のデータ
- **実装例**: 破損したファイルを修復して情報取得

## 実装時の注意点

1. **教育的価値**: 実際のセキュリティ概念を正しく学べるように
2. **難易度調整**: 基礎知識＋調査で解けるレベルに
3. **倫理的配慮**: 実際の攻撃に使えないよう配慮
4. **ツールの使用**: 一般的なセキュリティツールの使い方を学べる
5. **段階的学習**: 簡単な概念から複雑な概念へ