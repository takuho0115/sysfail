# 困ったとき

## まず `sysfail doctor` を実行する

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

## よく出る警告と対処

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
