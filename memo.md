## p43
- dockerのインストール先：ホストとなるwindows  
- Docker DesktopをWindowsにインストールする際、  
  セットアップ画面に表示される  
　「Allow Windows Containers to be used with this installation」  
　というチェックボックスへの対応:空のままでよい。  
    - 参考：https://nakaterux.hatenablog.com/entry/2025/08/18/000230

## p46
- docker-compose.yml
  - textの内容の修正が必要
    - ref:
## p47
- dockerコマンドが見つからない。
- `docker compose up -d` は失敗する。
  環境変数PATHにdockerのbinを登録する下りがこの本にはない。
