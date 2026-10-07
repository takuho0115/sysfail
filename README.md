# sysfail

ターミナルで遊ぶ、インフラ障害対応の訓練ゲームです。手元の Docker の上に壊れたサーバー群が立ち上がるので、ログや状態を調べて原因を突き止め、復旧させます。レベルは全部で 100 あります。

このリポジトリは配布用です。ソースコードは公開していません。実行ファイルは [Releases](https://github.com/takuho0115/sysfail/releases) から入手してください。

- [システム要件](#システム要件)
- [インストール](#インストール)
- [前提確認（sysfail doctor）](#前提確認sysfail-doctor)
- [遊び方](#遊び方)
- [教育版で遊ぶ（--edu）](#教育版で遊ぶ--edu)
- [Windows で遊ぶ（WSL2 ランチャー）](#windows-で遊ぶwsl2-ランチャー)
- [macOS（Apple Silicon）: 検証中のプレビュー](#macosapple-silicon-検証中のプレビュー)
- [不具合の報告・質問](#不具合の報告質問)
- [ライセンス](#ライセンス)

## システム要件

- **OS**: Linux（Ubuntu 22.04 以上、CentOS/RHEL 9 以上を推奨）または WSL2
- **Docker**: Engine 25.0.5 以上（28.0.0〜28.3.2 は除く）
- **docker compose**: v2 プラグイン 2.24 以上（v1 には対応していません）
- **tmux**: 3.2 以上

WSL2 でも遊べます。確認済みのカーネルは `sysfail doctor` の表に載っているものだけで、載っていないカーネルでは「確認していないカーネル」と注記が出ます。Windows のターミナルから起動するランチャー（`sysfail.exe`）もあります（[Windows で遊ぶ](#windows-で遊ぶwsl2-ランチャー)）。

sysfail をコンテナの中で動かす構成（dind・DooD）にも対応していますが、次の点に気をつけてください。

- DooD で外側の docker デーモンが privileged で動いている場合、ゲームのコンテナからの脱出が外側のホストまで届くおそれがあります。そのリスクを受け入れられる場合だけ使ってください。
- docker を TCP ポートで公開する場合（`DOCKER_HOST=tcp://...` など）は、必ず TLS の認証（`--tlsverify`）を有効にしてください。認証なしの TCP API は使えません。

## インストール

### 配布物

配布物は Linux 用（linux/amd64・linux/arm64）の実行ファイルと、Windows 用のランチャー（windows/amd64）です。macOS（Apple Silicon）用は検証中のプレビューとして配布しています。

各 Release には次の 9 ファイルが添付されています。

| ファイル | 内容 |
|---|---|
| `sysfail-linux-amd64` / `sysfail-linux-arm64` | 実行ファイル |
| `sysfail-darwin-arm64` | macOS（Apple Silicon）用の実行ファイル（検証中のプレビュー。Apple の公証なし、ad-hoc 署名） |
| `install.sh` | macOS 用のインストールスクリプト（検証してから置く） |
| `sysfail-windows-amd64.exe` | Windows 用ランチャー（同じ Release の `sysfail-linux-amd64` と組で使う） |
| `THIRD_PARTY_NOTICES.txt` | 実行ファイルに組み込まれた第三者のソフトウェアの著作権表示とライセンス文 |
| `LICENSE.md` | 利用条件 |
| `SHA256SUMS` | 実行ファイル・ランチャー・`install.sh`・`THIRD_PARTY_NOTICES.txt`・`LICENSE.md` の SHA-256 |
| `SHA256SUMS.sigstore.json` | `SHA256SUMS` の署名（Sigstore のキーレス署名） |

配布元は `https://github.com/takuho0115/sysfail/releases` だけです。Release の本文やほかのページにあるリンクを配布元として扱わないでください。

```bash
TAG=v1.0.0          # 入れたいバージョン
ARCH=amd64          # arm64 の端末では arm64
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
wget "$BASE/sysfail-linux-$ARCH" "$BASE/LICENSE.md" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
```

### 配布物の検証（必須）

次の (a)〜(c) を、この順にすべて行ってください。(b) のチェックサムだけでは、`SHA256SUMS` ごと差し替えられた場合に気づけません。

(a) の identity には、署名したワークフローの場所を書きます。これは非公開のソースリポジトリ `takuho0115/sysfail-sim-game` の `release.yaml` で、配布元の `takuho0115/sysfail` ではありません。ソースリポジトリは開けませんが、署名の確認に中身は要りません。identity を配布元の名前に書き換えたり、正規表現で緩めたりしないでください。

```bash
# (a) SHA256SUMS の署名を確かめる（cosign 2.x 以降。identity と issuer は完全一致で指定する）
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS

# (b) 署名を確かめた SHA256SUMS で実行ファイルを確かめる
sha256sum -c SHA256SUMS --ignore-missing

# (c) バージョンを確かめる（"sysfail version $TAG" と表示されること）
chmod +x "sysfail-linux-$ARCH"
"./sysfail-linux-$ARCH" --version

sudo mv "sysfail-linux-$ARCH" /usr/local/bin/sysfail
```

GitHub の artifact attestation（来歴）は提供していません。どのコミットとワークフローの実行から作られたかは (a) の署名の証明書に記録され、Sigstore の公開の透明性ログ（Rekor）に残っています。

## 前提確認（sysfail doctor）

遊ぶ前に、必ず `sysfail doctor` で動作環境を確かめてください。

```bash
sysfail doctor

# 出力例（前提を満たす場合。抜粋）:
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

`[NG]` が 1 件でもあれば終了コードは 3、無ければ 0 です。`[!!]` は警告で、起動はできます。コンテナランタイムの版の行は、いまは常に警告になります。

よく出る警告と対処は次のとおりです。

**残骸コンテナ**: 以前のゲームのコンテナが残っていると警告が出ます。現行版の残骸は、次にゲームを起動したときに自動で片付きます。旧版のラベルが付いた残骸は、自分で確かめてから削除してください。

**ホストの待ち受けポート**: ホストの全インターフェイス（0.0.0.0、::）やゲートウェイのアドレスで待ち受けているポートには、ゲームのコンテナから届くおそれがあります。ゲームは自動で設定しないので、ファイアウォール（ufw、firewalld など）で遮断するか、各サービスの待ち受けを 127.0.0.1 に限ってください。WSL2 では WSL の中（Linux 側）で設定します。

```bash
sudo ufw deny 3306/tcp  # 例: ポート 3306 を遮断
```

**bridge-nf**: ネットワークの間を中継するレベル（routed）では `net.bridge.bridge-nf-call-iptables=0` が必要です。遊ぶ間だけ変えて、終わったら戻してください。

```bash
BRIDGE_NF=$(cat /proc/sys/net/bridge/bridge-nf-call-iptables)
[ "$BRIDGE_NF" = "1" ] && sudo sysctl -w net.bridge.bridge-nf-call-iptables=0

# ゲーム終了後に戻す
sudo sysctl -w net.bridge.bridge-nf-call-iptables=$BRIDGE_NF
```

Kubernetes など、1 を前提とするソフトが同じホストにある場合は、その動作に影響します。

**カーネルモジュール（WSL2 ではない Linux）**: 通信の遅延・帯域制限・パケット操作を使う 8 つのレベル（L002・L032・L052・L059・L064・L068・L086・L100）は、カーネルモジュールを使います。WSL2 では準備は要りません。WSL2 ではない Linux では読み込まれていないことが多く、そのままではこれらのレベルが「kernel modules not loaded」で起動しません。ゲームはモジュールを自分では読み込まないので、doctor が「対処」に出す 2 つのコマンド（いま読み込むものと、再起動後も残すもの）を実行してください。表示はカーネルに合わせて変わります。

```bash
# 例: 読み込む（再起動すると消える）
sudo modprobe -a sch_netem sch_tbf sch_htb cls_u32 ip_tables iptable_filter xt_TCPMSS nf_tables nft_ct nft_numgen

# 例: 再起動後も残す（/etc/modules-load.d/sysfail.conf を上書きする）
printf '%s\n' sch_netem sch_tbf sch_htb cls_u32 ip_tables iptable_filter xt_TCPMSS nf_tables nft_ct nft_numgen | sudo tee /etc/modules-load.d/sysfail.conf
```

modprobe が `nft_meta` や `nft_exthdr` について「Module not found」と出したら、その 2 つを除いて実行し直してください（`nf_tables` に含まれているカーネルがあります）。

## 遊び方

```bash
sysfail 1        # レベル 1 から始める
sysfail          # レベル選択画面から始める
sysfail list     # レベルの一覧（クリア状況・評価・ベストタイム）
sysfail stop     # ゲームを止めて環境を片付ける
```

画面は左右 2 つに分かれます。左が隔離されたシェル、右が症状・成功条件・ヒント・経過時間を表示する司令画面です。

## 教育版で遊ぶ（--edu）

`--edu` を付けると、教材の付いた教育版で遊べます。教育版でクリアした結果は、通常の記録（クリア状況・ベストタイム）には残りません。

```bash
sysfail list --edu                       # 教材のあるレベルを確かめる
sysfail 1 --edu                          # 教育版で始める
sysfail 6 --edu --edu-style socratic     # 型を選ぶ（L006 以降）
```

- `--edu-style` には `guided`（伴走式）か `socratic` を指定します。L001〜L005 は `guided` だけです。選んだ型は、次のレベルや次回の起動にも引き継がれます。
- `--edu-style` は `--edu` と一緒に指定します。
- 教材の無いレベルを `--edu` で始めると、その旨が表示され、通常の司令画面で続けるかを選べます。

## Windows で遊ぶ（WSL2 ランチャー）

`sysfail.exe` は、Windows のターミナルから WSL2 の中の Linux 版 `sysfail` を起動するランチャーです。ゲーム本体は WSL2 の中で動きます。起動前に WSL2・Linux 版・Docker を検査し、問題があれば対処を表示して、ゲームを起動しません。

### 前提

- **Windows**: Windows 10 / 11（amd64）
- **WSL2**: PowerShell で `wsl --version` を実行してバージョンが表示されること（表示されない古い WSL は `wsl --update` で更新）。使うディストロが `wsl --list --verbose` で VERSION 2 になっていること
- **ディストロ**: Linux amd64。中に Docker Engine と docker compose v2 プラグインを入れます（バージョンの条件は[システム要件](#システム要件)と同じ）。tmux は要りません
- **Windows Terminal**（`wt.exe`）: 無い場合は、`--console` を付けたときと同じく、コンソール画面 2 つで開きます

Docker Desktop の WSL 統合でも、ディストロの中から `docker` に接続できれば起動します。ただし保証はせず、`doctor` が警告を出します。保証するのはディストロ内の Docker Engine だけです。

### 入れ方

`sysfail.exe` には、同じ Release の `sysfail-linux-amd64` の SHA-256 が埋め込まれています。起動のたびに Linux 版のハッシュを照合し、合わなければ起動しません。2 つは必ず同じ Release のものを使い、片方を更新したらもう片方も入れ替えてください。

1. ディストロの中で 2 ファイルをダウンロードし、[配布物の検証](#配布物の検証必須)の (a)〜(c) を行います（`ARCH=amd64`）。(b) で 2 ファイルとも `OK` になることを確かめてください。

   ```bash
   TAG=v1.0.0
   BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
   wget "$BASE/sysfail-linux-amd64" "$BASE/sysfail-windows-amd64.exe" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
   ```

2. ディストロの中で Linux 版を置きます。ランチャーは、Linux 版とその親ディレクトリが root の所有で、ほかのユーザーが書き込めないことを確かめます。所有者が自分のままになる `mv` ではなく、`install` を使ってください。

   ```bash
   sudo install -o root -g root -m 0755 sysfail-linux-amd64 /usr/local/bin/sysfail
   ```

   `/usr/local/bin/sysfail` 以外に置いた場合は、ランチャーに `--linux-path <絶対パス>` を付けます。

3. ランチャーを `sysfail.exe` という名前で、Windows の任意のフォルダに置きます。

   ```bash
   # 例: C:\Users\<ユーザー名>\sysfail に置く
   mkdir -p "/mnt/c/Users/<ユーザー名>/sysfail"
   cp sysfail-windows-amd64.exe "/mnt/c/Users/<ユーザー名>/sysfail/sysfail.exe"
   ```

   Windows のブラウザで直接ダウンロードした場合は、PowerShell の `Get-FileHash .\sysfail.exe -Algorithm SHA256` の値が、(a) で署名を確かめた `SHA256SUMS` の `sysfail-windows-amd64.exe` の行と一致することを確かめてください。

### 使い方

`sysfail.exe` を置いたフォルダで、PowerShell から実行します。

```powershell
.\sysfail.exe doctor                    # 前提確認（WSL の中で sysfail doctor を実行する）
.\sysfail.exe 3                         # レベル 3 を始める
.\sysfail.exe                           # レベル選択画面から始める
.\sysfail.exe --console 3               # Windows Terminal を使わず、コンソール画面 2 つで開く
.\sysfail.exe stop                      # ゲームを止めて環境を片付ける
.\sysfail.exe --distro Ubuntu-24.04 3   # 既定以外のディストロを使う
```

- ゲームは Windows Terminal の新しいウィンドウに、「司令画面」と「作業シェル」の 2 つのタブで開きます。
- `--console` を付けると、司令画面を今のコンソールで、作業シェルを新しいコンソールで開きます。
- `--distro` を省くと、WSL の既定のディストロ（`wsl --list --verbose` で `*` の付いたもの）を使います。既定のディストロが無い場合や WSL1 の場合は起動しません。
- `list`・`--edu` などほかのコマンドやオプションも使えます。ランチャーのオプション（`--distro`・`--console`・`--linux-path`）はコマンドより前に置き、Linux 版のフラグを先頭に置くときは `--` で区切ります（例: `.\sysfail.exe -- --assets-dir /opt/sysfail-assets 3`）。
- 司令画面で `q` を押すと、環境を片付けて終わります。作業シェルのタブは残るので、手で閉じてください。途中で画面を閉じてしまった場合は、両方を閉じて `.\sysfail.exe stop` を実行してから起動し直してください。

### 署名していない exe について

`sysfail.exe` にはコード署名がありません。

- 初回の起動で SmartScreen の「Windows によって PC が保護されました」が出たら、「詳細情報」を押してから「実行」を押します。実行するのは、上の手順でハッシュを確かめたものだけにしてください。
- セキュリティソフトが `sysfail.exe` を隔離・削除したり、0 バイトのファイルに置き換えたりすることがあります。その場合は置いたフォルダをスキャンの対象から外し、ハッシュを確かめ直したファイルを置き直してください。各社の窓口に誤検知として申告することもできます。

### 制約

- クリア状況やログは、WSL 側の対象ディストロの既定ユーザーのホームに保存します。Windows 側には保存しません。ディストロを変えると記録も別になります。
- 両方の画面とも、対象ディストロの既定ユーザーで動きます。`--assets-dir` などに渡すパスも WSL 側のパスです。
- ランチャーはシェルを通さずに Linux 版を起動するため、`~/.bashrc` などで設定した環境変数は効きません。
- tmux のデタッチ・再開と、arm64 の Windows には対応していません。
- ゲーム環境の隔離は WSL2 の中の Docker によるもので、ランチャーが Windows 側で隔離を足すわけではありません。

## macOS（Apple Silicon）: 検証中のプレビュー

> macOS は検証中のプレビューで、保証の対象外です（保証を始めるときはリリースノートで告知します）。
>
> 保証の予定がある構成は、Apple Silicon の Mac で colima（sysfail 専用のプロファイル）を使い、routed を使わない 98 レベルを遊ぶ場合だけです。それ以外（Docker Desktop・OrbStack・Rancher Desktop、Intel Mac、条件を満たさない colima、L052・L068）は警告付きで起動しますが、保証しません。

確かめた範囲と、まだ確かめていない範囲は次のとおりです。

| 確認済み | 未確認 |
|---|---|
| darwin/arm64 のビルドと、Go のリンカが付ける ad-hoc 署名 | colima の上での実際のプレイ（98 レベルの通しの検証、隔離テスト） |
| macOS（GitHub の Apple Silicon のランナー）での単体テスト | 実機の Mac での Gatekeeper の挙動（取得の経路ごとの違い） |
| install.sh の検証の流れ（cosign と SHA256SUMS が通らなければ何も置かない） | Terminal.app・iTerm2 での長時間の操作 |

実機での確認が済むまで、`sysfail doctor` は「保証範囲: 検証中」と、満たしていない条件を表示します。

### 既知の制限

- L052・L068（ネットワークの間を中継するレベル）は、macOS では起動前に止まります（`platform-routed`）。
- 保証の条件（colima のプロファイル・マウント・ネットワーク・ディスク・バージョンの組み合わせ）は doctor が確かめます。実機での合格記録が無い今は、条件を満たしても「検証中」のままです（`macos-version-unverified`）。
- Docker Desktop・OrbStack・Rancher Desktop は対象外です。
- Intel Mac 用の配布物はありません。

### colima の推奨設定

sysfail 専用のプロファイルを作ります。既定のプロファイルは流用しないでください。

```bash
brew install colima docker
mkdir -p "$HOME/.sysfail-colima-empty"
colima start -p sysfail --arch aarch64 --vm-type vz --cpu 4 --memory 8 --disk 60 \
  --mount "$HOME/.sysfail-colima-empty"
docker context use colima-sysfail
sysfail doctor
```

- マウントは、空の専用ディレクトリを読み取り専用で 1 つだけにします。ホーム全体のマウントは、読み取り専用でも保証外です。
- `--network-address` は付けません。`--disk` は Mac の空き容量より小さくします。

### 入手の方法

macOS 用の配布物は `sysfail-darwin-arm64` だけです。Apple の公証は受けていない（ad-hoc 署名）ので、本物かどうかは cosign と SHA256SUMS で確かめます。

次の 2 つの方法を勧めます。どちらも取得した物を自動で検証します。Homebrew tap は v1.0.0 の公開と検証が済んでから用意するので、それまでは推奨 2 か「手で取得する場合」を使ってください。

#### 推奨 1: Homebrew tap

```bash
brew install takuho0115/sysfail/sysfail
sysfail --version
```

正規の tap は `takuho0115/sysfail` だけです。似た名前の tap は使わないでください。Formula の sha256 を確かめたい場合は、`brew info --json=v2 takuho0115/sysfail/sysfail` の出力の sha256 と、下の「手で取得する場合」の手順 1 で検証した `SHA256SUMS` の `sysfail-darwin-arm64` の行を見比べます。tap で入れた sysfail が初回の起動で Gatekeeper に止められた場合は、何も操作せずに Issue で知らせてください。

#### 推奨 2: install.sh

取得、検証、中身の確認、実行を分けて行います（取得した物をそのままシェルに渡す 1 行の手順は用意していません）。

```bash
TAG=v1.0.0   # 入れたいバージョン
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
curl -fsSLO "$BASE/install.sh"
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/SHA256SUMS.sigstore.json"

# SHA256SUMS の署名を確かめる（brew install cosign。identity と issuer は完全一致）
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS
# install.sh を照らし合わせる
grep '  install.sh$' SHA256SUMS | shasum -a 256 -c -
# 中身を読んでから実行する（既定の置き場所は ~/.local/bin。--prefix で変えられる）
less install.sh
sh install.sh
```

install.sh は、cosign と SHA256SUMS の検証が両方通ったときだけ sysfail を置きます。管理者の権限は使いません。`--prefix` には、既にあり、自分が所有し、シンボリックリンクでないディレクトリだけを指定できます。既にある `sysfail` は、それが sysfail のときだけ置き換えます。

#### 手で取得する場合

上の 2 つが使えない場合の手順です。手順 1・2 を必ずこの順で行い、両方が通ったときだけ手順 4 に進んでください。

```bash
TAG=v1.0.0
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
curl -fsSLO "$BASE/sysfail-darwin-arm64"
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/SHA256SUMS.sigstore.json"

# 1.（主）SHA256SUMS の署名を確かめる
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity "https://github.com/takuho0115/sysfail-sim-game/.github/workflows/release.yaml@refs/tags/$TAG" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  SHA256SUMS

# 2.（主）署名を確かめた SHA256SUMS で実行ファイルを確かめる
grep '  sysfail-darwin-arm64$' SHA256SUMS | shasum -a 256 -c -

# 3.（補助）ad-hoc 署名であること（Signature=adhoc と出る）。作者を示すものではないので、本物かどうかの根拠は 1・2 だけ
codesign -dv ./sysfail-darwin-arm64

# 4. 1・2 が通った場合だけ: そのファイル 1 つだけ、ダウンロードの印を外して確かめる
mv sysfail-darwin-arm64 sysfail
chmod +x sysfail
xattr -d com.apple.quarantine sysfail
./sysfail --version
```

注意:

- 手順 4 の操作は、手順 1・2 を通した公式の sysfail の 1 ファイルにだけ行います。検証していないファイル、sysfail 以外のファイル、Issue・チャット・ほかのサイトで指示されたファイルには行わないでください。ディレクトリにまとめて行うこともしないでください。
- Gatekeeper 全体を無効にする操作や、システム設定での恒久的な許可はしないでください。

### 動作報告のお願い

macOS で動かした結果は、うまくいった場合も含めて知らせてもらえると助かります。[Issues](https://github.com/takuho0115/sysfail/issues/new/choose) の「macOS の動作報告」のテンプレートに、macOS のバージョン、チップ、colima のバージョン、`sysfail doctor` の出力を貼ってください。入れ方そのものの問題（起動しない、Gatekeeper に止められた など）は「macOS での導入・起動の問題」のテンプレートを使ってください。

## 不具合の報告・質問

不具合の報告と質問は、このリポジトリで受け付けます。

- **不具合の報告**: [Issues](https://github.com/takuho0115/sysfail/issues/new/choose)。テンプレートに沿って、OS、`sysfail --version`、`sysfail doctor` の出力、再現の手順を書いてください。
- **質問・要望**: [Discussions](https://github.com/takuho0115/sysfail/discussions)

トークンやパスワードなどの秘密は貼らないでください。ユーザー名やホスト名は伏せてかまいません。

## ライセンス

All rights reserved. 配布はこのリポジトリでの一次配布に限り、再配布（二次配布）は禁止です。詳しくは [LICENSE.md](LICENSE.md) を見てください。

実行ファイルに組み込まれた第三者のソフトウェアの著作権表示とライセンス文は、各 Release に添付の `THIRD_PARTY_NOTICES.txt` にあります。
