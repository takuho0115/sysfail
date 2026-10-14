# Linux / WSL2

Linux（amd64・arm64）と WSL2 では、Release に添付の `install.sh` で入れるのがおすすめです。手でファイルを置く手順は、このページの後半にあります。

## install.sh で入れる（おすすめ）

install.sh は sysfail を取得して確かめ、自分のホームの中のディレクトリ（既定は `~/.local/bin`）に置きます。続けてゲーム用イメージのビルドまで行います。あらかじめ [cosign](https://docs.sigstore.dev/cosign/system_config/installation/)（2.x 以降）を入れておいてください。

取得、確認、中身を読む、実行の 4 つを分けて行います（安全のため、ダウンロードした内容をそのままシェルに渡す 1 行の手順は用意していません）。

```bash
TAG=v1.2.3   # 入れたいバージョン
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
curl -fsSLO "$BASE/install.sh"
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/SHA256SUMS.sigstore.json"

# SHA256SUMS の署名を確かめる（identity と issuer は一字一句このとおりに）
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS
# install.sh が改ざんされていないか確かめる
grep '  install.sh$' SHA256SUMS | sha256sum -c -
# 中身を読んでから実行する（既定の置き場所は ~/.local/bin。--prefix で変えられます）
less install.sh
sh install.sh
```

install.sh がすることは次のとおりです。

1. CPU（amd64・arm64）に合った sysfail を取得し、署名とハッシュの確認が両方通ったときだけ置きます。管理者の権限は使いません。
2. 続けて `sysfail doctor --build` を実行し、ゲーム用イメージをビルドしてから、動作環境を詳しく確かめます。ビルドの前に「初回のみ 5〜20 分・ディスク約 4GB」と表示します。docker のデータ領域の空きが 6GB 未満のときはビルドせず、空きを作る対処を表示します（WSL2 など空きを測れない環境では、目安だけ表示してビルドします）。

ビルドや動作環境の確認で問題が出ても、置いた sysfail コマンドはそのまま使えます。表示された対処を済ませてから、`sysfail doctor --build` をやり直してください。ビルドを後回しにするときは `sh install.sh --no-build` と実行します。その場合、ゲーム用イメージは最初にゲームを始めたときにビルドされます。

- 置き場所が PATH に入っていなければ、その旨を表示します。`~/.local/bin` などを PATH に足してください。
- `--prefix` に指定できるのは、すでに存在し、自分が持ち主で、シンボリックリンクではないディレクトリだけです。
- すでに `sysfail` という名前のファイルがある場合は、それが sysfail のときだけ置き換えます。sysfail 以外のファイルは置き換えずに止まります。

## 手でファイルを置く

install.sh を使わない場合の手順です。

```bash
TAG=v1.2.3          # 入れたいバージョン
ARCH=amd64          # ARM の端末では arm64
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
wget "$BASE/sysfail-linux-$ARCH" "$BASE/LICENSE.md" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
```

ダウンロードしたら、[配布物が本物か確かめる](../verify.md)の手順で確認してから置くことをおすすめします。

```bash
chmod +x "sysfail-linux-$ARCH"
"./sysfail-linux-$ARCH" --version    # "sysfail version $TAG" と表示されること
sudo mv "sysfail-linux-$ARCH" /usr/local/bin/sysfail
sysfail doctor --build               # ゲーム用イメージをビルドして、動作環境を確かめる（初回のみ 5〜20 分・約 4GB）
```
