# 暗号メモ - 謎解きクイズ用

## 1. 古典暗号

### シーザー暗号（Caesar Cipher）
- **概要**: アルファベットを固定数ずらす単純な換字式暗号
- **暗号化**: 各文字を指定した数だけ後ろにシフト
- **復号**: 逆方向に同じ数だけシフト
- **例**: ROT13（13文字シフト）
- **実装例**:
  ```python
  # 暗号化
  def caesar_encrypt(text, shift=13):
      return ''.join(chr((ord(c) - ord('A') + shift) % 26 + ord('A')) 
                     if c.isupper() else c for c in text)
  ```
- **謎解き活用例**: 
  - Gitコミットメッセージ: `Fix bug in CEBWRPG NRTVF` (ROT13)
  - ファイル名: `frperg_svyr.gkg` → `secret_file.txt`
  - ヒント: シフト数自体を謎にする（例：ポート番号の下2桁）

### ヴィジュネル暗号（Vigenère Cipher）
- **概要**: キーワードを使った多表式換字暗号
- **暗号化**: キーワードの各文字をシフト量として使用
- **復号**: キーワードを使って逆シフト
- **実装例**:
  ```python
  def vigenere_encrypt(text, key):
      key = key.upper()
      result = ''
      for i, char in enumerate(text.upper()):
          if char.isalpha():
              shift = ord(key[i % len(key)]) - ord('A')
              result += chr((ord(char) - ord('A') + shift) % 26 + ord('A'))
      return result
  ```
- **謎解き活用例**: 
  - キー: `PROJECTAEGIS`
  - ヒントをREADMEに埋め込む: "The **P**ower **R**ests **O**n..."
  - ブランチ名がキー: `feature/shadow-collective`

### ポリビウス暗号（Polybius Square）
- **概要**: 文字を2桁の数字に変換
- **暗号化**: 5×5のグリッドで文字を座標に変換
- **復号**: 座標から文字を復元
- **謎解き活用例**: IPアドレスやポート番号に見せかける

### エニグマ暗号（Enigma Machine）
- **概要**: ドイツの暗号機、回転式ローターを使用
- **暗号化**: 複数のローターとプラグボードを組み合わせて文字を変換
- **復号**: 同じ設定で再度暗号化
- **謎解き活用例**: 複雑なパスワードやキーを生成

## 2. エンコーディング系

### Base64
- **概要**: バイナリデータをASCII文字列に変換
- **暗号化**: 
  ```bash
  echo "secret message" | base64
  # c2VjcmV0IG1lc3NhZ2UK
  ```
- **復号**: 
  ```bash
  echo "c2VjcmV0IG1lc3NhZ2UK" | base64 -d
  # secret message
  ```
- **謎解き活用例**: 
  - コミットハッシュに埋め込む: `git commit -m "UHJvdG9jb2wgU2V2ZW4gYWN0aXZhdGVk"`
  - Authorizationヘッダー: `Basic YWxleGNoZW46cHJvamVjdGFlZ2lz`
  - 画像のdata URI: `data:image/png;base64,iVBORw0KGgo...`

### URL エンコーディング
- **概要**: URLで使用できない文字を%XXで表現
- **暗号化**: encodeURIComponent()
- **復号**: decodeURIComponent()
- **謎解き活用例**: APIエンドポイントやパラメータに隠す

### 16進数（Hex）エンコーディング
- **概要**: バイナリデータを16進数文字列に変換
- **暗号化**: xxd, hexdump
- **復号**: xxd -r
- **謎解き活用例**: カラーコードやメモリアドレスに偽装

## 3. プログラミング関連の暗号化

### XOR暗号
- **概要**: ビット単位の排他的論理和を使用
- **暗号化**: plaintext XOR key
- **復号**: ciphertext XOR key（同じ操作）
- **実装例**:
  ```python
  def xor_encrypt(data, key):
      return bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])
  
  # 例: IPアドレスをキーとして使用
  ip_bytes = bytes(map(int, '10.47.7.13'.split('.')))
  encrypted = xor_encrypt(b'Protocol Seven', ip_bytes)
  ```
- **謎解き活用例**: 
  - バイナリファイルのヘッダーをXORで隠す
  - キーはファイルサイズやタイムスタンプ
  - 複数回XORを適用して難易度を上げる

### ハッシュ関数
- **MD5**: 128ビットのハッシュ値（衝突の可能性あり）
- **SHA-256**: 256ビットのハッシュ値（より安全）
- **謎解き活用例**: ファイル名やコミットハッシュとして使用

### JWT（JSON Web Token）
- **概要**: Header.Payload.Signatureの形式
- **構造**:
  ```json
  // Header
  {"alg": "HS256", "typ": "JWT"}
  // Payload
  {"sub": "alexchen", "role": "guardian", "exp": 1718380800}
  ```
- **復号**: 
  ```bash
  # JWTをデコード
  echo "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGV4Y2hlbiJ9.X" | 
    cut -d'.' -f2 | base64 -d
  ```
- **謎解き活用例**: 
  - Payloadに次のヒントを埋め込む
  - 署名の秘密鍵が別の謎の答え
  - 有効期限(exp)が特定の日時を示す

## 4. CTF系暗号

### ステガノグラフィー
- **概要**: 画像や音声ファイルにメッセージを隠す
- **手法**: LSB（最下位ビット）への埋め込み
- **ツール**: 
  ```bash
  # 埋め込み
  steghide embed -cf cover.jpg -ef secret.txt -p "password"
  
  # 抽出
  steghide extract -sf cover.jpg -p "password"
  
  # 分析
  zsteg image.png
  binwalk -e suspicious.jpg
  ```
- **謎解き活用例**: 
  - プロジェクトロゴに暗号を埋め込む
  - スクリーンショットのLSBにデータを隠す
  - パスワードは別の謎から取得: `git log --oneline | head -1`

### モールス信号
- **概要**: 短点(.)と長点(-)で文字を表現
- **暗号化**: 文字を・と－に変換
- **復号**: モールス信号表を参照
- **謎解き活用例**: ログのタイムスタンプやHTTPステータスコードのパターン

### バイナリ/ASCII変換
- **概要**: 文字を2進数で表現
- **暗号化**: 各文字を8ビットの2進数に変換
- **復号**: 8ビットごとに文字に変換
- **謎解き活用例**: コミットIDやエラーコードとして使用

## 5. 現代暗号（ヒントとして使用）

### RSA暗号
- **概要**: 公開鍵暗号方式
- **謎解き活用例**: 公開鍵を見つけて秘密のメッセージを暗号化

### AES暗号
- **概要**: 対称鍵暗号方式
- **謎解き活用例**: 設定ファイルやデータベースの暗号化

## 6. その他のテクニック

### 逆順（Reverse）
- **概要**: 文字列を逆順にする
- **謎解き活用例**: ファイル名やコメントで使用

### 文字の置換
- **概要**: 特定の文字を別の文字に置き換え
- **例**: l33t speak（1337 5p34k）
- **謎解き活用例**: 変数名やコメントで使用

### ゼロ幅文字
- **概要**: 見えない文字（U+200B, U+200C, U+200D）
- **謎解き活用例**: ソースコード内に見えないメッセージを埋め込む

## 実装時の注意点

1. **難易度の調整**: 
   - **初級**: 単一の暗号（ROT13、Base64）
   - **中級**: 2-3個の暗号の組み合わせ
   - **上級**: 暗号+ステガノグラフィー+プログラミング

2. **ヒントの配置**: 
   - ファイル名や変数名にヒント: `rot13_encoded.txt`
   - コメントに暗示: `// Thirteen steps forward`
   - エラーメッセージ: `Error: Invalid base (expecting 64)`

3. **コンテキスト**: 
   - Protocol Seven → 7層の暗号化
   - Shadow Collective → 影(ステガノグラフィー)
   - Alex Chen (A.C.) → イニシャルをキーに

4. **ツールの使用**: 
   ```bash
   # 標準的なツールを使用
   git log --oneline | grep "AC" | cut -d' ' -f1
   curl -H "X-Cipher: $(echo $KEY | base64)" https://api.example.com
   openssl enc -aes-256-cbc -d -in encrypted.bin -out plain.txt
   ```

5. **複合的な謎の例**:
   ```
   Step 1: GitコミットからBase64文字列を取得
   Step 2: Base64デコード → XORキーを取得
   Step 3: 画像ファイルをXORキーで復号
   Step 4: 画像からステガノグラフィーで最終メッセージ
   ```