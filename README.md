# sysfail

sysfail は、サーバー障害の対応をターミナルで練習するゲームです。レベルを始めると、あなたのパソコンの Docker の上に「どこかが壊れたサーバー群」が立ち上がります。ログや設定を調べて原因を突き止め、サービスを元に戻せばクリアです。

レベルは全部で 100 あります。インフラやサーバー運用を学びはじめた人が実戦の勘を養うのにも、経験者が腕試しをするのにも使えます。教材付きの教育版もあります。

このリポジトリは配布用で、ソースコードは公開していません。実行ファイルは [Releases](https://github.com/takuho0115/sysfail/releases) から入手してください。

- [動作環境](#動作環境)
- [インストール](#インストール)
- [はじめての遊び方](#はじめての遊び方)
- [教育版で遊ぶ](#教育版で遊ぶ)
- [困ったとき](#困ったとき)
- [配布物が本物か確かめる](#配布物が本物か確かめる)
- [不具合の報告・質問](#不具合の報告質問)
- [ライセンス](#ライセンス)

## 動作環境

次のソフトが必要です。

- **OS**: Linux（Ubuntu 22.04 以上、CentOS/RHEL 9 以上を推奨）、または WSL2（Windows の中で Linux を動かす Microsoft の仕組み）
- **Docker**: アプリを隔離された「コンテナ」の中で動かすソフトです。Engine 25.0.5 以上が必要です（28.0.0〜28.3.2 は使えません）
- **docker compose**: 複数のコンテナをまとめて起動する Docker の追加機能です。v2 プラグインの 2.24 以上が必要です（古い v1 には対応していません）
- **tmux**: ターミナルの画面を分割するソフトです。3.2 以上が必要です

Windows では、WSL2 の中で遊びます。Windows のターミナルから起動できるランチャー（`sysfail.exe`）も用意しています（[Windows](#windows) を参照）。WSL2 のカーネルによっては、`sysfail doctor` に「確認していないカーネル」と注記が出ます。これは作者が動作を確かめたカーネルの一覧に載っていないという意味です。

macOS（Apple Silicon）版は、まだ検証中のプレビューです（[macOS](#macosapple-silicon-検証中のプレビュー) を参照）。

### sysfail 自体をコンテナの中で動かす場合

sysfail をコンテナの中で動かす構成にも対応していますが、次の 2 点に注意してください。

- コンテナの中から外側の Docker を操作する構成で、外側の Docker が特権モード（privileged）で動いている場合、ゲーム内のコンテナからの抜け出しが外側のパソコンにまで届くおそれがあります。そのリスクを受け入れられる場合だけ使ってください。
- Docker をネットワーク越しに操作する場合（`DOCKER_HOST=tcp://...` など）は、必ず TLS による認証（`--tlsverify`）を有効にしてください。認証のない接続では起動しません。

## インストール

Releases の各バージョンには、次の 9 ファイルが添付されています。

| ファイル | 内容 |
|---|---|
| `sysfail-linux-amd64` / `sysfail-linux-arm64` | Linux 用の実行ファイル（一般的な PC は amd64、ARM の端末は arm64） |
| `sysfail-darwin-arm64` | macOS（Apple Silicon）用の実行ファイル（検証中のプレビュー） |
| `install.sh` | macOS 用のインストールスクリプト |
| `sysfail-windows-amd64.exe` | Windows 用ランチャー（同じバージョンの `sysfail-linux-amd64` と組で使います） |
| `THIRD_PARTY_NOTICES.txt` | 実行ファイルに組み込んだ他者のソフトウェアの著作権表示とライセンス文 |
| `LICENSE.md` | 利用条件 |
| `SHA256SUMS` | 各ファイルの SHA-256（ファイルが改ざんされていないかを調べるための値） |
| `SHA256SUMS.sigstore.json` | `SHA256SUMS` に対する作者の電子署名 |

公式の配布場所は `https://github.com/takuho0115/sysfail/releases` だけです。ほかのページや Release の本文に書かれたリンクからは入手しないでください。

### Linux / WSL2

```bash
TAG=v1.0.0          # 入れたいバージョン
ARCH=amd64          # ARM の端末では arm64
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
wget "$BASE/sysfail-linux-$ARCH" "$BASE/LICENSE.md" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
```

ダウンロードしたら、[配布物が本物か確かめる](#配布物が本物か確かめる)の手順で確認してから置くことをおすすめします。

```bash
chmod +x "sysfail-linux-$ARCH"
"./sysfail-linux-$ARCH" --version    # "sysfail version $TAG" と表示されること
sudo mv "sysfail-linux-$ARCH" /usr/local/bin/sysfail
```

### Windows

`sysfail.exe` は、Windows のターミナルから WSL2 の中の Linux 版 sysfail を起動するランチャーです。ゲーム本体は WSL2 の中で動きます。起動の前に WSL2・Linux 版・Docker の状態を調べ、問題があれば直し方を表示して止まります。

#### 必要なもの

- **Windows**: Windows 10 / 11（amd64）
- **WSL2**: PowerShell で `wsl --version` を実行してバージョンが表示されること（表示されない古い WSL は `wsl --update` で更新します）。使う Linux（ディストロ）が `wsl --list --verbose` で VERSION 2 になっていること
- **ディストロ**: Linux amd64。中に Docker Engine と docker compose v2 プラグインを入れます（バージョンの条件は[動作環境](#動作環境)と同じです）。tmux は要りません
- **Windows Terminal**（`wt.exe`）: 無くても遊べます。その場合はコンソール画面 2 つで開きます

Docker Desktop の WSL 連携でも、ディストロの中から `docker` コマンドが使えれば起動します。ただし動作の保証はなく、`doctor` が警告を出します。保証するのは、ディストロの中に直接入れた Docker Engine だけです。

#### 入れ方

`sysfail.exe` には、同じバージョンの `sysfail-linux-amd64` の SHA-256 が埋め込まれています。起動のたびにこれを照合し、一致しなければ起動しません。2 つは必ず同じバージョンのものを使い、片方を更新したらもう片方も入れ替えてください。

1. ディストロの中で 2 ファイルをダウンロードします。[配布物が本物か確かめる](#配布物が本物か確かめる)の手順（`ARCH=amd64`）で、2 ファイルとも `OK` になることを確かめてください。

   ```bash
   TAG=v1.0.0
   BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
   wget "$BASE/sysfail-linux-amd64" "$BASE/sysfail-windows-amd64.exe" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
   ```

2. ディストロの中に Linux 版を置きます。ランチャーは安全のため、Linux 版とその置き場所が root の持ち物で、ほかのユーザーが書き換えられないことを確かめます。`mv` だと持ち主が自分のままになるので、`install` を使ってください。

   ```bash
   sudo install -o root -g root -m 0755 sysfail-linux-amd64 /usr/local/bin/sysfail
   ```

   `/usr/local/bin/sysfail` 以外に置いた場合は、ランチャーに `--linux-path <絶対パス>` を付けます。

3. ランチャーを `sysfail.exe` という名前で、Windows の好きなフォルダに置きます。

   ```bash
   # 例: C:\Users\<ユーザー名>\sysfail に置く
   mkdir -p "/mnt/c/Users/<ユーザー名>/sysfail"
   cp sysfail-windows-amd64.exe "/mnt/c/Users/<ユーザー名>/sysfail/sysfail.exe"
   ```

   Windows のブラウザで直接ダウンロードした場合は、PowerShell で `Get-FileHash .\sysfail.exe -Algorithm SHA256` を実行し、その値が署名を確かめた `SHA256SUMS` の `sysfail-windows-amd64.exe` の行と一致することを確かめてください。

#### 使い方

`sysfail.exe` を置いたフォルダで、PowerShell から実行します。

```powershell
.\sysfail.exe doctor                    # 動作環境の確認（WSL の中で sysfail doctor を実行します）
.\sysfail.exe 3                         # レベル 3 を始める
.\sysfail.exe                           # レベル選択画面から始める
.\sysfail.exe --console 3               # Windows Terminal を使わず、コンソール画面 2 つで開く
.\sysfail.exe stop                      # ゲームを止めて環境を片付ける
.\sysfail.exe --distro Ubuntu-24.04 3   # 既定以外のディストロを使う
```

- ゲームは Windows Terminal の新しいウィンドウに、「司令画面」と「作業シェル」の 2 つのタブで開きます。
- `--console` を付けると、司令画面を今のコンソールで、作業シェルを新しいコンソールで開きます。
- `--distro` を省くと、WSL の既定のディストロ（`wsl --list --verbose` で `*` が付いたもの）を使います。既定のディストロが無い場合や WSL1 の場合は起動しません。
- `list`・`--edu` など、Linux 版のコマンドやオプションも使えます。ランチャー自身のオプション（`--distro`・`--console`・`--linux-path`）はコマンドより前に置きます。Linux 版のオプションを先頭に置くときは `--` で区切ります（例: `.\sysfail.exe -- --assets-dir /opt/sysfail-assets 3`）。
- 司令画面で `q` を押すと、環境を片付けて終わります。作業シェルのタブは残るので、手で閉じてください。途中で画面を閉じてしまった場合は、両方を閉じて `.\sysfail.exe stop` を実行してから起動し直してください。

#### 「Windows によって PC が保護されました」と出たら

`sysfail.exe` にはコード署名（Windows が発行元を確認するための署名）がありません。

- 初回の起動で SmartScreen の「Windows によって PC が保護されました」が出たら、「詳細情報」を押してから「実行」を押します。ただし、上の手順でハッシュを確かめたファイルだけを実行してください。
- セキュリティソフトが `sysfail.exe` を隔離・削除したり、0 バイトのファイルに置き換えたりすることがあります。その場合は置いたフォルダをスキャンの対象から外し、ハッシュを確かめ直したファイルを置き直してください。セキュリティソフトの窓口に誤検知として申告することもできます。

#### Windows での制限

- クリア状況やログは、WSL のディストロの既定ユーザーのホームに保存します。Windows 側には保存しません。ディストロを変えると、記録も別になります。
- 2 つの画面とも、ディストロの既定ユーザーで動きます。`--assets-dir` などに渡すパスも WSL 側のパスで書きます。
- ランチャーはシェルを通さずに Linux 版を起動するため、`~/.bashrc` などで設定した環境変数は効きません。
- tmux の一時離脱・再開（デタッチ・アタッチ）と、arm64 の Windows には対応していません。
- ゲーム環境の隔離は WSL2 の中の Docker が担います。ランチャーが Windows 側で隔離を追加するわけではありません。

### macOS（Apple Silicon）: 検証中のプレビュー

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

#### 既知の制限

- L052・L068（ネットワークの間を中継するレベル）は、macOS では起動前に止まります。
- 保証の条件（colima の設定・マウント・ネットワーク・ディスク・バージョンの組み合わせ）は doctor が確かめます。ただし実機での確認がまだなので、条件を満たしていても「検証中」と表示されます。
- Docker Desktop・OrbStack・Rancher Desktop は対象外です。
- Intel Mac 用の配布物はありません。

#### colima の推奨設定

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

#### 入手の方法

macOS 用の配布物は `sysfail-darwin-arm64` だけです。Apple の公証（Apple による安全確認）は受けていないので、本物かどうかは cosign と SHA256SUMS で確かめます（詳しくは[配布物が本物か確かめる](#配布物が本物か確かめる)）。

おすすめは次の 2 つです。どちらも、取得したファイルを自動で確かめます。Homebrew tap は v1.0.0 の公開と確認が済んでから用意します。それまでは「おすすめ 2」か「手で取得する場合」を使ってください。

##### おすすめ 1: Homebrew tap

```bash
brew install takuho0115/sysfail/sysfail
sysfail --version
```

公式の tap は `takuho0115/sysfail` だけです。似た名前の tap は使わないでください。Formula の sha256 を確かめたい場合は、`brew info --json=v2 takuho0115/sysfail/sysfail` の出力にある sha256 と、下の「手で取得する場合」の手順 1 で確かめた `SHA256SUMS` の `sysfail-darwin-arm64` の行を見比べます。tap で入れた sysfail が初回の起動で Gatekeeper に止められた場合は、何も操作せずに Issue で知らせてください。

##### おすすめ 2: install.sh

取得・確認・中身を読む・実行、を一つずつ分けて行います（ダウンロードした内容をそのままシェルに渡す 1 行の手順は用意していません）。

```bash
TAG=v1.0.0   # 入れたいバージョン
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

##### 手で取得する場合

上の 2 つが使えない場合の手順です。手順 1・2 を必ずこの順で行い、両方とも通ったときだけ手順 4 に進んでください。

```bash
TAG=v1.0.0
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

##### 動作報告のお願い

macOS で動かした結果は、うまくいった場合も含めて知らせていただけると助かります。[Issues](https://github.com/takuho0115/sysfail/issues/new/choose) の「macOS の動作報告」テンプレートに、macOS のバージョン、チップ、colima のバージョン、`sysfail doctor` の出力を貼ってください。インストールそのものの問題（起動しない、Gatekeeper に止められた など）は「macOS での導入・起動の問題」テンプレートを使ってください。

## はじめての遊び方

まず `sysfail doctor` で、動作環境がそろっているか確かめてください（[困ったとき](#困ったとき)も参照）。準備ができたら、レベル 1 から始めましょう。

```bash
sysfail 1        # レベル 1 から始める
sysfail          # レベル選択画面から始める
sysfail list     # レベルの一覧（クリア状況・評価・ベストタイム）
sysfail stop     # ゲームを止めて環境を片付ける
```

始めると、画面が左右 2 つに分かれます。

- **左: 作業用のシェル**。ゲームのために隔離された環境です。ここでコマンドを打って調べ、直します。
- **右: 司令画面**。いま起きている症状、クリアの条件、ヒント、経過時間が表示されます。

## 教育版で遊ぶ

`--edu` を付けると、教材の付いた教育版で遊べます。教育版でクリアした結果は、通常の記録（クリア状況・ベストタイム）には残りません。

```bash
sysfail list --edu                       # 教材のあるレベルを確かめる
sysfail 1 --edu                          # 教育版で始める
sysfail 6 --edu --edu-style socratic     # 教え方を選ぶ（レベル 6 以降）
```

- `--edu-style` で教え方を選べます。`guided` は一緒に進めてくれる伴走式、`socratic` は問いかけで考えを導く方式です。レベル 1〜5 は `guided` だけです。選んだ教え方は、次のレベルや次回の起動にも引き継がれます。
- `--edu-style` は `--edu` と一緒に指定します。
- 教材の無いレベルを `--edu` で始めると、その旨が表示され、通常の司令画面で続けるかどうかを選べます。

## 困ったとき

### まず `sysfail doctor` を実行する

`sysfail doctor` は、ゲームに必要なソフトや設定がそろっているかをまとめて調べるコマンドです。遊ぶ前に一度は実行してください。

```bash
sysfail doctor

# 出力例（必要なものがそろっている場合。一部抜粋）:
# 動作前提の確認
#   [OK] docker コマンド: あり
#   [OK] docker Engine の版: 27.0.0
#   [OK] docker compose: 2.27.0
#   [OK] tmux: 3.3a
#   [!!] コンテナランタイムの版: runc 1.2.8 / containerd 1.7.27
#        対処: コンテナランタイムの版を確認できませんでした（拒否表が空、または版を取得できません）。...
#   [OK] 動作保証の環境: Linux + Docker Engine
# 結果: 全ての前提を満たしています、警告 1 件
```

- `[OK]` は問題なし、`[!!]` は警告（ゲームは起動できます）、`[NG]` は起動に必要な条件を満たしていないことを表します。
- 問題がある行には「対処」として直し方が表示されます。
- 「コンテナランタイムの版」の行は、現在は常に警告になります。気にしなくてかまいません。
- スクリプトから使う場合、`[NG]` が 1 件でもあれば終了コードは 3、無ければ 0 です。

### よく出る警告と対処

**前のゲームのコンテナが残っている**: 以前に遊んだときのコンテナが残っていると警告が出ます。現在のバージョンで作られたものは、次にゲームを起動したときに自動で片付きます。古いバージョンで作られたものは、中身を確かめてから自分で削除してください。

**パソコンで待ち受けているポートがある**: パソコンのすべてのネットワーク（0.0.0.0、::）やゲートウェイのアドレスで待ち受けているサービスには、ゲームのコンテナから接続できてしまうおそれがあります。ゲームはこれを自動で設定しません。ファイアウォール（ufw、firewalld など）で遮断するか、各サービスの待ち受けを 127.0.0.1 に限ってください。WSL2 では、WSL の中（Linux 側）で設定します。

```bash
sudo ufw deny 3306/tcp  # 例: ポート 3306 を遮断
```

**bridge-nf の設定**: ネットワークの間を中継するレベルでは、Linux の設定 `net.bridge.bridge-nf-call-iptables` を 0 にする必要があります。遊ぶ間だけ変えて、終わったら戻してください。

```bash
BRIDGE_NF=$(cat /proc/sys/net/bridge/bridge-nf-call-iptables)
[ "$BRIDGE_NF" = "1" ] && sudo sysctl -w net.bridge.bridge-nf-call-iptables=0

# ゲーム終了後に戻す
sudo sysctl -w net.bridge.bridge-nf-call-iptables=$BRIDGE_NF
```

Kubernetes など、この設定が 1 であることを前提にするソフトが同じパソコンにある場合は、その動作に影響します。

**カーネルモジュールが読み込まれていない（WSL2 ではない Linux）**: 通信の遅延・帯域制限・パケット操作を再現する 8 つのレベル（L002・L032・L052・L059・L064・L068・L086・L100）は、Linux のカーネルモジュール（カーネルに後から足す機能）を使います。WSL2 では準備は要りません。WSL2 ではない Linux では読み込まれていないことが多く、そのままではこれらのレベルが「kernel modules not loaded」と表示して起動しません。ゲームはモジュールを自動では読み込みません。doctor が「対処」に表示する 2 つのコマンド（すぐ読み込むものと、再起動後も残すもの）を実行してください。表示されるコマンドはカーネルに合わせて変わります。

```bash
# 例: 読み込む（再起動すると消える）
sudo modprobe -a sch_netem sch_tbf sch_htb cls_u32 ip_tables iptable_filter xt_TCPMSS nf_tables nft_ct nft_numgen

# 例: 再起動後も残す（/etc/modules-load.d/sysfail.conf を上書きする）
printf '%s\n' sch_netem sch_tbf sch_htb cls_u32 ip_tables iptable_filter xt_TCPMSS nf_tables nft_ct nft_numgen | sudo tee /etc/modules-load.d/sysfail.conf
```

modprobe が `nft_meta` や `nft_exthdr` について「Module not found」と表示したら、その 2 つを除いて実行し直してください（カーネルによっては `nf_tables` に含まれています）。

## 配布物が本物か確かめる

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

## 不具合の報告・質問

不具合の報告と質問は、このリポジトリで受け付けています。

- **不具合の報告**: [Issues](https://github.com/takuho0115/sysfail/issues/new/choose) から、テンプレートに沿って OS、`sysfail --version` の出力、`sysfail doctor` の出力、再現の手順を書いてください。
- **質問・要望**: [Discussions](https://github.com/takuho0115/sysfail/discussions) へどうぞ。

トークンやパスワードなどの秘密の情報は貼らないでください。ユーザー名やホスト名は伏せてかまいません。

## ライセンス

sysfail の著作権は作者が持っています（All rights reserved）。

- 入手できるのは、このリポジトリ（公式の配布場所）からだけです。
- 入手した sysfail やその一部を、ほかの人に配ったり、別の場所で公開したりすること（再配布）は、有料・無料を問わず禁止しています。

正式な条文は [LICENSE.md](LICENSE.md) にあります。

実行ファイルに組み込んだ他者のソフトウェアの著作権表示とライセンス文は、各 Release に添付の `THIRD_PARTY_NOTICES.txt` にあります。
