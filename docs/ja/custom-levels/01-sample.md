# 1. まずサンプルを動かす

配布物の `custom-levels-kit.tar.gz` を展開すると、`custom-levels/examples/custom-level/` にサンプルのステージ `L101-portal-forbidden` が入っています。社内ポータルが 403 を返すようになった、という小さな障害です。

```
custom-level/
└── L101-portal-forbidden/
    ├── level.yaml                 ステージの定義
    └── hosts/
        └── web01/                 ホスト web01 用のファイル
            ├── setup.sh           正常な状態を作る
            ├── inject.sh          障害を起こす
            └── files/             コンテナにそのまま置くファイル
                └── usr/share/nginx/html/index.html
```

検証し、一覧に出し、遊ぶ。この 3 つのコマンドで一通りの流れを試せます。

```sh
tar -xzf custom-levels-kit.tar.gz
cd custom-levels/examples
sysfail level check custom-level                 # 書式の検証（Docker 不要）
sysfail --levels-dir custom-level list           # 一覧に「自作」として出る
sysfail --levels-dir custom-level 101            # 遊ぶ
```

`--levels-dir` を付けている間は、同梱のレベルは一覧に出ません。外せば元に戻ります。

Windows のランチャー（`sysfail-windows-amd64.exe`）では、`--levels-dir C:\Users\me\stages` のように Windows のパスも渡せます。ランチャーが WSL の中の `/mnt/c/Users/me/stages` に読み替えて渡します（WSL の自動マウント先を `/mnt` 以外に変えている場合は、Linux 側のパスを直接書いてください）。相対パスは使えません。
