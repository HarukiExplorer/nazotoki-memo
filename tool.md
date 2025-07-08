# 謎解きツール・リソース

## 1. ネットワークリソース

### IPアドレス
- **活用方法**: 特定のIPアドレスにアクセスすると次のヒントが表示される
- **実装例**: 
  - IPv4: `10.47.7.13` (Project Aegisの内部ネットワーク)
  - IPv6に暗号を埋め込む: `2001:db8:85a3::8a2e:370:7334`
  - 各オクテットがASCIIコード: `65.76.69.88` → "ALEX"
  - タイムベースIP: `10.47.{hour}.{minute}` (現在時刻に基づく)
- **謎解きのヒント**:
  ```bash
  # IPから情報を取得
  curl http://10.47.7.13:31337/hint
  nmap -sV -p- 10.47.7.13
  
  # 特定のUser-Agentが必要
  curl -A "AegisGuardian/1.0" http://10.47.7.13:31337/protocol
  
  # 時刻ベースのアクセス
  hour=$(date +%H)
  minute=$(date +%M)
  curl http://10.47.$hour.$minute/timelock
  ```
- **高度な実装**:
  ```python
  # IPアドレスからフィボナッチ数列を生成
  def ip_to_fibonacci(ip):
      octets = list(map(int, ip.split('.')))
      fib = [octets[0], octets[1]]
      for i in range(2, octets[2]):
          fib.append(fib[-1] + fib[-2])
      return fib[octets[3] % len(fib)]
  
  # 例: 10.47.7.13 → fib[13 % 7] = キー
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
  - ヘッダーの順序自体が暗号
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
  
  # 複数ヘッダーの組み合わせパズル
  curl -H "X-Layer-1: $(echo -n 'Protocol' | base64)" \
       -H "X-Layer-2: $(echo -n 'Seven' | rot13)" \
       -H "X-Layer-3: $(echo -n 'Activated' | md5sum | cut -c1-8)" \
       -H "X-Timestamp: $(date +%s)" \
       https://api.example.com/multilayer
  ```
- **レスポンスヘッダーの確認**:
  ```bash
  curl -I https://example.com | grep "X-"
  # X-Hint: Check commit a7b3c9d
  # X-Next-Step: /api/v2/protocol
  # X-Time-Window: 300 (5分間のみ有効)
  ```
- **動的ヘッダー検証**:
  ```python
  # リクエストヘッダーの順序で異なる結果
  def validate_header_sequence(headers):
      sequence = ''.join([h[0] for h in headers.keys()])
      if sequence == 'XPASGT':  # 特定の順序
          return reveal_secret()
      return "Invalid sequence"
  ```

### WebSocket
- **活用方法**: リアルタイムで変化するメッセージ
- **実装例**: 
  ```javascript
  // クライアント側実装
  const ws = new WebSocket('wss://puzzle.aegis.com/protocol');
  
  ws.onopen = () => {
    // 認証トークンを送信
    ws.send(JSON.stringify({
      type: 'auth',
      token: btoa('alexchen:' + Date.now())
    }));
  };
  
  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    // 時刻ベースメッセージ
    if (data.type === 'time_based' && new Date().getMinutes() === 42) {
      console.log('Protocol Seven Key:', data.secret);
    }
    
    // プレイヤーの進捗に応じたメッセージ
    if (data.type === 'progressive') {
      handleProgressiveHint(data.level, data.hint);
    }
  };
  ```
  ```python
  # サーバー側実装 (WebSocketサーバー)
  import asyncio
  import websockets
  import json
  from datetime import datetime
  
  async def puzzle_handler(websocket, path):
      authenticated = False
      
      async for message in websocket:
          data = json.loads(message)
          
          # 認証処理
          if data['type'] == 'auth':
              # トークン検証
              if validate_token(data['token']):
                  authenticated = True
                  await websocket.send(json.dumps({
                      'type': 'auth_success',
                      'message': 'Welcome to Protocol Seven'
                  }))
          
          # リアルタイムヒント配信
          if authenticated:
              # 特定時刻にのみ送信
              if datetime.now().minute == 42:
                  await websocket.send(json.dumps({
                      'type': 'time_based',
                      'secret': generate_time_based_key()
                  }))
              
              # プレイヤーのアクションに反応
              if data['type'] == 'action':
                  response = process_player_action(data['action'])
                  await websocket.send(json.dumps(response))
  ```

### API エンドポイント
- **活用方法**:
  - RESTful APIの特定のエンドポイントに情報を隠す
  - 認証トークンが必要なAPI
  - パラメータの組み合わせで異なる結果
- **実装例**: 
  ```bash
  # 基本エンドポイント
  curl https://api.aegis.com/v1/protocol/seven
  
  # 暗号化されたIDを使用
  encrypted_id=$(echo -n "alexchen-$(date +%s)" | base64)
  curl https://api.aegis.com/v1/secret/$encrypted_id
  
  # JWT認証が必要
  token="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  curl -H "Authorization: Bearer $token" \
       https://api.aegis.com/v1/classified/data
  
  # マルチパラメータパズル
  curl "https://api.aegis.com/v1/puzzle?\
  layer=7&\
  protocol=seven&\
  timestamp=$(date +%s)&\
  hash=$(echo -n 'aegis' | sha256sum | cut -c1-8)"
  ```
  ```python
  # APIサーバー実装例 (Flask)
  from flask import Flask, request, jsonify
  import jwt
  import hashlib
  from datetime import datetime, timedelta
  
  app = Flask(__name__)
  
  @app.route('/api/v1/secret/<encrypted_id>')
  def get_secret(encrypted_id):
      try:
          # IDを復号
          decrypted = base64.b64decode(encrypted_id).decode()
          username, timestamp = decrypted.split('-')
          
          # タイムスタンプ検証（5分以内）
          if int(time.time()) - int(timestamp) > 300:
              return jsonify({'error': 'Token expired'}), 401
          
          # ユーザーごとに異なるヒント
          hints = {
              'alexchen': 'Check commit a7b3c9d',
              'shadow': 'Protocol Seven awaits',
              'aegis': 'The truth is distributed'
          }
          
          return jsonify({
              'hint': hints.get(username, 'Unknown user'),
              'next': f'/api/v2/layer/{len(username)}'
          })
      except:
          return jsonify({'error': 'Invalid ID'}), 400
  
  @app.route('/api/v1/puzzle')
  def puzzle_endpoint():
      # パラメータの組み合わせを検証
      layer = request.args.get('layer')
      protocol = request.args.get('protocol')
      timestamp = request.args.get('timestamp')
      hash_param = request.args.get('hash')
      
      # 正しい組み合わせか確認
      if (layer == '7' and 
          protocol == 'seven' and 
          abs(int(timestamp) - int(time.time())) < 60):
          
          # ハッシュ検証
          expected_hash = hashlib.sha256(b'aegis').hexdigest()[:8]
          if hash_param == expected_hash:
              return jsonify({
                  'success': True,
                  'key': generate_protocol_key(),
                  'message': 'Protocol Seven initialized'
              })
      
      return jsonify({'error': 'Invalid parameters'}), 400
  ```

## 3. ファイル形式

### 画像ファイル
- **活用方法**:
  - ステガノグラフィー（データの隠蔽）
  - EXIFデータに座標や時刻情報
  - 画像内のQRコード
  - LSB（最下位ビット）への情報埋め込み
- **実装ツール**: 
  ```bash
  # steghideを使った埋め込み
  echo "Protocol Seven Key: a7b3c9d" > secret.txt
  steghide embed -cf aegis_logo.jpg -ef secret.txt -p "shadow"
  
  # 抽出
  steghide extract -sf aegis_logo.jpg -p "shadow"
  
  # EXIFデータの操作
  exiftool -GPSLatitude="37.7749" -GPSLongitude="-122.4194" \
           -Comment="The key lies in the commits" image.jpg
  
  # EXIFデータの確認
  exiftool image.jpg | grep -E "GPS|Comment"
  
  # zstegでのLSB分析
  zsteg suspicious_image.png
  # b1,rgb,lsb,xy .. text: "alexchen:protocol7"
  
  # binwalkで隠しファイル検出
  binwalk -e image.jpg
  # 0x1000     ZIP archive "hidden_data.zip"
  ```
  ```python
  # LSBステガノグラフィー実装
  from PIL import Image
  import numpy as np
  
  def embed_lsb(image_path, message, output_path):
      img = Image.open(image_path)
      pixels = np.array(img)
      
      # メッセージをバイナリに変換
      binary_msg = ''.join(format(ord(c), '08b') for c in message)
      binary_msg += '00000000'  # 終端マーカー
      
      msg_idx = 0
      for i in range(pixels.shape[0]):
          for j in range(pixels.shape[1]):
              if msg_idx < len(binary_msg):
                  # RGB各チャンネルのLSBを修正
                  for k in range(3):
                      if msg_idx < len(binary_msg):
                          pixels[i,j,k] = (pixels[i,j,k] & 0xFE) | int(binary_msg[msg_idx])
                          msg_idx += 1
      
      # 保存
      Image.fromarray(pixels).save(output_path)
  
  def extract_lsb(image_path):
      img = Image.open(image_path)
      pixels = np.array(img)
      
      binary_msg = ''
      for i in range(pixels.shape[0]):
          for j in range(pixels.shape[1]):
              for k in range(3):
                  binary_msg += str(pixels[i,j,k] & 1)
      
      # バイナリを文字に変換
      message = ''
      for i in range(0, len(binary_msg), 8):
          byte = binary_msg[i:i+8]
          if byte == '00000000':  # 終端マーカー
              break
          message += chr(int(byte, 2))
      
      return message
  ```

### 音声ファイル
- **活用方法**:
  - スペクトログラムで文字を表示
  - 逆再生で聞こえるメッセージ
  - 特定の周波数に情報を埋め込む
  - DTMFトーンで数値情報をエンコード
- **実装ツール**: 
  ```bash
  # 逆再生メッセージの作成
  # 1. メッセージを音声合成
  espeak "Protocol Seven Activated" -w forward.wav
  
  # 2. 逆再生
  sox forward.wav reversed.wav reverse
  
  # 3. オリジナルファイルにミックス
  sox -m original.wav reversed.wav mixed.wav
  
  # スペクトログラム分析
  sox audio.wav -n spectrogram -o spectrum.png
  
  # 特定周波数の抽出
  sox audio.wav filtered.wav highpass 15000
  
  # DTMFトーン生成
  multimon-ng -a DTMF audio.wav
  # DTMF: 1337042  # 隠されたコード
  ```
  ```python
  # スペクトログラムに文字を埋め込む
  import numpy as np
  import matplotlib.pyplot as plt
  from scipy.io import wavfile
  from PIL import Image, ImageDraw, ImageFont
  
  def create_spectrogram_message(message, output_file):
      # テキストを画像に変換
      img = Image.new('L', (800, 200), 0)
      draw = ImageDraw.Draw(img)
      font = ImageFont.truetype('/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf', 60)
      draw.text((10, 50), message, fill=255, font=font)
      
      # 画像をスペクトログラムデータに変換
      img_array = np.array(img)
      
      # 周波数ドメインにマッピング
      sample_rate = 44100
      duration = 10  # 秒
      samples = sample_rate * duration
      
      # 逆FFTで音声信号を生成
      audio = np.zeros(samples)
      for t in range(0, samples, 1024):
          window = np.zeros(1024)
          # スペクトログラムの各列を周波数成分として使用
          col_idx = int(t / samples * img_array.shape[1])
          if col_idx < img_array.shape[1]:
              freqs = img_array[:, col_idx]
              for i, amp in enumerate(freqs):
                  if amp > 128:  # 闾値
                      freq = 5000 + i * 50  # 5kHzから15kHzまで
                      window += amp * np.sin(2 * np.pi * freq * np.arange(1024) / sample_rate)
          
          audio[t:t+1024] += window[:min(1024, samples-t)]
      
      # 正規化して保存
      audio = np.int16(audio / np.max(np.abs(audio)) * 32767)
      wavfile.write(output_file, sample_rate, audio)
  
  # モールス信号の音声化
  def morse_to_audio(morse_code, output_file):
      sample_rate = 44100
      dot_duration = 0.1  # 秒
      dash_duration = 0.3  # 秒
      frequency = 800  # Hz
      
      audio = []
      
      for char in morse_code:
          if char == '.':
              duration = dot_duration
          elif char == '-':
              duration = dash_duration
          elif char == ' ':
              # 無音
              audio.extend(np.zeros(int(0.2 * sample_rate)))
              continue
          else:
              continue
          
          # トーン生成
          t = np.linspace(0, duration, int(duration * sample_rate))
          tone = np.sin(2 * np.pi * frequency * t)
          audio.extend(tone)
          
          # 短い無音
          audio.extend(np.zeros(int(0.1 * sample_rate)))
      
      # 保存
      audio = np.array(audio)
      audio = np.int16(audio * 32767)
      wavfile.write(output_file, sample_rate, audio)
  ```

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
  - 部分的QRコード（複数を組み合わせて完成）
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
  
  # 時刻ベースQRコード生成
  message="$(date +%H:%M) - Protocol Seven Key: $(openssl rand -hex 16)"
  qrencode -o "qr_$(date +%Y%m%d_%H%M).png" "$message"
  ```
- **エラー訂正レベルの活用**:
  ```python
  import qrcode
  from PIL import Image, ImageDraw
  
  # エラー訂正レベルH(30%)でも読めるように
  # QRコードの一部を意図的に破損させて隠しメッセージを埋め込む
  def create_damaged_qr(data, hidden_msg):
      qr = qrcode.QRCode(
          error_correction=qrcode.constants.ERROR_CORRECT_H,
          box_size=10,
          border=4,
      )
      qr.add_data(data)
      qr.make(fit=True)
      
      img = qr.make_image(fill_color="black", back_color="white")
      draw = ImageDraw.Draw(img)
      
      # 特定の位置に隠しパターンを描画
      # QRコードは読めるが、パターンが別のヒントになる
      for i, char in enumerate(hidden_msg):
          x = 50 + i * 20
          y = 50 + ord(char) % 50
          draw.rectangle([x, y, x+10, y+10], fill="gray")
      
      return img
  ```
- **分割QRコード**:
  ```python
  # QRコードを4つに分割し、別々の場所に配置
  def split_qr_code(qr_image):
      width, height = qr_image.size
      mid_w, mid_h = width // 2, height // 2
      
      # 4つの部分に分割
      parts = [
          qr_image.crop((0, 0, mid_w, mid_h)),         # 左上
          qr_image.crop((mid_w, 0, width, mid_h)),     # 右上
          qr_image.crop((0, mid_h, mid_w, height)),    # 左下
          qr_image.crop((mid_w, mid_h, width, height)) # 右下
      ]
      
      # 各部分を異なる場所に隠す
      # 例: Git履歴、画像EXIF、DNSレコード、HTTPヘッダー
      return parts
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
  - Pull Requestのレビューコメント
  - Actions の実行結果
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
  
  # コミットハッシュの最初の文字で文章を作成
  git log --oneline -20 | cut -c1 | tr -d '\n'
  # "SEVENLAYERSDEEP"
  
  # タグに隠されたメッセージ
  git tag -l -n99 | grep -E "v[0-9]+\.[4-7]\.[0-9]+"
  # v1.4.2  Critical security update - Check line 47
  # v2.7.3  Performance improvements - X marks the spot
  ```
- **隠しファイル**:
  ```bash
  # .gitignoreされたファイルを確認
  git ls-files --others --ignored --exclude-standard
  # secret_key.txt
  
  # 削除されたファイルの履歴を探索
  git log --diff-filter=D --summary | grep delete
  # delete mode 100644 src/protocols/seven.key
  
  # 特定のファイルの過去のバージョンを取得
  git show HEAD~7:src/config/hidden.json
  ```
- **GitHub Actions の活用**:
  ```yaml
  # .github/workflows/puzzle.yml
  name: Puzzle Validator
  on:
    pull_request:
      types: [opened]
  
  jobs:
    validate:
      runs-on: ubuntu-latest
      steps:
        - name: Check Solution
          run: |
            if [[ "${{ github.event.pull_request.title }}" == *"Protocol Seven"* ]]; then
              echo "::set-output name=hint::$(echo 'Next: Check commit a7b3c9d' | base64)"
            fi
  ```
- **高度なGit操作**:
  ```python
  import subprocess
  import hashlib
  
  def extract_git_puzzle():
      # コミット履歴から特定パターンを抽出
      commits = subprocess.check_output(
          ['git', 'log', '--pretty=%H %s', '-100']
      ).decode().split('\n')
      
      # コミットメッセージの頭文字で暗号を作成
      initials = ''.join([c.split()[1][0] for c in commits if c])
      
      # 特定のコミットハッシュを組み合わせ
      special_commits = [c for c in commits if '7' in c.split()[0][:7]]
      
      # ハッシュの7文字目でキーを生成
      key = ''.join([c[6] for c in special_commits[:7]])
      
      return f"Initials: {initials}, Key: {key}"
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
  - 特定の時刻にかけると異なるメッセージ
- **実装例**: 
  ```python
  # Twilioを使ったIVRシステム
  from twilio.twiml.voice_response import VoiceResponse, Gather
  from datetime import datetime
  
  def handle_call():
      response = VoiceResponse()
      
      # 時刻ベースメッセージ
      hour = datetime.now().hour
      if hour == 3:  # 午03時
          response.say("Protocol Seven active. Enter code.")
      else:
          response.say("Access denied. Call back at 03:00 UTC.")
      
      # DTMF入力を受け付け
      gather = Gather(num_digits=7, action='/verify')
      gather.say("Enter seven digit access code")
      response.append(gather)
      
      return str(response)
  
  def verify_code(digits):
      # 1337042 = Elite timestamp
      if digits == '1337042':
          response = VoiceResponse()
          response.say("Access granted. Check repository a7b3c9d.")
          # 音声でモールス信号
          response.play('https://example.com/morse_hint.wav')
      else:
          response = VoiceResponse()
          response.say("Invalid code. Connection terminated.")
      
      return str(response)
  ```

### SMS
- **活用方法**:
  - 二要素認証風の暗号送信
  - 時限式メッセージ
  - 複数SMSを組み合わせたパズル
- **実装例**:
  ```python
  import time
  from twilio.rest import Client
  
  def send_puzzle_sms(phone_number):
      # 7つの部分に分割したSMSを送信
      messages = [
          "[1/7] Protocol activation sequence initiated.",
          "[2/7] Key fragment: a7b3",
          "[3/7] Timestamp: 1750041720",
          "[4/7] Repository: github.com/alexchen/",
          "[5/7] Branch: feature/shadow-collective",
          "[6/7] Commit: Check seventh character",
          "[7/7] Combine all sevens for access."
      ]
      
      client = Client(account_sid, auth_token)
      
      for i, msg in enumerate(messages):
          # 各メッセージを遅延を入れて送信
          client.messages.create(
              body=msg,
              from_='+1234567890',
              to=phone_number
          )
          time.sleep(7)  # 7秒間隔
      
      # 最後に消えるSMS（Webhookで削除）
      final_msg = client.messages.create(
          body="This message will self-destruct in 42 seconds.",
          from_='+1234567890',
          to=phone_number
      )
      
      # 42秒後に削除
      schedule_deletion(final_msg.sid, 42)
  ```

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
  - 時間差分で暗号キーを生成
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
  
  # 時間窓アクセス（特定の分だけ）
  minute=$(date +%M)
  if [ $minute -eq 42 ]; then
    echo "Access granted: $(openssl rand -base64 32)"
  else
    echo "Access denied. Try at minute :42"
  fi
  ```
- **Cron形式の活用**:
  ```bash
  # 毎日午03:42にのみアクセス可能
  # 42 3 * * * /path/to/reveal_secret.sh
  
  # フィボナッチ時刻パターン
  # 分: 1, 1, 2, 3, 5, 8, 13, 21, 34, 55
  # これらの分にのみ新しいヒントが出現
  ```
- **時間ベース暗号化**:
  ```python
  import time
  import hashlib
  from datetime import datetime, timezone
  
  def time_based_encryption(message, time_window=300):
      # 5分間の時間窓で同じキーを生成
      current_epoch = int(time.time())
      time_slot = current_epoch // time_window
      
      # 時間スロットからキーを生成
      key = hashlib.sha256(f"aegis-{time_slot}".encode()).hexdigest()
      
      # XOR暗号化（簡単な例）
      encrypted = ''
      for i, char in enumerate(message):
          key_char = key[i % len(key)]
          encrypted += chr(ord(char) ^ ord(key_char))
      
      return encrypted, time_slot
  
  # 特定の時刻にのみ復号可能
  def conditional_decrypt(encrypted, target_hour=3, target_minute=42):
      now = datetime.now(timezone.utc)
      if now.hour == target_hour and now.minute == target_minute:
          # 復号処理
          return decrypt_message(encrypted)
      else:
          next_time = now.replace(hour=target_hour, minute=target_minute)
          if next_time < now:
              next_time = next_time.replace(day=now.day + 1)
          wait_time = (next_time - now).total_seconds()
          return f"Decryption available in {int(wait_time)} seconds"
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
  - コンテナラベルを使ったヒント
  - レイヤー構造に暗号を埋め込み
- **実装例**:
  ```dockerfile
  # Dockerfile
  FROM alpine:latest
  
  # 環境変数にヒント
  ENV PROTOCOL="Seven"
  ENV LAYER_COUNT="7"
  ENV SECRET_PATH="/opt/aegis/.hidden"
  
  # ラベルに情報
  LABEL maintainer="alexchen@projectaegis.com"
  LABEL version="1.3.3.7"
  LABEL hint="Check the seventh layer"
  LABEL commit="a7b3c9d"
  
  # マルチステージビルドで各ステージにヒント
  FROM alpine:latest as builder
  RUN echo "Protocol Seven Key Part 1: a7b3" > /tmp/part1.txt
  
  FROM alpine:latest as runtime
  COPY --from=builder /tmp/part1.txt /opt/part1.txt
  RUN echo "Protocol Seven Key Part 2: c9d" > /opt/part2.txt
  
  # 隠しファイル
  RUN mkdir -p /opt/aegis/.hidden && \
      echo "The truth is distributed" > /opt/aegis/.hidden/message.txt && \
      chmod 600 /opt/aegis/.hidden/message.txt
  
  # スクリプトにヒントを埋め込み
  COPY <<EOF /entrypoint.sh
  #!/bin/sh
  # Protocol Seven Initialization
  if [ "\$1" = "--reveal" ]; then
      cat /opt/part1.txt /opt/part2.txt | tr -d '\n'
      echo ""
  fi
  exec "\$@"
  EOF
  
  RUN chmod +x /entrypoint.sh
  ENTRYPOINT ["/entrypoint.sh"]
  ```
  ```bash
  # Dockerイメージの分析
  # ラベル情報の確認
  docker inspect aegis-puzzle:latest | jq '.[0].Config.Labels'
  
  # 環境変数の確認
  docker run --rm aegis-puzzle:latest env | grep -E "PROTOCOL|LAYER|SECRET"
  
  # イメージのレイヤー履歴
  docker history aegis-puzzle:latest --no-trunc
  
  # コンテナ内の隠しファイルを探索
  docker run --rm -it aegis-puzzle:latest find / -name ".*" -type f 2>/dev/null
  
  # Docker Composeでの謎解き
  ```
  ```yaml
  # docker-compose.yml
  version: '3.7'
  
  services:
    puzzle-1:
      image: aegis-puzzle:layer1
      environment:
        - HINT=Check_layer_7
      labels:
        - "puzzle.order=1"
        - "puzzle.next=puzzle-2"
    
    puzzle-2:
      image: aegis-puzzle:layer2
      depends_on:
        - puzzle-1
      environment:
        - PREVIOUS_KEY=${LAYER1_KEY}
      labels:
        - "puzzle.order=2"
        - "puzzle.key.partial=a7b3"
    
    # ... 続く
    
    puzzle-7:
      image: aegis-puzzle:final
      depends_on:
        - puzzle-6
      environment:
        - FINAL_KEY=${LAYER1_KEY}${LAYER2_KEY}${LAYER3_KEY}
      ports:
        - "31337:31337"
      labels:
        - "puzzle.order=7"
        - "puzzle.final=true"
  ```

## 10. その他のリソース

### ブロックチェーン
- **活用方法**:
  - トランザクションデータに情報
  - スマートコントラクト
  - NFTメタデータに暗号
  - ブロックハッシュのパターン
- **実装例**:
  ```solidity
  // Ethereumスマートコントラクト
  pragma solidity ^0.8.0;
  
  contract ProtocolSevenPuzzle {
      mapping(address => string) private playerKeys;
      string private constant HINT = "Check block 1337042";
      
      function submitKey(string memory key) public {
          require(bytes(key).length == 7, "Key must be 7 chars");
          playerKeys[msg.sender] = key;
          
          // 7人が異なるキーを提出した場合
          if (checkAllKeys()) {
              emit ProtocolSevenUnlocked("a7b3c9d");
          }
      }
      
      function getHint() public pure returns (string memory) {
          // ブロック番号が1337042の時のみ
          if (block.number == 1337042) {
              return "The truth is distributed";
          }
          return HINT;
      }
  }
  ```
  ```python
  # BitcoinトランザクションにOP_RETURNでデータ埋め込み
  from bitcoinlib.transactions import Transaction
  
  def embed_puzzle_in_bitcoin():
      # OP_RETURNで最大80バイトのデータを埋め込み
      data = "Protocol Seven: github.com/alexchen/keys"
      tx = Transaction()
      tx.add_output(0, script_type='nulldata', 
                     data=data.encode('utf-8'))
      
      # トランザクションIDの最初の7文字がヒント
      # 例: a7b3c9d...
      return tx
  ```

### AI/機械学習要素
```python
# プレイヤーの行動パターンからヒントを生成
from sklearn.cluster import KMeans
import numpy as np

def adaptive_puzzle_system(player_actions):
    """
    プレイヤーの行動を分析して
    適切な難易度のヒントを提供
    """
    # プレイヤーをスキルレベルでクラスタリング
    kmeans = KMeans(n_clusters=3)  # 初級、中級、上級
    clusters = kmeans.fit_predict(player_actions)
    
    hints = {
        0: "Base64デコードを試してみてください",
        1: "Gitコミットの7番目の文字を確認",
        2: "LSBステガノグラフィー + XOR暗号"
    }
    
    return hints[clusters[-1]]
```

## 高度な謎解きテクニック

### マルチプレイヤー要素
```python
# 複数プレイヤーが必要なパズル
def distributed_key_system():
    """
    7人のプレイヤーがそれぞれ異なる方法で
    キーの一部を取得し、全員が揃って初めて
    Protocol Sevenを解除できる
    """
    keys = {
        'player1': 'GitHubコミットから: a7',
        'player2': 'DNS TXTレコードから: b3',
        'player3': '画像LSBから: c9',
        'player4': '音声スペクトログラムから: d1',
        'player5': 'WebSocketリアルタイムから: e5',
        'player6': 'Dockerラベルから: f7',
        'player7': 'QRコード分割から: 42'
    }
    # 全員のキーを結合: a7b3c9d1e5f742
```

### ブロックチェーン活用
```bash
# Ethereumスマートコントラクトに暗号を保存
# トランザクションハッシュ: 0xa7b3c9d...
# ブロック番号: 1337042
# コントラクトアドレスの下7桁がヒント
```

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
   - レートリミット実装（DDoS対策）

5. **難易度調整**:
   ```
   初級: DNS TXTレコード → Base64 → メッセージ
   中級: GitHub API → ステガノグラフィー → XOR
   上級: 時限式 + WebSocket + Docker + 複数レイヤー
   エキスパート: マルチプレイヤー + ブロックチェーン
   ```

6. **プレイヤー体験**:
   - 成功体験を早めに提供
   - ヒントシステムの実装
   - コミュニティでの情報共有を促進