# myball — ZMK自作ファームウェア(トラックボール + ロータリーエンコーダ)

## 構成

| | 左手 | 右手 |
|---|---|---|
| マイコン | nice!nano v2 | nice!nano v2 |
| キー数 | 15 + 親指2 = 17 | 15 + 親指2 = 17 |
| 追加デバイス | ロータリーエンコーダ (EC11) | トラックボール (PMW3610, SPI) |

合計34キー。ロータリーエンコーダはデフォルトで音量調整、トラックボールは常時マウスカーソルとして動作し、RAISEレイヤーでクリックを割り当てています。

## ファイル構成

```
zmk-config/
├── build.yaml                          # ビルド対象(board×shieldの組み合わせ)
├── .github/workflows/build.yml         # GitHub Actionsで自動ビルド
└── config/
    ├── west.yml                        # PMW3610ドライバーモジュールを追加
    ├── myball.keymap                   # キーマップ本体(左右共通)
    ├── myball_left.conf                # 左手用Kconfig
    ├── myball_right.conf               # 右手用Kconfig
    └── boards/shields/myball/
        ├── Kconfig.shield              # シールド名の登録
        ├── Kconfig.defconfig           # 左右で必要な機能を自動ON
        ├── myball.dtsi                 # 左右共通のキーマトリクス定義
        ├── myball_left.overlay         # 左手固有(エンコーダ配線)
        └── myball_right.overlay        # 右手固有(トラックボール配線)
```

## ⚠️ 必ず編集が必要な箇所(実配線に合わせる)

このファームウェアは「ゼロから作る」ためのテンプレートなので、ピン番号はすべて仮の値です。実際の基板・配線に合わせて以下を書き換えてください。

1. `config/boards/shields/myball/myball.dtsi`
   - `row-gpios` / `col-gpios` : キーマトリクスの行・列ピン
2. `config/boards/shields/myball/myball_left.overlay`
   - `a-gpios` / `b-gpios` : ロータリーエンコーダのA相/B相ピン
3. `config/boards/shields/myball/myball_right.overlay`
   - `SPIM_SCK` / `SPIM_MOSI` / `SPIM_MISO` : PMW3610とのSPI通信ピン
   - `cs-gpios` : チップセレクトピン
   - `irq-gpios` : モーション割り込みピン

nice!nano v2 のピン番号(P0.xx / P1.xx)とPro Micro互換番号(&pro_microのインデックス)の対応表は、ZMKの [nice_nano ピン配置](https://zmk.dev/docs/development/hardware-integration/new-shield) や nice!nano のピンアウト図を参照してください。

## ビルド方法

### GitHub Actionsを使う場合(推奨・簡単)

1. このフォルダの中身をまるごとGitHubの新規リポジトリにpush
2. Actionsタブで自動ビルドが走る(`build.yaml`の内容に従い `myball_left` / `myball_right` / `settings_reset` の3つの.uf2ファイルが生成される)
3. 生成された `myball_left.uf2` / `myball_right.uf2` をそれぞれの nice!nano にドラッグ&ドロップで書き込み

### ローカルでwest buildする場合

```bash
west init -l config
west update
west build -p -b nice_nano_v2 -- -DSHIELD=myball_left
west build -p -b nice_nano_v2 -d build_right -- -DSHIELD=myball_right
```

## トラブルシューティング

- **`Incorrect product id 0xFF (expecting 0x3E)!` エラー**
  → `myball_right.conf` の `CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000` のコメントを外す
- **トラックボールの向きが逆・上下反転している**
  → `myball_right.overlay` の `trackball` ノード内の `swap-xy;` `invert-x;` `invert-y;` のコメントを外して調整
- **エンコーダが逆回転として認識される**
  → `myball_left.overlay` の `a-gpios` と `b-gpios` を入れ替える

## キー数・レイアウトの変更方法

- キー数を増減する場合は `myball.dtsi` の `default_transform`(map)と `columns`/`rows`、および `row-gpios`/`col-gpios` を配線に合わせて修正し、`myball.keymap` のバインディング数もそれに合わせてください。
- 親指キーの数を増やす場合は `RC(3,x)` の行を追加してください。
