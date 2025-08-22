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
  Docker Desktop起動していないと、dockerコマンドの途中でエラーになる。
  
  - <img width="1587" height="900" alt="image" src="https://github.com/user-attachments/assets/7dc772ea-200a-4013-9088-c7d7ff3fe5b3" />
  - <img width="1587" height="900" alt="image" src="https://github.com/user-attachments/assets/471d5e90-6eec-4dd4-8243-1b76bbfb96fa" />

- 環境変数PATHにdockerのbinを登録する下りがこの本にはない。  
  結果、`docker compose up -d` はコマンドが見つからず失敗する。  
  - 対応:
    - 1:環境変数の登録をしないで実行したいので、カレントディレクトリ移動してコマンド実行  
      ```
      cd C:\Program Files\Docker\Docker\resources\bin
      docker compose -f "C:\dev\docker.compose.yml" up -d
      ```
    - 2:'docker compose -f "C:\dev\docker.compose.yml" up -d'で下記エラー発生  
      'error getting credentials - err: exec: "docker-credential-desktop": cannot run executable found relative to current directory, out: '  
    - 3:コマンドプロンプトで'type %USERPROFILE%\.docker\config.json'実行
    - 4:結果
      ```
      {
        "auths": {},
        "credsStore": "desktop",
        "currentContext": "desktop-linux"
      }
      ```
    - 5:`config.json`に`docker-credential-desktop`を呼び出す記載があるが、
      `docker-credential-desktop`を呼び出せないことによるエラー。
      調査によると、`docker-credential-desktop`を呼び出すために動く機能'exec.LookPath'がGoで書かれており、
      Go 1.19 以降はセキュリティ強化のため、
      相対パスやカレント由来で見つかった実行ファイルは暗黙に実行しないよう変更されたらしい。
      （そのとき典型的に出るのが “cannot run executable found relative to current directory” の類のエラー）。  
    - 6:`docker-credential-desktop`について調査した結果、`docker-credential-desktop`は呼び出し不要と判断した。
      - 理由
        - `docker-credential-desktop`の役割はDockerHubへのログイン上のセキュリティ向上。
          -`docker login`で入力するDockerHubのユーザー名やパスワードを平文で`config.json`に書かずに、  
          Windowsの資格情報マネージャーやmacOSのKeychainに保存・取得する。
          つまり「DockerDesktopとOSの認証ストアを橋渡しする専用の小さな exe」。
        - DockerHubへのログインは必須ではない。
          - ただし、DockerHubは匿名ユーザーに対してレート制限（pullの回数制限）をかけている。
    - 7:config.jsonの`"credsStore": "desktop",`を行ごと削除し保存
      - 修正後のconfig.json
        ```
        {
          "auths": {},
          "currentContext": "desktop-linux"
        }
        ```
    - 8:コマンドラインで下記実行。
      ```
      cd C:\Program Files\Docker\Docker\resources\bin
      docker compose -f "C:\dev\docker.compose.yml" up -d
      ```
    - 9:無事、エラーなくコンテナ起動できた。

      が
