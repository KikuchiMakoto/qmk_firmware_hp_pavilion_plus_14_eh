# QMK Firmware for HP Pavilion Plus 14-eh
`./keyboards/converter` に `git clone` して使用すること。

## ビルド方法 (QMK_MSYS不要・uvx + Scoop環境)

Scoop で導入済みの `make`, `busybox`, `gcc-arm-none-eabi` と `uvx` を使用し、`qmk_firmware` ルートディレクトリ（`C:\Users\kmakoto\qmk_firmware`）から PowerShell で直接 `.uf2` をビルド可能です。

```powershell
$env:PATH = "$env:USERPROFILE\scoop\apps\make\current\bin;$env:USERPROFILE\scoop\apps\gcc-arm-none-eabi\current\bin;$env:USERPROFILE\scoop\shims;$env:PATH"
uvx --with-requirements requirements.txt qmk compile -kb converter/hp_pavilion_plus_14_eh/rpi_pico -km default
```

ビルド完了後、`C:\Users\kmakoto\qmk_firmware\converter_hp_pavilion_plus_14_eh_rpi_pico_default.uf2` が生成されます。

## 基板ピン配置および実装仕様 (KiCad PCB `hp_pavilion_plus_rp2040` 実機)

> **【重要】実機FPCコネクタの偶数・奇数ピン反転について**  
> 0.8mmピッチFPCコネクタ（J1）の千鳥配列フットプリント定義の差異により、実機ではコネクタ端子の **奇数ピンと偶数ピンが全域でペア反転（$1 \leftrightarrow 2, 3 \leftrightarrow 4, \dots, 39 \leftrightarrow 40$）** しています。  
> 現在のファームウェア（`keyboard.json`）はこの反転後の二部グラフ（8 Rows × 15 Cols）に合わせてGPIOとマトリクス交点を再編・最適化済みです。

### Matrix Rows (8本)
| Row Index | Pico GPIO | 実機接続 (`J1` 1-idx) | 設計Net / FPC Pin (0-idx) |
| :---: | :---: | :---: | :---: |
| `0` | `GP4` | `J1` Pin 29 | `PFC29` (Pin 28) |
| `1` | `GP7` | `J1` Pin 32 | `PFC32` (Pin 31) |
| `2` | `GP8` | `J1` Pin 33 | `PFC33` (Pin 32) |
| `3` | `GP10` | `J1` Pin 35 | `PFC35` (Pin 34) |
| `4` | `GP11` | `J1` Pin 36 | `PFC36` (Pin 35) |
| `5` | `GP13` | `J1` Pin 38 | `PFC38` (Pin 37) |
| `6` | `GP14` | `J1` Pin 39 | `PFC39` (Pin 38) |
| `7` | `GP15` | `J1` Pin 40 | `PFC40` (Pin 39) |

### Matrix Cols (15本)
| Col Index | Pico GPIO | 実機接続 (`J1` 1-idx) | 設計Net / FPC Pin (0-idx) | 備考 |
| :---: | :---: | :---: | :---: | :--- |
| `0` | `GP16` | `J1` Pin 17 | `PFC17` (Pin 16) | |
| `1` | `GP17` | `J1` Pin 18 | `PFC18` (Pin 17) | |
| `2` | `GP18` | `J1` Pin 19 | `PFC19` (Pin 18) | |
| `3` | `GP19` | `J1` Pin 20 | `PFC20` (Pin 19) | |
| `4` | `GP20` | `J1` Pin 22 | `PFC22` (Pin 21) | **※Alt用 (下記注記参照)** |
| `5` | `GP21` | `J1` Pin 23 | `PFC23` (Pin 22) | |
| `6` | `GP22` | `J1` Pin 24 | `PFC24` (Pin 23) | Windows 専用 |
| `7` | `GP0`  | `J1` Pin 25 | `PFC25` (Pin 24) | Fn 専用 |
| `8` | `GP1`  | `J1` Pin 26 | `PFC26` (Pin 25) | |
| `9` | `GP2`  | `J1` Pin 27 | `PFC27` (Pin 26) | |
| `10` | `GP3`  | `J1` Pin 28 | `PFC28` (Pin 27) | |
| `11` | `GP5`  | `J1` Pin 30 | `PFC30` (Pin 29) | |
| `12` | `GP6`  | `J1` Pin 31 | `PFC31` (Pin 30) | |
| `13` | `GP9`  | `J1` Pin 34 | `PFC34` (Pin 33) | |
| `14` | `GP12` | `J1` Pin 37 | `PFC37` (Pin 36) | |

> **※Altキー（Left Alt / Right Alt）のハードウェア注意点**  
> ピン反転により、本来J1 Pin 22（GP20）に入るべきAlt信号が、基板上で未結線（NC）としていた `J1 Pin 21` に流れています。Left/Right Alt を使用する場合は、基板上で **J1 Pin 21 と Pin 22 の間をハンダブリッジ** して接続してください。

---

### インジケータ LED & MUTE LED の仕様
| LED名称 | Pico GPIO | 実機接続 (`J1` 1-idx) | カソード側 | 動作状況 |
| :--- | :---: | :---: | :---: | :--- |
| **CapsLock LED (白)** | `GP27` | `J1` Pin 1 | `GND` (Pin 7) | **動作可**（QMK標準LED機能で自動連動） |
| **MUTE LED (橙, F8)** | `GP26` | `J1` Pin 2 | `GND` (Pin 8) | **現状未対応 (オミット)** |

#### 【MUTE LED（F8）の現状について】
* ハードウェア配線自体は Pico `GP26` 経由で F8 の橙色LEDに接続されています。
* 標準のUSB HIDキーボード仕様にはOSホスト側からマイクミュート状態を受信するLEDレポート規格が存在せず、QMKコア側にも `LED_MIC_MUTE_PIN` の自動制御機能は含まれていません。
* そのため、現状のファームウェアでは **MUTE LEDは未対応（消灯状態のままオミット）** としています。今後点灯制御を行う場合は、`keymap.c` 内で `process_record_user` によるキー押下トグル制御や、Raw HID経由での独自ソフトウェア連携コードを追加して制御してください。

---

## キーマップ構成 (HP Action Keys 準拠)
* **レイヤー0（単体押し）**: メディア・アクション機能（検索, 輝度Down/Up, ミュート, 音量Down/Up, メディア再生操作, ディスプレイ切替, Insert）
* **レイヤー1（Fn同時押し）**: 標準 `F1` 〜 `F12` キー

---

## USB 仕様および pid.codes (Open Source Hardware) 規約準拠

USB-IF 本家の規格およびオープンソースハードウェアコミュニティ向け公式 USB ID 管理機構である pid.codes（`0x1209`）の規約に準拠しています。

* **Vendor ID (VID)**: `0x1209` (InterBiometrics / pid.codes - Open Source Hardware)
* **Product ID (PID)**: `0x0002` (pid.codes Test PID 2)
* **Manufacturer**: `Makoto KUNO <makomako0829bump@gmail.com>`
* **Product Name**: `HP Pavilion Plus 14-eh Keyboard Converter`
* **Serial Number**: `makomako0829bump@gmail.com:hp_pavilion_plus_14_eh`
  * USB ディスクリプタ（iSerialNumber）にメールアドレスプレフィックス形式のシリアル番号を埋め込んでいます。

---

## Remap によるキーマップ変更・マクロ設定

本ファームウェアは動的キーマップ 6 レイヤーおよびマクロ機能（Macro 0〜15）に対応しており、Remap 上で直感的にカスタマイズできます。

### 設定用定義ファイル
* **`remap.json`**: 本ディレクトリ直下に同梱されています（実機のキーサイズ・最下段の合流レイアウトに完全準拠）。
  * ※実機キーボードで `PrtSc` と `Delete` の間にあるボタン（電源ボタン）はマトリクス未結線のため、誤設定を防ぐ目的で Remap 上では物理的な隙間（空きスペース）として定義されています。

### Remap の利用手順:
1. Google Chrome または Microsoft Edge で **[https://remap-keys.app/configure](https://remap-keys.app/configure)** を開きます。
2. 「START REMAP FOR YOUR KEYBOARD」をクリックし、接続されたコンバーターを選択します。
3. 初回接続時またはレイアウトが表示されない場合は、本フォルダ直下の **`remap.json`** を画面上にドラッグ＆ドロップしてインポートします。
4. キーマップ編集画面が開き、GUI 上で直感的にキーの再配置を行えます。

### マクロ（Macro）の使い方（Fn＋キーの割り当てなど）:
1. レイヤー 1（Fn レイヤー）の任意のキーに、キーコード `M0` 〜 `M15`（Macro 0〜15）を配置します。
2. Remap の「MACRO」タブを開き、対象マクロ（例: `M0`）に送信したいキーストローク（例: `Ctrl + C`、ショートカット、定型テキスト等）を登録して保存します。
3. 実機で `Fn` を押しながらそのキーを押すことで、登録したマクロが実行されます。

---

## 配布用アセット（GitHub Releases）
GitHub Release を作成する際は、以下のファイルを一緒に配布することを推奨します:
1. `converter_hp_pavilion_plus_14_eh_rpi_pico_default.uf2`（ファームウェア本体）
2. `remap.json`（Remap 用定義ファイル）


