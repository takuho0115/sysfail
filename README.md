# sysfail

sysfail は、サーバー障害の対応をターミナルで練習するゲームです。レベルを始めると、あなたのパソコンの Docker の上に「どこかが壊れたサーバー群」が立ち上がります。ログや設定を調べて原因を突き止め、サービスを元に戻せばクリアです。

レベルは全部で 100 あります。インフラやサーバー運用を学びはじめた人が実戦の勘を養うのにも、経験者が腕試しをするのにも使えます。教材付きの教育版もあります。

このリポジトリは配布用で、ソースコードは公開していません。実行ファイルは [Releases](https://github.com/takuho0115/sysfail/releases) から入手してください。

## ドキュメント

入れ方や遊び方は、ドキュメントサイト <https://takuho0115.github.io/sysfail/> に載せています。

| ページ | 内容 |
|---|---|
| [動作環境](https://takuho0115.github.io/sysfail/ja/requirements/) | 必要な OS・Docker・docker compose・tmux のバージョン |
| [インストール](https://takuho0115.github.io/sysfail/ja/install/) | Releases に添付しているファイルと、OS ごとの入れ方の案内 |
| [Linux / WSL2](https://takuho0115.github.io/sysfail/ja/install/linux/) | Linux 版の入れ方 |
| [Windows](https://takuho0115.github.io/sysfail/ja/install/windows/) | 検証中のプレビュー。WSL2 の中の sysfail を起動するランチャー `sysfail.exe` の入れ方と使い方 |
| [macOS（Apple Silicon）](https://takuho0115.github.io/sysfail/ja/install/macos/) | 検証中のプレビュー版の入れ方と、colima の推奨設定 |
| [配布物が本物か確かめる](https://takuho0115.github.io/sysfail/ja/verify/) | cosign と SHA256SUMS で、ダウンロードしたファイルを確かめる手順 |
| [遊び方](https://takuho0115.github.io/sysfail/ja/play/) | はじめての遊び方と、教育版の遊び方 |
| [困ったとき](https://takuho0115.github.io/sysfail/ja/troubleshooting/) | `sysfail doctor` の見方と、よく出る警告への対処 |
| [レベル別ガイド](https://takuho0115.github.io/sysfail/ja/guides/) | レベルごとに、どこから調べるかのヒント |
| [自作ステージの作り方](https://takuho0115.github.io/sysfail/ja/custom-levels/) | 自分で作ったステージを検証して遊ぶ手順 |

## すぐに始める（Linux / WSL2）

[動作環境](https://takuho0115.github.io/sysfail/ja/requirements/)を満たした Linux か WSL2 で、次の手順で入れて、遊び始めます。Windows と macOS の手順はドキュメントサイトを見てください。

```bash
TAG=v1.1.0          # 入れたいバージョン
ARCH=amd64          # ARM の端末では arm64
BASE=https://github.com/takuho0115/sysfail/releases/download/$TAG
wget "$BASE/sysfail-linux-$ARCH" "$BASE/LICENSE.md" "$BASE/SHA256SUMS" "$BASE/SHA256SUMS.sigstore.json"
# ここで配布物が本物か確かめる（下の「配布物の検証」を参照）
chmod +x "sysfail-linux-$ARCH"
sudo mv "sysfail-linux-$ARCH" /usr/local/bin/sysfail

sysfail doctor   # 動作環境がそろっているか確かめる
sysfail 1        # レベル 1 から始める
```

<a id="配布物の検証必須"></a>

## 配布物の検証

公式の配布場所は `https://github.com/takuho0115/sysfail/releases` だけです。ダウンロードしたファイルは、実行する前に、作者の署名と SHA-256 で本物かどうか確かめることを強くおすすめします。手順は[配布物が本物か確かめる](https://takuho0115.github.io/sysfail/ja/verify/)を見てください。

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
