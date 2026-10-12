# インストール

Releases の各バージョンには、次の 10 ファイルが添付されています（v1.1.0 以降。v1.0.x は `custom-levels-kit.tar.gz` を除く 9 ファイル）。

| ファイル | 内容 |
|---|---|
| `sysfail-linux-amd64` / `sysfail-linux-arm64` | Linux 用の実行ファイル（一般的な PC は amd64、ARM の端末は arm64） |
| `sysfail-darwin-arm64` | macOS（Apple Silicon）用の実行ファイル（検証中のプレビュー） |
| `install.sh` | macOS 用のインストールスクリプト |
| `sysfail-windows-amd64.exe` | Windows 用ランチャー（同じバージョンの `sysfail-linux-amd64` と組で使います） |
| `THIRD_PARTY_NOTICES.txt` | 実行ファイルに組み込んだ他者のソフトウェアの著作権表示とライセンス文 |
| `LICENSE.md` | 利用条件 |
| `custom-levels-kit.tar.gz` | カスタムステージを作るためのキット（雛形と書式の説明。v1.1.0 以降） |
| `SHA256SUMS` | 各ファイルの SHA-256（ファイルが改ざんされていないかを調べるための値） |
| `SHA256SUMS.sigstore.json` | `SHA256SUMS` に対する作者の電子署名 |

公式の配布場所は `https://github.com/takuho0115/sysfail/releases` だけです。ほかのページや Release の本文に書かれたリンクからは入手しないでください。

## OS ごとの手順

- [Linux / WSL2](linux.md): Linux 版の入れ方
- [Windows](windows.md): WSL2 の中の sysfail を起動するランチャー `sysfail.exe` の入れ方と使い方
- [macOS（Apple Silicon）](macos.md): 検証中のプレビュー版の入れ方と、colima の推奨設定

必要なソフトとバージョンは[動作環境](../requirements.md)にまとめています。ダウンロードしたファイルは、[配布物が本物か確かめる](../verify.md)の手順で確かめてから使ってください。
