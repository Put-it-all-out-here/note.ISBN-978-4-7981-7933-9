## p43
- dockerのインストール先：ホストとなるwindows  
- Docker DesktopをWindowsにインストールする際、  
  セットアップ画面に表示される  
　「Allow Windows Containers to be used with this installation」  
　というチェックボックスへの対応:空のままでよい。  
    - 参考：https://nakaterux.hatenablog.com/entry/2025/08/18/000230

## p46
- ホストのポートが他のプロセスで使用されていないか確認
  - コマンドラインで実行：`>netstat -ano | findstr :53306`
  - 空行が返れば53306は使用されていないポートということ
- docker-compose.yml
  - 本に在る例の修正が必要
    - 参考:https://qiita.com/tseno/items/8c0adf3d78bd613f2515
      - 修正前
        ```
        services:
          app-db:
            image: mysql:8
            command: --collation-server=utf8mb4_0900_bin --transaction-isolation=READ-COMMITTED
            restart: always
            environment:
              MYSQL_ROOT_PASSWORD: example
              TZ: Asia/Tokyo
            ports:
              - 53306:3306
        ``` 
      - 修正後
          ```
          services:
            app-db:
              image: mysql:8
              command: --collation-server=utf8mb4_0900_bin --transaction-isolation=READ-COMMITTED
              restart: always
              environment:
                MYSQL_ROOT_PASSWORD: example
                MYSQL_DATABASE: app
                MYSQL_USER: root1
                MYSQL_PASSWORD: password1
                TZ: Asia/Tokyo
              ports:
                - 53306:3306          
          ```
## p47
- Docker Desktopインストール後、Docker Desktop起動して、
  右下角にUpdateの表示あれば、クリックし、Updateする。
  
- <img width="1587" height="900" alt="image" src="https://github.com/user-attachments/assets/7dc772ea-200a-4013-9088-c7d7ff3fe5b3" />
- <img width="1587" height="900" alt="image" src="https://github.com/user-attachments/assets/471d5e90-6eec-4dd4-8243-1b76bbfb96fa" />

- dockerコマンドが見つからない。
- `docker compose up -d` は失敗する。  
  環境変数PATHにdockerのbinを登録する下りがこの本にはない。
  環境変数の登録をしないパターンの進め方:
    ```
    cd C:\Program Files\Docker\Docker\resources\bin
    docker compose -f "C:\dev\docker.compose.yml" up -d
    ```
