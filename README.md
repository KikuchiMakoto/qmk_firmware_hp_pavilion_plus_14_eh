# QMK Firmware for HP Pavilion Plus 14-eh
`./keyboards/converter` に `git clone` して使用すること。

## ビルド方法 (QMK_MSYS不要・uvx + Scoop環境)

Scoop で導入済みの `make`, `busybox`, `gcc-arm-none-eabi` と `uvx` を使用し、`qmk_firmware` ルートディレクトリ（`C:\Users\kmakoto\qmk_firmware`）から PowerShell で直接 `.uf2` をビルド可能です。

```powershell
$env:PATH = "$env:USERPROFILE\scoop\apps\make\current\bin;$env:USERPROFILE\scoop\apps\gcc-arm-none-eabi\current\bin;$env:USERPROFILE\scoop\shims;$env:PATH"
uvx --with-requirements requirements.txt qmk compile -kb converter/hp_pavilion_plus_14_eh/rpi_pico -km default
```

ビルド完了後、`C:\Users\kmakoto\qmk_firmware\converter_hp_pavilion_plus_14_eh_rpi_pico_default.uf2` が生成されます。

## 基板ピン配置 (KiCad PCB `hp_pavilion_plus_rp2040` 準拠)

### Matrix Rows (8本)
| Row Index | Pico GPIO | KiCad Net (`J1` 1-idx) | FPC Pin (0-idx) |
| :---: | :---: | :---: | :---: |
| `0` | `GP5` | `PFC30` (Pin 30) | `29` |
| `1` | `GP6` | `PFC31` (Pin 31) | `30` |
| `2` | `GP9` | `PFC34` (Pin 34) | `33` |
| `3` | `GP10` | `PFC35` (Pin 35) | `34` |
| `4` | `GP11` | `PFC36` (Pin 36) | `35` |
| `5` | `GP12` | `PFC37` (Pin 37) | `36` |
| `6` | `GP14` | `PFC39` (Pin 39) | `38` |
| `7` | `GP15` | `PFC40` (Pin 40) | `39` |

### Matrix Cols (15本)
| Col Index | Pico GPIO | KiCad Net (`J1` 1-idx) | FPC Pin (0-idx) | 備考 |
| :---: | :---: | :---: | :---: | :--- |
| `0` | `GP16` | `PFC17` (Pin 17) | `16` | |
| `1` | `GP17` | `PFC18` (Pin 18) | `17` | |
| `2` | `GP18` | `PFC19` (Pin 19) | `18` | |
| `3` | `GP19` | `PFC20` (Pin 20) | `19` | |
| `4` | `GP20` | `PFC22` (Pin 22) | `21` | Left Alt / Right Alt 専用 |
| `5` | `GP21` | `PFC23` (Pin 23) | `22` | |
| `6` | `GP22` | `PFC24` (Pin 24) | `23` | Windows 専用 |
| `7` | `GP0` | `PFC25` (Pin 25) | `24` | Fn 専用 |
| `8` | `GP1` | `PFC26` (Pin 26) | `25` | |
| `9` | `GP2` | `PFC27` (Pin 27) | `26` | |
| `10` | `GP3` | `PFC28` (Pin 28) | `27` | |
| `11` | `GP4` | `PFC29` (Pin 29) | `28` | |
| `12` | `GP7` | `PFC32` (Pin 32) | `31` | |
| `13` | `GP8` | `PFC33` (Pin 33) | `32` | Left Ctrl / Right Ctrl 専用 |
| `14` | `GP13` | `PFC38` (Pin 38) | `37` | Left Shift / Right Shift 専用 |

### インジケータ LED
| LED名称 | Pico GPIO | KiCad Net (`J1` 1-idx) | カソード側 (`J1` 1-idx) |
| :--- | :---: | :---: | :---: |
| **CapsLock LED (白)** | `GP26` | `LED_CAPSLK` (Pin 2) | `GND` (Pin 8) |
| **Mic Mute LED (橙, F8)** | `GP27` | `LED_MICMUTE` (Pin 1) | `GND` (Pin 7) |

