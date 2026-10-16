# 困ったとき {#troubleshooting}

## まず `sysfail doctor` を実行する {#doctor}

`sysfail doctor` は、ゲームに必要なソフトや設定がそろっているかをまとめて調べるコマンドです。遊ぶ前に一度は実行してください。

```bash
sysfail doctor

# 出力例（一部抜粋）:
# 動作前提の確認
#   [OK] docker コマンド: あり
#   [OK] docker Engine のバージョン: 27.0.0
#   [OK] docker compose: 2.27.0
#   [OK] tmux: 3.3a
#   [!!] コンテナランタイムのバージョン: runc 1.2.8 / containerd 1.7.27
#        対処: コンテナランタイムのバージョンを確認できませんでした（拒否表が空、またはバージョンを取得できません）。既知の脆弱性の有無は確認されていません
#        詳しく: https://takuho0115.github.io/sysfail/ja/troubleshooting/pf-07/
#   [OK] 動作保証の環境: Linux + Docker Engine
# 結果: 全ての前提を満たしています。警告 1 件
```

- `[OK]` は問題なし、`[!!]` は警告（ゲームは起動できます）、`[NG]` は起動に必要な条件を満たしていないことを表します。
- 問題がある行には、1 行の「対処」と、その項目の説明ページの URL（「詳しく」）が表示されます。手順やコマンドの例は説明ページにあります。ネットワークにつながらないときも、「対処」の 1 行で何を直せばよいかが分かります。
- `sysfail doctor` は、検証用のコンテナを起動して、隔離テストと、docker デーモンが動いているマシンのカーネルの確認まで行います（全体で最大約 2 分かかります）。`sysfail doctor --build` は、足りないゲーム用イメージを ghcr.io から取得してから、同じ確認を行います（初回のみ。約 450MB のダウンロード。docker のデータ領域の空きが 6GB 未満なら取得せず、空き容量を増やす方法を表示します）。`--assets-dir` に images/ を渡したときは、取得の代わりに手元でビルドします（5〜20 分・約 4GB）。
- スクリプトから使う場合、`[NG]` が 1 件でもあれば終了コードは 3、無ければ 0 です。

## doctor の診断項目 {#doctor-items}

- [PF-01 docker コマンド](pf-01.md)
- [PF-02 docker デーモン](pf-02.md)
- [PF-03 docker の種類](pf-03.md)
- [PF-04 docker Engine のバージョン](pf-04.md)
- [PF-05 docker compose](pf-05.md)
- [PF-06 tmux](pf-06.md)
- [PF-07 コンテナランタイムのバージョン](pf-07.md)
- [PF-08 docker データ領域の空き](pf-08.md)
- [PF-09 docker VM のメモリ](pf-09.md)
- [PF-10 動作保証の環境](pf-10.md)
- [PF-11 docker の接続先](pf-11.md)
- [PF-12 ネットワークのアドレス範囲](pf-12.md)
- [PF-13 ファイアウォールの方式](pf-13.md)
- [PF-14 ホスト自身の待ち受け](pf-14.md)
- [PF-15 ブリッジの netfilter](pf-15.md)
- [PF-16 残骸コンテナ](pf-16.md)
- [PF-17 隔離テスト（docker 脱出防止）](pf-17.md)
- [PF-18 カーネルモジュール](pf-18.md)
- [PF-19 ゲーム用イメージ](pf-19.md)
