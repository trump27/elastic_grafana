# Elasticksearch 7.4 & Grafana

nginxでリバースプロキシ & certbot
```
grafana: /
elastic: /elastic
kibana: /kibana
```

## 補足
- メモリ2GB推奨らしいが1.5GBでも動作した
- nginxでリバースプロキシ
- sample with Xpack.security
- Kibanaはコメントしている
- kibanaへはelasticsearchに指定したパスワードでログイン。ユーザはelastic

## 準備

ローカルに格納するためのフォルダ作成と権限付与
```bash
mkdir es-data
chown -R 1000 es-data
mkdir grafana-data
sudo chown -R 472 grafana-data
```

## 証明書

### Lets Encryptで証明書取得

クラウドで使用する場合はdnsを設定、80番ポート解放後に行う
```bash
sudo certbot-auto certonly -d xxxx.yyyy.com
# 1: Spin up a temporary webserver (standalone) を選択
```
証明書取得後のメッセージに表示されるパスを設定
```
- '/etc/letsencrypt/live/xxxx.yyyy.com/fullchain.pem:/etc/nginx/server.crt:ro'
- '/etc/letsencrypt/live/xxxx.yyyy.com/privkey.pem:/etc/nginx/server.key:ro'
```

#### 証明書の更新

```
certbot-auto renew

# 自動更新
sudo vi /etc/cron.d/letsencrypt
# 設定
00 22 * * 2 root /usr/sbin/certbot-auto renew --post-hook "docker restart elastic_nginx_1"
```

### 自己署名証明書

Chrome(77.0)で弾かれるなど不便な点が多いため推奨しない
```bash
cd nginx/
openssl genrsa -des3 -out server.key 2048
openssl req -new -key server.key -out server.csr
# Organizational Unit Name, Common Name, Email, A challenge password, An optional company nameは空

cp server.key server.key.org
openssl rsa -in server.key.org -out server.key

openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
```

## dockerインストール

https://docs.docker.com/install/linux/docker-ce/ubuntu/

```bash
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
sudo apt-get install -y docker-ce

sudo docker run hello-world
```

## dcoker-composeインストール

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## Elasticsearchのためのホスト設定

```bash
sudo sysctl -w vm.max_map_count=262144
```