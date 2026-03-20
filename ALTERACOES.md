# Documentação das Alterações — Marlin Firmware para Anet A8 Mini

> **Autor:** Edilson Correa  
> **Base:** Marlin 2.1.2.1 (commit `09d0b4d152`)  
> **Branch:** `anet_a8_mini_tmc2209`  
> **Placa alvo:** MKS Gen L V2.1  
> **Impressora:** Anet A8 Mini  

---

## Sumário

1. [Identificação e Placa](#1-identificação-e-placa)
2. [Drivers de Motor — TMC2209](#2-drivers-de-motor--tmc2209)
3. [Configurações de Comunicação Serial](#3-configurações-de-comunicação-serial)
4. [Geometria e Dimensões da Impressora](#4-geometria-e-dimensões-da-impressora)
5. [BLTouch / Sensor de Nivelamento](#5-bltouch--sensor-de-nivelamento)
6. [Nivelamento Automático da Mesa (ABL)](#6-nivelamento-automático-da-mesa-abl)
7. [Controle de Temperatura — PIDs](#7-controle-de-temperatura--pids)
8. [Sensores de Temperatura](#8-sensores-de-temperatura)
9. [Movimentação e Mecânica](#9-movimentação-e-mecânica)
10. [Configurações TMC2209 Avançadas](#10-configurações-tmc2209-avançadas)
11. [Display — BTT Mini12864](#11-display--btt-mini12864)
12. [LEDs e NeoPixel](#12-leds-e-neopixel)
13. [Ventilação e Coolers](#13-ventilação-e-coolers)
14. [Perfis de Pré-aquecimento](#14-perfis-de-pré-aquecimento)
15. [EEPROM e Persistência](#15-eeprom-e-persistência)
16. [Recursos Adicionais do Firmware](#16-recursos-adicionais-do-firmware)
17. [Troca de Filamento (Advanced Pause)](#17-troca-de-filamento-advanced-pause)
18. [Menu Customizado](#18-menu-customizado)
19. [Alterações nos Arquivos de Pinos](#19-alterações-nos-arquivos-de-pinos)
20. [Histórico de Commits](#20-histórico-de-commits)

---

## 1. Identificação e Placa

**Arquivo:** `Marlin/Configuration.h`

| Parâmetro | Valor anterior | Valor novo |
|---|---|---|
| `STRING_CONFIG_H_AUTHOR` | `"(none, default config)"` | `"(Edilson Correa)"` |
| `MOTHERBOARD` | `BOARD_RAMPS_14_EFB` | `BOARD_MKS_GEN_L_V21` |
| `CUSTOM_MACHINE_NAME` | `"3D Printer"` (comentado) | `"Anet A8 Mini"` (comentado) |
| `MACHINE_UUID` | `"00000000-..."` (comentado) | `"3869fe0b-1677-47d7-8830-3ef8caad2cf5"` (comentado) |

> A placa foi migrada de uma RAMPS 1.4 genérica para a **MKS Gen L V2.1**, que é compatível com pinagem RAMPS mas possui suporte nativo a drivers TMC via UART.

---

## 2. Drivers de Motor — TMC2209

**Arquivo:** `Marlin/Configuration.h`

Todos os drivers foram migrados de **A4988** para **TMC2209** (modo UART silencioso). O eixo Z duplo (Z2) também foi habilitado.

| Eixo | Antes | Depois |
|---|---|---|
| `X_DRIVER_TYPE` | `A4988` | `TMC2209` |
| `Y_DRIVER_TYPE` | `A4988` | `TMC2209` |
| `Z_DRIVER_TYPE` | `A4988` | `TMC2209` |
| `Z2_DRIVER_TYPE` | comentado | `TMC2209` |
| `E0_DRIVER_TYPE` | `A4988` | `TMC2209` |

> O **eixo Z2** foi habilitado para sincronizar dois motores de passo no eixo Z, permitindo nivelamento independente via `Z_STEPPER_AUTO_ALIGN`.

---

## 3. Configurações de Comunicação Serial

**Arquivo:** `Marlin/Configuration.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `BAUD_RATE_GCODE` | comentado | **habilitado** |
| `BAUDRATE_2` | `250000` (comentado) | `115200` (comentado) |

---

## 4. Geometria e Dimensões da Impressora

**Arquivo:** `Marlin/Configuration.h`

### Dimensões da área de impressão

| Parâmetro | Antes | Depois |
|---|---|---|
| `X_BED_SIZE` | 200 mm | **150 mm** |
| `Y_BED_SIZE` | 200 mm | **150 mm** |
| `Z_MAX_POS` | 200 mm | **150 mm** |

> A Anet A8 Mini possui mesa de **150×150 mm** e altura de impressão de **150 mm**, diferentemente da Anet A8 padrão (220×220×240 mm).

### Limites de posição

| Parâmetro | Antes | Depois |
|---|---|---|
| `X_MIN_POS` | `0` | `-3` |
| `MIN_SOFTWARE_ENDSTOPS` | habilitado | **desabilitado** |

> `X_MIN_POS = -3` adiciona uma margem negativa no eixo X para permitir que o bico alcance toda a extensão esquerda da mesa, compensando o offset do sensor de nivelamento.

---

## 5. BLTouch / Sensor de Nivelamento

**Arquivo:** `Marlin/Configuration.h`

### Habilitação do BLTouch

| Parâmetro | Antes | Depois |
|---|---|---|
| `Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN` | habilitado | **desabilitado** |
| `Z_MIN_PROBE_PIN` | comentado | `32` |
| `Z_PROBE_SERVO_NR` | comentado | `0` |
| `Z_SERVO_ANGLES` | comentado | `{ 70, 0 }` |
| `BLTOUCH` | comentado | **habilitado** |

### Offset do probe (posição relativa bico→sensor)

| Parâmetro | Antes | Depois |
|---|---|---|
| `NOZZLE_TO_PROBE_OFFSET` | `{ 10, 10, 0 }` | `{ -27, 0, 0 }` |

> O sensor está posicionado **27 mm à esquerda** do bico no eixo X. O offset Z é calibrado via EEPROM com o Probe Offset Wizard.

### Parâmetros de probing

| Parâmetro | Antes | Depois |
|---|---|---|
| `XY_PROBE_FEEDRATE` | `133*60` mm/min | `150*60` mm/min |
| `MULTIPLE_PROBING` | comentado | `2` |
| `EDITABLE_SERVO_ANGLES` | comentado | **habilitado** |

### Script pós-nivelamento

```
Antes:  (comentado) "G1 Z10 F12000\nG1 X15 Y330\nG1 Z0.5\nG1 Z10"
Depois: "M500\nG28\nM140 S0\nM104 S0\nM501"
```

> Após o G29 (nivelamento), o firmware salva os dados na EEPROM (`M500`), volta ao home (`G28`), desliga a mesa e o hotend, e recarrega as configurações (`M501`).

---

## 6. Nivelamento Automático da Mesa (ABL)

**Arquivo:** `Marlin/Configuration.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `AUTO_BED_LEVELING_BILINEAR` | comentado | **habilitado** |
| `RESTORE_LEVELING_AFTER_G28` | comentado | **habilitado** |
| `PREHEAT_BEFORE_LEVELING` | comentado | **habilitado** |
| `DEBUG_LEVELING_FEATURE` | comentado | **habilitado** |
| `LCD_BED_LEVELING` | comentado | **habilitado** |
| `LCD_BED_TRAMMING` | comentado | **habilitado** |

> O método **Bilinear** cria uma malha de compensação de altura sobre a mesa, corrigindo irregularidades em tempo real durante a impressão. O pré-aquecimento antes do nivelamento garante leituras mais precisas (expansão térmica considerada).

---

## 7. Controle de Temperatura — PIDs

**Arquivo:** `Marlin/Configuration.h`

### PID do Hotend

Valores calibrados em **17/07/2023** via `M303 E0 C8 S200` para o hotend personalizado da Anet A8 Mini.

| Parâmetro | Antes | Depois |
|---|---|---|
| `DEFAULT_Kp` | 22.20 | **24.88** |
| `DEFAULT_Ki` | 1.08 | **1.91** |
| `DEFAULT_Kd` | 114.00 | **81.00** |

### PID da Mesa (Cama Aquecida)

| Parâmetro | Antes | Depois |
|---|---|---|
| `PIDTEMPBED` | comentado | **habilitado** |
| `DEFAULT_bedKp` | 10.00 | **115.40** |
| `DEFAULT_bedKi` | 0.023 | **16.57** |
| `DEFAULT_bedKd` | 305.4 | **535.75** |

> Os PIDs foram calibrados especificamente para a **mesa 150×150 mm** da Anet A8 Mini, que por ser menor tem comportamento térmico diferente das mesas maiores.

---

## 8. Sensores de Temperatura

**Arquivo:** `Marlin/Configuration.h` e `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `TEMP_SENSOR_BED` | `0` (desabilitado) | `1` (100kΩ termistor NTC) |
| `TEMP_SENSOR_BOARD` | `0` (desabilitado) | `1` (habilitado) |
| `TEMP_BOARD_PIN` | comentado | `TEMP_1_PIN` |

> O sensor de temperatura da placa permite monitorar o aquecimento dos drivers e ativar automaticamente o cooler da placa via `USE_CONTROLLER_FAN`.

### Proteção térmica ajustada

**Arquivo:** `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `WATCH_TEMP_PERIOD` | 40 s | **20 s** |
| `THERMAL_PROTECTION_BED_PERIOD` | 20 s | **40 s** |
| `THERMAL_PROTECTION_BED_HYSTERESIS` | 2°C | **4°C** |
| `WATCH_BED_TEMP_PERIOD` | 60 s | **80 s** |
| `MAX_CONSECUTIVE_LOW_TEMPERATURE_ERROR_ALLOWED` | comentado | `8` |

> Os períodos da mesa foram aumentados para evitar falsos alarmes de proteção térmica, comuns em mesas menores que atingem temperatura mais rapidamente.

---

## 9. Movimentação e Mecânica

**Arquivo:** `Marlin/Configuration.h`

### Passos por milímetro

| Eixo | Antes | Depois |
|---|---|---|
| X | 80 | 80 |
| Y | 80 | 80 |
| Z | 400 | 400 |
| E0 | 500 | **138.78** |

> O extrusor foi recalibrado. O valor **138.78 steps/mm** é típico de extrusores diretos com motor de passo padrão e relação de redução calibrada.

### Velocidade máxima

| Eixo | Antes | Depois |
|---|---|---|
| X (mm/s) | 300 | **400** |
| Y (mm/s) | 300 | **400** |
| Z (mm/s) | 5 | **20** |
| E (mm/s) | 25 | **50** |

### Aceleração máxima

| Parâmetro | Antes | Depois |
|---|---|---|
| X/Y (mm/s²) | 3000 | **2000** |
| Z (mm/s²) | 100 | 100 |
| E (mm/s²) | 10000 | 10000 |

### Aceleração padrão

| Parâmetro | Antes | Depois |
|---|---|---|
| `DEFAULT_ACCELERATION` (impressão) | 3000 | **400** |
| `DEFAULT_RETRACT_ACCELERATION` | 3000 | **1000** |
| `DEFAULT_TRAVEL_ACCELERATION` | 3000 | **1000** |

> A aceleração de impressão reduzida para **400 mm/s²** melhora a qualidade de impressão em uma estrutura acrílica como a Anet A8 Mini, que é propensa a vibrações.

### Direção dos motores

| Parâmetro | Antes | Depois |
|---|---|---|
| `INVERT_Y_DIR` | `true` | **`false`** |

### Homing

| Parâmetro | Antes | Depois |
|---|---|---|
| `HOMING_FEEDRATE_MM_M` (X/Y) | `50*60` mm/min | **`90*60` mm/min** |
| `HOMING_BUMP_MM` (X/Y/Z) | `{ 5, 5, 2 }` | **`{ 0, 0, 0 }`** |

> `HOMING_BUMP_MM = 0` desativa o segundo toque (bump) no homing, comportamento recomendado ao usar TMC2209 com StallGuard ou BLTouch preciso.

### Comprimento máximo de extrusão

| Parâmetro | Antes | Depois |
|---|---|---|
| `EXTRUDE_MAXLENGTH` | 200 mm | **600 mm** |

> Aumentado para 600 mm para suportar a troca automática de filamento com tubos longos (Bowden).

---

## 10. Configurações TMC2209 Avançadas

**Arquivo:** `Marlin/Configuration_adv.h`

### Corrente dos motores

| Driver | Antes (mA RMS) | Depois (mA RMS) |
|---|---|---|
| X | 800 | **400** |
| Y | 800 | **400** |
| Z | 800 | **400** |
| Z2 | 800 | **400** |
| E0 | 800 | **440** |

> 400 mA RMS é adequado para os motores 17HS4401 típicos da Anet A8 Mini (pico ~565 mA). O extrusor usa 440 mA para maior torque. Todos com 16 microsteps.

### Endereços UART (modo UART compartilhado)

| Driver | Antes | Depois |
|---|---|---|
| `X_SLAVE_ADDRESS` | comentado | `0` |
| `Y_SLAVE_ADDRESS` | comentado | `0` |
| `Z_SLAVE_ADDRESS` | comentado | `0` |
| `Z2_SLAVE_ADDRESS` | comentado | `0` |
| `E0_SLAVE_ADDRESS` | comentado | `0` |

### Funcionalidades TMC habilitadas

| Feature | Antes | Depois |
|---|---|---|
| `HYBRID_THRESHOLD` | comentado | **habilitado** |
| `TMC_DEBUG` | comentado | **habilitado** |

> **HYBRID_THRESHOLD** alterna automaticamente entre StealthChop (silencioso, baixa velocidade) e SpreadCycle (alta velocidade, mais torque) com thresholds configurados (padrão 100 mm/s).

---

## 11. Display — BTT Mini12864

**Arquivo:** `Marlin/Configuration.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `BTT_MINI_12864_V1` | comentado | **habilitado** |
| `DISPLAY_CHARSET_HD44780` | `JAPANESE` | `WESTERN` |
| `SDSUPPORT` | comentado | **habilitado** |
| `REVERSE_ENCODER_DIRECTION` | comentado | **habilitado** |
| `SPEAKER` | comentado | **habilitado** |

**Arquivo:** `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `PROBE_OFFSET_WIZARD` | comentado | **habilitado** |
| `PROBE_OFFSET_WIZARD_XY_POS` | comentado | `{ X_CENTER, Y_CENTER }` |
| `SOUND_MENU_ITEM` | comentado | **habilitado** |

### Pinos do display (pins_RAMPS.h)

| Sinal | Pino anterior | Pino novo |
|---|---|---|
| `BEEPER_PIN` | `AUX2_08_PIN` | `37` |
| `LCD_BACKLIGHT_PIN` | `AUX2_10_PIN` | `-1` (desabilitado) |
| `DOGLCD_A0` | `AUX2_07_PIN` | `16` |
| `DOGLCD_CS` | `AUX2_09_PIN` | `17` |
| `LCD_RESET_PIN` | não definido | `23` |
| `BTN_EN1` | `AUX2_06_PIN` | `31` |
| `BTN_EN2` | `AUX2_04_PIN` | `33` |
| `BTN_ENC` | `AUX2_03_PIN` | `35` |
| `SD_DETECT_PIN` | `AUX3_02_PIN` | `49` |
| `KILL_PIN` | `AUX2_05_PIN` | `41` |
| `LCD_CONTRAST` | não definido | `200` |

---

## 12. LEDs e NeoPixel

**Arquivo:** `Marlin/Configuration.h`

### RGB LED (hardware)

| Parâmetro | Antes | Depois |
|---|---|---|
| `RGB_LED` | comentado | **habilitado** |
| `RGB_LED_R_PIN` | comentado | `25` |
| `RGB_LED_G_PIN` | comentado | `27` |
| `RGB_LED_B_PIN` | comentado | `29` |
| `RGB_LED_W_PIN` | comentado | `-1` |

### NeoPixel

| Parâmetro | Antes | Depois |
|---|---|---|
| `NEOPIXEL_LED` | comentado | **habilitado** |
| `NEOPIXEL_TYPE` | `NEO_GRBW` | **`NEO_RGB`** |

### Menu de controle de LEDs

**Arquivo:** `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `LED_CONTROL_MENU` | comentado | **habilitado** |
| `LED_USER_PRESET_GREEN` | `128` | **`255`** |
| `LED_USER_PRESET_WHITE` | `255` | **`0`** |
| `LED_USER_PRESET_BRIGHTNESS` | `255` | **`200`** |
| `LED_USER_PRESET_STARTUP` | comentado | **habilitado** |

---

## 13. Ventilação e Coolers

**Arquivo:** `Marlin/Configuration.h` e `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `FAN_SOFT_PWM` | comentado | **habilitado** |
| `USE_CONTROLLER_FAN` | comentado | **habilitado** |
| `CONTROLLER_FAN_PIN` | `-1` | **`4`** |
| `CONTROLLER_FAN_MIN_BOARD_TEMP` | comentado | `40°C` |
| `CONTROLLER_FAN_EDITABLE` | comentado | **habilitado** |
| `FAN_KICKSTART_TIME` | comentado | `100 ms` |
| `FAN_KICKSTART_POWER` | comentado | `180` |
| `E0_AUTO_FAN_PIN` | `-1` | **`7`** |

> O cooler da placa (pino 4) liga automaticamente quando a temperatura da placa atinge 40°C ou quando os drivers estão ativos. O cooler do extrusor (pino 7, MOSFET E1) liga automaticamente com o hotend.

---

## 14. Perfis de Pré-aquecimento

**Arquivo:** `Marlin/Configuration.h`

| Perfil | Hotend (antes→depois) | Mesa (antes→depois) |
|---|---|---|
| **PLA** | 180°C → **200°C** | 70°C → **60°C** |
| **PETG** *(novo)* | — | — → **230°C / 80°C** |
| **ABS** | 240°C / 110°C | → **230°C / 70°C** |

> O perfil **PETG** foi adicionado como segundo preset (antigo ABS foi movido para terceiro). As temperaturas do PLA foram ajustadas para refletir o comportamento real do hotend da impressora.

---

## 15. EEPROM e Persistência

**Arquivo:** `Marlin/Configuration.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `EEPROM_SETTINGS` | comentado | **habilitado** |

> Com EEPROM habilitado, todas as configurações salvas via `M500` (PIDs, offsets, steps/mm, etc.) persistem após reinicialização.

---

## 16. Recursos Adicionais do Firmware

**Arquivo:** `Marlin/Configuration.h` e `Marlin/Configuration_adv.h`

| Feature | Antes | Depois | Descrição |
|---|---|---|---|
| `NOZZLE_PARK_FEATURE` | comentado | **habilitado** | Estaciona o bico em posição segura |
| `NOZZLE_PARK_POINT` | `{ X_MIN+10, Y_MAX-10, 20 }` | **`{ X_MIN+3, Y_MIN, 20 }`** | Posição de estacionamento |
| `PRINTCOUNTER` | comentado | **habilitado** | Contador de impressões e tempo total |
| `BABYSTEPPING` | comentado | **habilitado** | Ajuste fino do Z durante impressão |
| `DOUBLECLICK_FOR_Z_BABYSTEPPING` | comentado | **habilitado** | Duplo clique no encoder abre babystep |
| `MOVE_Z_WHEN_IDLE` | comentado | **habilitado** | Menu de mover Z em idle |
| `BABYSTEP_DISPLAY_TOTAL` | comentado | **habilitado** | Mostra total de babysteps |
| `SET_PROGRESS_MANUALLY` | comentado | **habilitado** | Suporte ao M73 para progresso |
| `HOST_ACTION_COMMANDS` | comentado | **habilitado** | Comunicação com host (OctoPrint) |
| `ADVANCED_PAUSE_FEATURE` | comentado | **habilitado** | Pausa avançada / troca de filamento |

---

## 17. Troca de Filamento (Advanced Pause)

**Arquivo:** `Marlin/Configuration_adv.h`

| Parâmetro | Antes | Depois |
|---|---|---|
| `PAUSE_PARK_RETRACT_FEEDRATE` | 60 mm/s | **40 mm/s** |
| `FILAMENT_CHANGE_UNLOAD_FEEDRATE` | 10 mm/s | **70 mm/s** |
| `FILAMENT_CHANGE_UNLOAD_LENGTH` | 100 mm | **300 mm** |
| `FILAMENT_CHANGE_SLOW_LOAD_LENGTH` | 0 mm | **25 mm** |
| `FILAMENT_CHANGE_FAST_LOAD_FEEDRATE` | 6 mm/s | **40 mm/s** |
| `FILAMENT_CHANGE_FAST_LOAD_LENGTH` | 0 mm | **300 mm** |
| `ADVANCED_PAUSE_PURGE_LENGTH` | 50 mm | **60 mm** |

> O comprimento de 300 mm foi definido para cobrir o caminho completo do tubo Bowden até o bico, necessário para garantir que o filamento antigo seja completamente removido.

---

## 18. Menu Customizado

**Arquivo:** `Marlin/Configuration_adv.h`

`CUSTOM_MENU_MAIN` habilitado com os seguintes itens:

| Item | Descrição | G-code |
|---|---|---|
| 1 | Home & UBL Info | `G28` → `G29` → `M500` → `G28` |
| 2 | Preheat PLA | *(gerado automaticamente)* |
| 3 | *(original)* | — |
| 4 | *(original)* | — |
| 5 | **Centralizar** | `G28` → `G1 Z5` → `G1 X75 Y75` |

> O item 1 foi alterado para executar o nivelamento completo e salvar na EEPROM. O item 5 centraliza o bico na posição X75 Y75 (centro da mesa 150×150).

---

## 19. Alterações nos Arquivos de Pinos

### `Marlin/src/pins/ramps/pins_MKS_GEN_L_V21.h`

O driver do slot **E1** foi reconfigurado para controlar o **segundo motor Z (Z2)**:

| Define anterior | Define novo | Pino |
|---|---|---|
| `E1_CS_PIN` | `Z2_CS_PIN` | 12 |
| `E1_DIAG_PIN` | `Z2_DIAG_PIN` | 15 |
| `E1_SERIAL_TX_PIN` | `Z2_SERIAL_TX_PIN` | 20 |
| `E1_SERIAL_RX_PIN` | `Z2_SERIAL_RX_PIN` | 12 |

### `Marlin/src/pins/ramps/pins_RAMPS.h`

#### Remapeamento E1 → Z2 (pinos de step/dir/enable/cs)

| Define anterior | Define novo | Pino |
|---|---|---|
| `E1_STEP_PIN` | `Z2_STEP_PIN` | 36 |
| `E1_DIR_PIN` | `Z2_DIR_PIN` | 34 |
| `E1_ENABLE_PIN` | `Z2_ENABLE_PIN` | 30 |
| `E1_CS_PIN` | `Z2_CS_PIN` | 44 |

#### Serial UART para Z2

| Parâmetro | Antes | Depois |
|---|---|---|
| `Z2_SERIAL_TX_PIN` | `-1` | `20` |
| `Z2_SERIAL_RX_PIN` | `-1` | `12` |

#### AUX2 — Pinos do display BTT Mini12864

| Pino AUX2 | Valor anterior | Valor novo | Uso |
|---|---|---|---|
| `AUX2_03_PIN` | 59 | **35** | `BTN_ENC` |
| `AUX2_04_PIN` | 63 | **33** | `BTN_EN2` |
| `AUX2_05_PIN` | 64 | **41** | `KILL_PIN` |
| `AUX2_06_PIN` | 40 | **40** | *(igual)* |
| `AUX2_07_PIN` | 44 | **16** | `DOGLCD_A0` |
| `AUX2_08_PIN` | 42 | **37** | `BEEPER_PIN` |
| `AUX2_09_PIN` | 66 | **17** | `DOGLCD_CS` |
| `AUX2_10_PIN` | 65 | **-1** | `LCD_BACKLIGHT` (desabilitado) |

#### Adições para o BTT Mini12864

```c
#define LCD_BACKLIGHT_PIN  -1
#define LCD_CONTRAST       200
#define LCD_RESET_PIN      23      // novo
#define SD_DETECT_PIN      49      // alterado de AUX3_02_PIN
```

---

## 20. Histórico de Commits

| Commit | Descrição |
|---|---|
| `96b5237924` | Ajustes iniciais para a Anet A8 Mini (dimensões, placa, drivers) |
| `fa838e3985` | Ajustes para DRV8825 (intermediário) e driver do eixo Z2 |
| `855e355077` | Menu PETG adicionado; ajuste do pino do cooler da placa |
| `3362475585` | PIDs de temperatura calibrados; posicionamento do sensor de nivelamento |
| `c6b67fb687` | Merge branch `anet_a8_mini` |
| `80b307bb03` | Margem esquerda X (-3); progresso no display; Probe Wizard; filamento 300mm |
| `84d6c95542` | Suporte ao Display BTT Mini12864 |
| `0967ac0cb6` | Ajustes para a MKS Gen L 2.1 (pinos Z2, AUX2) |
| `1f6c32672e` | Ajuste do offset do BLTouch (`{ -27, 0, 0 }`) |
| `1bce98a2dd` | Migração completa para TMC2209 (correntes, UART, HYBRID_THRESHOLD) |

---

*Gerado em 14/03/2026 a partir do diff entre o Marlin 2.1.2.1 original e o branch `anet_a8_mini_tmc2209`.*
