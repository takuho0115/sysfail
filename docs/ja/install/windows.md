# Windows: 検証中のプレビュー

> Windows 版（`sysfail.exe`）は検証中のプレビューで、動作の保証はありません（保証を始めるときはリリースノートでお知らせします）。
>
> v1.2.0 には、教育版（`--edu`）で起動するとターミナルが 1 つしか開かない不具合がありました。v1.2.1 で直っています。

`sysfail.exe` は、Windows のターミナルから WSL2 の中の Linux 版 sysfail を起動するランチャーです。ゲーム本体は WSL2 の中で動きます。起動の前に WSL2・Linux 版・Docker の状態を調べ、問題があれば直し方を表示して止まります。

## 必要なもの

- **Windows**: Windows 10 / 11（amd64）
- **WSL2**: PowerShell で `wsl --version` を実行してバージョンが表示されること（表示されない古い WSL は `wsl --update` で更新します）。使う Linux（ディストリビューション）が `wsl --list --verbose` で VERSION 2 になっていること
- **ディストリビューション**: Linux amd64。中に Docker Engine と docker compose v2 プラグインを入れます（バージョンの条件は[動作環境](../requirements.md)と同じです）。tmux は要りません
- **Windows Terminal**（`wt.exe`）: 無くても遊べます。その場合はコンソール画面 2 つで開きます

Docker Desktop の WSL 連携でも、ディストリビューションの中から `docker` コマンドが使えれば起動します。ただし動作の保証はなく、`doctor` が警告を出します。保証するのは、ディストリビューションの中に直接入れた Docker Engine だけです。

## 入れ方

`sysfail.exe` には、同じバージョンの Linux 版（`sysfail-linux-amd64`）とその SHA-256 の値が入っています。初めてダブルクリックしたときに、この Linux 版を WSL2 の中に入れます。ネットワークからは取得しません。起動のたびに SHA-256 を照合し、一致しなければ起動しません。

1. Windows のブラウザで、[Releases](https://github.com/takuho0115/sysfail/releases) から同じバージョンの `sysfail-windows-amd64.exe`・`SHA256SUMS`・`SHA256SUMS.sigstore.json` を同じフォルダにダウンロードします。**exe を実行する前に**、[配布物が本物か確かめる](../verify.md)の手順で `SHA256SUMS` の署名を確かめてから、PowerShell で `Get-FileHash .\sysfail-windows-amd64.exe -Algorithm SHA256` を実行して、その値を `SHA256SUMS` の `sysfail-windows-amd64.exe` の行と照合してください。

2. 確かめた exe の名前を `sysfail.exe` に変え、Windows の好きなフォルダに置いて、ダブルクリックします。

3. WSL2 に Linux 版が入っていなければ、ディストリビューション名、root 権限で置くこと、置き場所（`/usr/local/bin/sysfail`）、入れるバージョンを示した確認が表示されます。`yes` と入力すると Linux 版を入れ、続けてゲーム用イメージをビルドしてから（初回のみ 5〜20 分・約 4GB）、ゲームが始まります。

2 回目からは、確認なしでゲームが始まります。新しいバージョンの `sysfail.exe` をダブルクリックしたときは、「vX から vY に更新します」という確認が出ます。更新するときは、新しい exe も手順 1 で確かめてから使ってください。

自動で入れたり更新したりするのは、引数を付けずに起動したときだけです（`--distro`・`--console` だけを付けた場合も含みます）。次の場合は何も変えずに止まり、`sysfail.exe install` の実行を案内します。

- レベル番号や `doctor` などの引数を付けて起動したとき
- `--linux-path` を指定したとき
- 同じか、より新しいバージョンが入っているとき
- 入っているファイルを確かめられないとき、または確かめた結果に問題があるとき
- ゲームのプレイ中のとき（`.\sysfail.exe stop` で止めてから起動し直してください）

エラーで止まったときは、`sysfail.exe` が「Enter キーを押すと閉じます」と表示し、画面を開いたままにします。

### 手動で入れる・消す

自動で入らなかったときや、確認を出さずに入れたいときは、`sysfail.exe` を置いたフォルダで PowerShell から実行します。

```powershell
.\sysfail.exe install         # Linux 版を入れる・更新する。続けてゲーム用イメージをビルドする（--yes を付けると確認を省きます）
.\sysfail.exe doctor          # 動作環境を確かめる
.\sysfail.exe uninstall       # 入れた Linux 版を消す
```

`install` は Linux 版を置いたあと、WSL の中で `sysfail doctor --build` を実行し、ゲーム用イメージのビルドと動作環境の確認まで行います。ビルドに失敗しても Linux 版は入ったままです。表示された対処を済ませてから、`.\sysfail.exe doctor --build` をやり直してください。

`uninstall` が消すのは、入れた Linux 版のファイル 1 つだけです。クリア状況などの記録や Docker のイメージは残ります。

### Linux 版を自分で置く（上級者向け）

`sysfail.exe install` が使えない環境では、ディストリビューションの中に Linux 版を自分で置くこともできます。`sysfail.exe` と Linux 版は必ず同じバージョンのものを使い、片方を更新したらもう片方も入れ替えてください。

1. ディストリビューションの中で、下のコマンドで必要なファイルをダウンロードします。[配布物が本物か確かめる](../verify.md)の手順（`ARCH=amd64`）で、Linux 版と Windows 版の 2 ファイルとも `OK` になることを確かめてください。

   ```bash
   TAG=v1.2.3
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
- セキュリティソフトが `sysfail.exe` を隔離・削除したり、0 バイトのファイルに置き換えたりすることがあります。その場合は、手元の `sysfail.exe` で `Get-FileHash` を実行し、署名を確かめた `SHA256SUMS` の行と照合してください。一致しなければ使わずに削除し、Releases から取り直してください。一致した場合は誤検知なので、セキュリティソフトの窓口に申告し、定義の更新を待ってください。待てないときは、「Linux 版を自分で置く」の手順 1・2 で Linux 版を置けば、WSL の中で `sysfail` を直接実行して遊べます。会社などが管理している PC では、管理者に相談してください。

## Windows での制限

- クリア状況やログは、WSL のディストリビューションの既定ユーザーのホームに保存します。Windows 側には保存しません。ディストリビューションを変えると、記録も別になります。
- 2 つの画面とも、ディストリビューションの既定ユーザーで動きます。`--assets-dir` などに渡すパスも WSL 側のパスで書きます。
- ランチャーはシェルを通さずに Linux 版を起動するため、`~/.bashrc` などで設定した環境変数は効きません。
- tmux の一時離脱・再開（デタッチ・アタッチ）と、arm64 の Windows には対応していません。
- ゲーム環境の隔離は WSL2 の中の Docker が担います。ランチャーが Windows 側で隔離を追加するわけではありません。
