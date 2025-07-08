# 謎解きツール・リソース

## 1. ネットワークリソース

### IPアドレス
- **活用方法**: 特定のIPアドレスにアクセスすると次のヒントが表示される
- **実装例**: 
  - IPv4: `10.47.7.13` (Project Aegisの内部ネットワーク)
  - IPv6に暗号を埋め込む: `2001:db8:85a3::8a2e:370:7334`
  - 各オクテットがASCIIコード: `65.76.69.88` → "ALEX"
- **謎解きのヒント**:
  ```bash
  # IPから情報を取得
  curl http://10.47.7.13:31337/hint
  nmap -sV -p- 10.47.7.13
  ```

### ドメイン名
- **活用方法**: 
  - サブドメインに情報を隠す
  - DNS TXTレコードにメッセージを設定
  - WHOISデータに架空の情報を含める
- **実装例**: 
  ```bash
  # TXTレコードに暗号を設定
  dig TXT protocol-seven.projectaegis.com
  # "v=spf1 include:_spf.google.com ~all" 
  # "hint=VGhlIHRydXRoIGlzIGRpc3RyaWJ1dGVk"
  
  # サブドメインパターン
  shadow.projectaegis.com
  collective.projectaegis.com
  seven.projectaegis.com  # 7番目が重要
  ```

### ポート番号
- **活用方法**: 
  - 特定のポートにのみ応答するサービス
  - ポート番号自体が暗号の一部
- **実装例**: 31337（エリート）、8080、443などの意味のある番号

### パケットキャプチャ
- **活用方法**: 
  - 特定のパケットキャプチャ（PCAP）ファイルに隠された情報
  - Wiresharkで特定のフィルタを使用してメッセージを抽出

## 2. 通信プロトコル

### HTTPヘッダー
- **活用方法**:
  - カスタムヘッダーに情報を埋め込む
  - User-Agentに特定の文字列が必要
  - Cookieに暗号化されたデータ
- **実装例**: 
  ```bash
  # 基本的な例
  curl -H "X-Project-Aegis: true" \
       -H "X-Shadow-Collective: false" \
       -H "X-Protocol: Seven" \
       https://api.example.com/secret
  
  # User-Agentで認証
  curl -A "Mozilla/5.0 (Aegis; Guardian)" https://example.com
  
  # Cookieに暗号
  curl -b "session=YWxleGNoZW46cHJvdG9jb2xzZXZlbg==" \
       https://example.com/dashboard
  ```
- **レスポンスヘッダーの確認**:
  ```bash
  curl -I https://example.com | grep "X-"
  # X-Hint: Check commit a7b3c9d
  # X-Next-Step: /api/v2/protocol
  ```

### WebSocket
- **活用方法**: リアルタイムで変化するメッセージ
- **実装例**: 特定の時刻にのみ表示される情報

### API エンドポイント
- **活用方法**:
  - RESTful APIの特定のエンドポイントに情報を隠す
  - 認証トークンが必要なAPI
- **実装例**: `/api/v1/secret/{暗号化されたID}`

## 3. ファイル形式

### 画像ファイル
- **活用方法**:
  - ステガノグラフィー（データの隠蔽）
  - EXIFデータに座標や時刻情報
  - 画像内のQRコード
- **実装ツール**: steghide, exiftool, zsteg

### 音声ファイル
- **活用方法**:
  - スペクトログラムで文字を表示
  - 逆再生で聞こえるメッセージ
  - 特定の周波数に情報を埋め込む
- **実装ツール**: Audacity, sox, ffmpeg

### 動画ファイル
- **活用方法**:
  - 特定のフレームに情報を隠す
  - 字幕トラックに暗号
- **実装ツール**: ffmpeg, VLC

### PDFファイル
- **活用方法**:
  - 非表示レイヤー
  - メタデータ
  - JavaScriptの埋め込み
- **実装ツール**: pdfinfo, pdfcrack

## 4. QRコード・バーコード

### QRコード
- **活用方法**:
  - 多層QRコード（QRコード内にQRコード）
  - エラー訂正レベルを利用したデータ隠蔽
  - 動的QRコード（時間で変化）
- **実装ツール**: 
  ```bash
  # QRコード生成
  qrencode -o hint.png "Protocol Seven Activated"
  
  # QRコード読み取り
  zbarimg secret_qr.png
  # QR-Code:https://github.com/alexchen/keys
  
  # 多層QRコードの作成
  # 1. 最終メッセージをQR化
  qrencode -o level2.png "The truth is distributed"
  # 2. level2.pngのURLをQR化
  qrencode -o level1.png "https://example.com/level2.png"
  ```
- **エラー訂正レベルの活用**:
  ```python
  # エラー訂正レベルH(30%)でも読めるように
  # QRコードの一部を意図的に破損させて
  # 隠しメッセージを埋め込む
  ```

### バーコード
- **活用方法**:
  - 商品コードに見せかけた暗号
  - 複数のバーコードを組み合わせる

## 5. ソーシャルメディア・Web

### GitHubリポジトリ
- **活用方法**:
  - コミットメッセージ
  - ブランチ名
  - Issue番号
  - Gistに隠されたコード
- **実装例**: 
  ```bash
  # コミットメッセージから抽出
  git log --grep="Protocol" --oneline
  # a7b3c9d Fix Protocol Seven initialization
  
  # 特定のコミットの詳細
  git show a7b3c9d --format=fuller
  
  # ブランチ名にヒント
  git branch -a | grep -E "shadow|seven|aegis"
  # feature/shadow-collective
  # hotfix/protocol-seven
  
  # Issue番号パターン
  # Issue #13, #37, #73 (素数のパターン)
  
  # GitHub APIを使用
  curl https://api.github.com/repos/alexchen/project-aegis/commits/a7b3c9d
  ```
- **隠しファイル**:
  ```bash
  # .gitignoreされたファイルを確認
  git ls-files --others --ignored --exclude-standard
  # secret_key.txt
  ```

### Twitter/X
- **活用方法**:
  - 特定のハッシュタグ
  - アカウントのプロフィール
  - 投稿時刻のパターン

### Pastebin系サービス
- **活用方法**:
  - 期限付きペースト
  - 特定のIDでアクセス

## 6. 電話・SMS

### 自動応答システム
- **活用方法**:
  - IVR（音声自動応答）でヒント提供
  - DTMF（プッシュ音）で暗号入力
- **実装例**: Amazon ConnectやTwilioを利用した自動応答システム

### SMS
- **活用方法**:
  - 二要素認証風の暗号送信
  - 時限式メッセージ

## 7. 地理的要素（一旦なし）

### GPS座標
- **活用方法**:
  - 実在の場所の座標
  - 架空の座標に意味を持たせる
  - What3Words形式

### 地図サービス
- **活用方法**:
  - Google Maps APIでカスタムマップ
  - ストリートビューの特定の場所

## 8. 時間的要素

### タイムスタンプ
- **活用方法**:
  - Unix時間を暗号の一部に
  - 特定の時刻にのみアクセス可能
  - タイムゾーンの違いを利用
- **実装例**:
  ```bash
  # Unixタイムスタンプの活用
  # 2025-06-15 03:42:00 UTC = 1750041720
  echo "1750041720" | base64
  
  # 時限式アクセス
  current_time=$(date +%s)
  target_time=1750041720
  if [ $current_time -ge $target_time ]; then
    echo "Protocol Seven key: $(cat secret.key)"
  else
    echo "Not yet. Wait $(($target_time - $current_time)) seconds."
  fi
  
  # タイムゾーンのヒント
  # PST: -8, EST: -5, UTC: 0, JST: +9
  # 例: 03:42 UTC = 19:42 PST (前日) = 12:42 JST
  ```
- **Cron形式の活用**:
  ```bash
  # 毎日午03:42にのみアクセス可能
  # 42 3 * * * /path/to/reveal_secret.sh
  ```

### スケジュール
- **活用方法**:
  - Cronジョブの記法でヒント
  - カレンダーイベント

## 9. 開発者ツール

### ブラウザコンソール（GitHubのPull Requestを利用するためなし）
- **活用方法**:
  - console.logに隠しメッセージ
  - JavaScriptの実行が必要な謎
  - デバッガーでブレークポイント

### ソースコード
- **活用方法**:
  - コメントアウトされたコード
  - 特定の変数名
  - minifyされたコードの復元

### Docker/コンテナ
- **活用方法**:
  - Dockerイメージ内に情報
  - 環境変数に暗号

## 10. その他のリソース

### ブロックチェーン
- **活用方法**:
  - トランザクションデータに情報
  - スマートコントラクト

## 実装時の注意事項

1. **倫理的配慮**: 
   - 実在のサービスや個人情報を使用しない
   - ダミーのドメインやIPアドレスを使用
   - 法的に問題のない範囲で実装

2. **コスト**: 
   - 無料サービスを優先的に使用
   - GitHub Pages、Cloudflare Workersなど
   - 有料APIは必要最小限に

3. **メンテナンス**: 
   - 外部サービスが停止しても謎が解けるように
   - バックアッププランを用意
   - ドキュメント化を徹底

4. **セキュリティ**: 
   - 参加者のシステムに悪影響を与えない
   - サンドボックス環境を提供
   - 明確な利用規約を設定

5. **難易度調整**:
   ```
   初級: DNS TXTレコード → Base64 → メッセージ
   中級: GitHub API → ステガノグラフィー → XOR
   上級: 時限式 + 地理的要素 + 複数レイヤー
   ```