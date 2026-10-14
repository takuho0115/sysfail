# macOS（Apple Silicon）: 検証中のプレビュー

> macOS 版は検証中のプレビューで、動作の保証はありません（保証を始めるときはリリースノートでお知らせします）。
>
> 保証を予定しているのは、Apple Silicon の Mac で colima（Mac で Docker を動かすためのソフト）に sysfail 専用の設定を作り、ネットワークの間を中継する 2 レベル（L052・L068）を除いた 98 レベルを遊ぶ場合だけです。それ以外の構成（Docker Desktop・OrbStack・Rancher Desktop、Intel Mac、下の推奨設定を満たさない colima、L052・L068）でも警告付きで起動しますが、保証はしません。

現時点で確かめたことと、まだ確かめていないことは次のとおりです。

| 確認済み | 未確認 |
|---|---|
| macOS 用のビルドと、自動で付く簡易署名（ad-hoc 署名） | colima の上での実際のプレイ（98 レベルを通した確認、隔離のテスト） |
| macOS（GitHub の Apple Silicon 環境）での単体テスト | 実機の Mac での Gatekeeper（macOS がダウンロードしたアプリを止める仕組み）の挙動 |
| install.sh の検証の流れ（署名とハッシュの確認が通らなければ何も置かない） | Terminal.app・iTerm2 での長時間の操作 |

実機での確認が済むまで、`sysfail doctor` は「保証範囲: 検証中」と、満たしていない条件を表示します。

## 既知の制限

- L052・L068（ネットワークの間を中継するレベル）は、macOS では起動前に止まります。
- 保証の条件（colima の設定・マウント・ネットワーク・ディスク・バージョンの組み合わせ）は doctor が確かめます。ただし実機での確認がまだなので、条件を満たしていても「検証中」と表示されます。
- Docker Desktop・OrbStack・Rancher Desktop は対象外です。
- Intel Mac 用の配布物はありません。

## colima の推奨設定

sysfail 専用のプロファイル（colima の設定のまとまり）を作ります。既定のプロファイルは使い回さないでください。

```bash
brew install colima docker
mkdir -p "$HOME/.sysfail-colima-empty"
colima start -p sysfail --arch aarch64 --vm-type vz --cpu 4 --memory 8 --disk 60 \
  --mount "$HOME/.sysfail-colima-empty"
docker context use colima-sysfail
sysfail doctor
```

- 共有するフォルダ（マウント）は、空の専用フォルダを読み取り専用で 1 つだけにします。ホームフォルダ全体の共有は、読み取り専用でも保証の対象外です。
- `--network-address` は付けません。`--disk` は Mac の空き容量より小さくします。

## 入手の方法

macOS 用の配布物は `sysfail-darwin-arm64` だけです。Apple の公証（Apple による安全確認）は受けていないので、本物かどうかは cosign と SHA256SUMS で確かめます（詳しくは[配布物が本物か確かめる](../verify.md)）。

おすすめは次の 2 つです。どちらも、取得したファイルを自動で確かめます。Homebrew tap は準備中です。それまでは「おすすめ 2」か「手で取得する場合」を使ってください。

### おすすめ 1: Homebrew tap

```bash
brew install takuho0115/sysfail/sysfail
sysfail --version
```

公式の tap は `takuho0115/sysfail` だけです。似た名前の tap は使わないでください。Formula の sha256 を確かめたい場合は、`brew info --json=v2 takuho0115/sysfail/sysfail` の出力にある sha256 と、下の「手で取得する場合」の手順 1 で確かめた `SHA256SUMS` の `sysfail-darwin-arm64` の行を見比べます。tap で入れた sysfail が初回の起動で Gatekeeper に止められた場合は、何も操作せずに Issue で知らせてください。

### おすすめ 2: install.sh

取得、確認、中身を読む、実行の 4 つを分けて行います（ダウンロードした内容をそのままシェルに渡す 1 行の手順は用意していません）。

```bash
TAG=v1.2.3   # 入れたいバージョン
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
curl -fsSLO "$BASE/install.sh"
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/SHA256SUMS.sigstore.json"

# SHA256SUMS の署名を確かめる（cosign は brew install cosign で入ります。identity と issuer は一字一句このとおりに）
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS
# install.sh が改ざんされていないか確かめる
grep '  install.sh$' SHA256SUMS | shasum -a 256 -c -
# 中身を読んでから実行する（既定の置き場所は ~/.local/bin。--prefix で変えられます）
less install.sh
sh install.sh
```

install.sh は、署名とハッシュの確認が両方通ったときだけ sysfail を置きます。管理者の権限は使いません。`--prefix` に指定できるのは、すでに存在し、自分が持ち主で、シンボリックリンクではないフォルダだけです。すでに `sysfail` という名前のファイルがある場合は、それが sysfail のときだけ置き換えます。

置いたあとは続けて `sysfail doctor --build` を実行し、ゲーム用イメージをビルドしてから動作環境を確かめます（初回のみ 5〜20 分・約 4GB）。ビルドや確認で問題が出ても、置いた sysfail コマンドはそのまま使えます。表示された対処を済ませてから `sysfail doctor --build` をやり直してください。ビルドを後回しにするときは `sh install.sh --no-build` と実行します。

### 手で取得する場合

上の 2 つが使えない場合の手順です。手順 1・2 を必ずこの順で行い、両方とも通ったときだけ手順 4 に進んでください。

```bash
TAG=v1.2.3
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
curl -fsSLO "$BASE/sysfail-darwin-arm64"
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/SHA256SUMS.sigstore.json"

# 1.（必須）SHA256SUMS の署名を確かめる
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS

# 2.（必須）署名を確かめた SHA256SUMS で、実行ファイルが改ざんされていないか確かめる
grep '  sysfail-darwin-arm64$' SHA256SUMS | shasum -a 256 -c -

# 3.（参考）簡易署名であること（Signature=adhoc と出る）。作者の証明にはならないので、本物かどうかの判断は 1・2 で行う
codesign -dv ./sysfail-darwin-arm64

# 4. 1・2 が通った場合だけ: そのファイル 1 つだけ、ダウンロードの印を外して確かめる
mv sysfail-darwin-arm64 sysfail
chmod +x sysfail
xattr -d com.apple.quarantine sysfail
./sysfail --version
```

注意:

- 手順 4 の操作は、手順 1・2 を通した公式の sysfail のファイル 1 つにだけ行ってください。確かめていないファイル、sysfail 以外のファイル、Issue・チャット・ほかのサイトで指示されたファイルには行わないでください。フォルダごとまとめて行うのもやめてください。
- Gatekeeper 全体を無効にしたり、システム設定でずっと許可したりはしないでください。

### 動作報告のお願い

macOS で動かした結果は、うまくいった場合も含めて知らせていただけると助かります。[Issues](https://github.com/takuho0115/sysfail/issues/new/choose) の「macOS の動作報告」テンプレートに、macOS のバージョン、チップ、colima のバージョン、`sysfail doctor` の出力を貼ってください。インストールそのものの問題（起動しない、Gatekeeper に止められた など）は「macOS での導入・起動の問題」テンプレートを使ってください。
