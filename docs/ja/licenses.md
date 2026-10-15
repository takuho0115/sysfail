# ライセンス

## sysfail 本体

sysfail の著作権は作者が持っています（All rights reserved）。再配布は禁止しています。正式な条文は、リポジトリの [LICENSE.md](https://github.com/takuho0115/sysfail/blob/main/LICENSE.md) にあります。

実行ファイルに組み込んだ他者のソフトウェアの著作権表示とライセンス文は、各 Release に添付の `THIRD_PARTY_NOTICES.txt` にあります。

## ゲーム用イメージに入るソフトウェア

sysfail は、初回の起動時か `sysfail doctor --build` の実行時に、次のゲーム用イメージを ghcr.io から取得します。

- `ghcr.io/takuho0115/sysfail-el9`
- `ghcr.io/takuho0115/sysfail-toolbox`

どちらも Rocky Linux 9 の公式イメージ（`rockylinux/rockylinux:9`）をもとにしていて、Rocky Linux 9 のパッケージが入っています。その中には、GPL や LGPL のように、ソースコードの提供を条件とするライセンスのものもあります。各パッケージのライセンスは、それぞれのパッケージに付いているライセンス文に従います。

### ソースの入手先

これらのパッケージのソースは、Rocky Linux が公開しているソース RPM（`.src.rpm`）から入手できます。

| 対象 | 場所 |
|---|---|
| 現在の Rocky Linux 9 | <https://dl.rockylinux.org/pub/rocky/9/> の `BaseOS/source/tree/Packages/` と `AppStream/source/tree/Packages/` |
| 以前のマイナーバージョン（9.0 から） | <https://dl.rockylinux.org/vault/rocky/> の `9.<番号>/` の下（構成は現在のものと同じ） |

### イメージに入っているパッケージを調べる

イメージに入っているパッケージとそのバージョンは、次のコマンドで一覧表示できます。

```bash
docker run --rm --entrypoint rpm ghcr.io/takuho0115/sysfail-el9@<digest> -qa
```

`<digest>` には、`docker images --digests` で表示される値を入れてください。toolbox のイメージを調べるときは、イメージ名を `ghcr.io/takuho0115/sysfail-toolbox@<digest>` に替えます。パッケージ名とバージョンが分かれば、上の入手先から対応するソース RPM を探せます。上のコマンドの `-qa` を `-qa --qf '%{SOURCERPM}\n'` に替えて実行すると、ソース RPM のファイル名が直接表示されます。
