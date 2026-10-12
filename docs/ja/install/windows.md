# Windows

`sysfail.exe` は、Windows のターミナルから WSL2 の中の Linux 版 sysfail を起動するランチャーです。ゲーム本体は WSL2 の中で動きます。起動の前に WSL2・Linux 版・Docker の状態を調べ、問題があれば直し方を表示して止まります。

## 必要なもの

- **Windows**: Windows 10 / 11（amd64）
- **WSL2**: PowerShell で `wsl --version` を実行してバージョンが表示されること（表示されない古い WSL は `wsl --update` で更新します）。使う Linux（ディストリビューション）が `wsl --list --verbose` で VERSION 2 になっていること
- **ディストリビューション**: Linux amd64。中に Docker Engine と docker compose v2 プラグインを入れます（バージョンの条件は[動作環境](../requirements.md)と同じです）。tmux は要りません
- **Windows Terminal**（`wt.exe`）: 無くても遊べます。その場合はコンソール画面 2 つで開きます

Docker Desktop の WSL 連携でも、ディストリビューションの中から `docker` コマンドが使えれば起動します。ただし動作の保証はなく、`doctor` が警告を出します。保証するのは、ディストリビューションの中に直接入れた Docker Engine だけです。

## 入れ方

`sysfail.exe` には、同じバージョンの `sysfail-linux-amd64` の SHA-256 が埋め込まれています。起動のたびにこれを照合し、一致しなければ起動しません。2 つは必ず同じバージョンのものを使い、片方を更新したらもう片方も入れ替えてください。

1. ディストリビューションの中で 2 ファイルをダウンロードします。[配布物が本物か確かめる](../verify.md)の手順（`ARCH=amd64`）で、2 ファイルとも `OK` になることを確かめてください。

   ```bash
   TAG=v1.1.0
   BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
   wget "$BASE/sysfail-linux-amd64" "$BASE/sysfail-windows-amd64.exe" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
   ```

2. ディストリビューションの中に Linux 版を置きます。ランチャーは安全のため、Linux 版とその置き場所が root の持ち物で、ほかのユーザーが書き換えられないことを確かめます。`mv` だと持ち主が自分のままになるので、`install` を使ってください。

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

## 使い方

`sysfail.exe` を置いたフォルダで、PowerShell から実行します。

```powershell
.\sysfail.exe doctor                    # 動作環境の確認（WSL の中で sysfail doctor を実行します）
.\sysfail.exe 3                         # レベル 3 を始める
.\sysfail.exe                           # レベル選択画面から始める
.\sysfail.exe --console 3               # Windows Terminal を使わず、コンソール画面 2 つで開く
.\sysfail.exe stop                      # ゲームを止めて環境を片付ける
.\sysfail.exe --distro Ubuntu-24.04 3   # 既定以外のディストリビューションを使う
```

- ゲームは Windows Terminal の新しいウィンドウに、「司令画面」と「作業シェル」の 2 つのタブで開きます。
- `--console` を付けると、司令画面を今のコンソールで、作業シェルを新しいコンソールで開きます。
- `--distro` を省くと、WSL の既定のディストリビューション（`wsl --list --verbose` で `*` が付いたもの）を使います。既定のディストリビューションが無い場合や WSL1 の場合は起動しません。
- `list`・`--edu` など、Linux 版のコマンドやオプションも使えます。ランチャー自身のオプション（`--distro`・`--console`・`--linux-path`）はコマンドより前に置きます。Linux 版のオプションを先頭に置くときは `--` で区切ります（例: `.\sysfail.exe -- --assets-dir /opt/sysfail-assets 3`）。
- 司令画面で `q` を押すと、環境を片付けて終わります。作業シェルのタブは残るので、手で閉じてください。途中で画面を閉じてしまった場合は、両方を閉じて `.\sysfail.exe stop` を実行してから起動し直してください。

## 「Windows によって PC が保護されました」と出たら

`sysfail.exe` にはコード署名（Windows が発行元を確認するための署名）がありません。

- 初回の起動で SmartScreen の「Windows によって PC が保護されました」が出たら、「詳細情報」を押してから「実行」を押します。ただし、上の手順でハッシュを確かめたファイルだけを実行してください。
- セキュリティソフトが `sysfail.exe` を隔離・削除したり、0 バイトのファイルに置き換えたりすることがあります。その場合は置いたフォルダをスキャンの対象から外し、ハッシュを確かめ直したファイルを置き直してください。セキュリティソフトの窓口に誤検知として申告することもできます。

## Windows での制限

- クリア状況やログは、WSL のディストリビューションの既定ユーザーのホームに保存します。Windows 側には保存しません。ディストリビューションを変えると、記録も別になります。
- 2 つの画面とも、ディストリビューションの既定ユーザーで動きます。`--assets-dir` などに渡すパスも WSL 側のパスで書きます。
- ランチャーはシェルを通さずに Linux 版を起動するため、`~/.bashrc` などで設定した環境変数は効きません。
- tmux の一時離脱・再開（デタッチ・アタッチ）と、arm64 の Windows には対応していません。
- ゲーム環境の隔離は WSL2 の中の Docker が担います。ランチャーが Windows 側で隔離を追加するわけではありません。
