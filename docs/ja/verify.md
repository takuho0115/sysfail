# 配布物が本物か確かめる（配布物の検証）

ダウンロードしたファイルが作者の作った本物で、途中で書き換えられていないことを確かめる手順です。必須ではありませんが、実行する前に行うことを強くおすすめします。

使う道具は 2 つです。

- **SHA256SUMS**: 各ファイルの SHA-256（ファイルの中身から計算する指紋のような値）の一覧です。ファイルが 1 バイトでも変わると値が変わるので、改ざんに気づけます。
- **cosign**: 電子署名を確かめるツールです。2.x 以降を使います。`SHA256SUMS` そのものが作者の公開したものかを、署名で確かめます。

ハッシュの確認だけでは、`SHA256SUMS` ごと差し替えられた場合に気づけません。そのため、次の 3 つをこの順にすべて行ってください。

```bash
# 1. SHA256SUMS の署名を確かめる（identity と issuer は一字一句このとおりに指定する）
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS

# 2. 署名を確かめた SHA256SUMS で、実行ファイルが改ざんされていないか確かめる
sha256sum -c SHA256SUMS --ignore-missing

# 3. バージョンを確かめる（"sysfail version $TAG" と表示されること）
chmod +x "sysfail-linux-$ARCH"
"./sysfail-linux-$ARCH" --version
```

`--certificate-identity` には、配布ファイルを作って署名した自動ビルドの場所を書きます。これは非公開のソースコード用リポジトリ `takuho0115/sysfail-sim-game` の `release.yaml` で、このリポジトリ（`takuho0115/sysfail`）ではありません。ソースコード用のリポジトリは開けませんが、署名の確認には中身は要りません。この値をこのリポジトリの名前に書き換えたり、正規表現で条件をゆるめたりしないでください。

GitHub の artifact attestation（ビルドの来歴を証明する GitHub の機能）は提供していません。どのコミットから、どの自動ビルドで作られたかは、手順 1 の署名の証明書に記録されています。この記録は、誰でも見られる Sigstore の公開ログ（Rekor）にも残っています。
