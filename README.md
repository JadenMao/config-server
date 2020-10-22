# config-server
Spring Cloud Config Server

## How to use

### 建立資料夾
``` bash
mkdir config
```

配置 Sonfig Server 設定檔  
因為 private-key 很長, 放在 env 很難看, 所以我才放檔案, 不然也是可以透過環境變數設定
```
cat > config/application.yml << EOF
spring:
  cloud:
    config:
      server:
        git:
          uri: git@gitlab.com:demo/dev-config.git
          private-key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIJKAIBAAKCAgEAuUVbnX6e4JdbN1rLJw875IAkc0j/sd7sA4HAmePYh/2xUW2Y
            qGiwZMOEhfZtFCa5LzWVm2ELd5YYYOOEsEVRFPQu4LGBfGS2AFzoQnPnLh+8HS3Q
            AqivmlIMlLgypGq/zHFsRyj/gksipxLkcC9mK6yWQherk1iVoMgKpf16YZs=
            -----END RSA PRIVATE KEY-----
EOF
```

### 設定服務  
修改 encrypt.key 為你要使用的金鑰
```
cat > docker-compose.yml << EOF
version: "3.8"
services:
  config-server:
    image: ghcr.io/cloud-technology/config-server/config-server:3
    container_name: config-server
    ports:
      - "8080:8080"
    volumes:
      - ./config:/config
    environment:
      JAVA_OPTS: "-Dfile.encoding=utf-8 -Djava.security.egd=file:/dev/./urandom"
      spring.config.location: "file:/config/"
      spring.cloud.config.server.git.ignore-local-ssh-settings: "true"
      spring.cloud.config.server.git.force-pull: "true"
      spring.cloud.config.server.git.clone-on-start: "true"
      encrypt.key: "12345678"
EOF
```

### 啟動 Config Server
```
docker-compose up
```

## 加密資料
``` bash
curl --location --request POST 'http://localhost:8080/encrypt' \
--header 'Content-Type: text/plain' \
--data-raw 'SamZhu'
```

## 解密資料
```bash
curl --location --request POST 'http://localhost:8080/decrypt' \
--header 'Content-Type: text/plain' \
--data-raw '1498c43e2c8e671dc9b1663a05e1683491549e5cb1f895af8f6d957c1ad2bc80'
```

## 測試取得設定
``` bash
curl --location --request GET 'localhost:8080/${application-name}/${profile}'
```
取得 git 上 application.yml + ${application-name}.yml + ${application-name}-${profile}.yml  
即可替服務做啟動
