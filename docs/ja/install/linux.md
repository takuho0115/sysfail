# Linux / WSL2

```bash
TAG=v1.1.0          # 入れたいバージョン
ARCH=amd64          # ARM の端末では arm64
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
wget "$BASE/sysfail-linux-$ARCH" "$BASE/LICENSE.md" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
```

ダウンロードしたら、[配布物が本物か確かめる](../verify.md)の手順で確認してから置くことをおすすめします。

```bash
chmod +x "sysfail-linux-$ARCH"
"./sysfail-linux-$ARCH" --version    # "sysfail version $TAG" と表示されること
sudo mv "sysfail-linux-$ARCH" /usr/local/bin/sysfail
```
