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

