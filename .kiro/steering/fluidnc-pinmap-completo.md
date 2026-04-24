---
inclusion: always
---

# MKS TinyBee V1.0 — Pinmap Completo

Fonte oficial: [makerbase-mks/MKS-TinyBee README](https://github.com/makerbase-mks/MKS-TinyBee)

**Nota de projeto:** Display LCD externo (EXP1/EXP2) NÃO será usado. Os GPIOs desses conectores estão disponíveis para endstops e outras funções.

---

## Todos os GPIOs do ESP32 na TinyBee

| GPIO    | Função na placa        | Tipo       | Uso no projeto                          |
|---------|------------------------|------------|-----------------------------------------|
| gpio.0  | EXP1 LCD_D4            | I/O        | Livre                                   |
| gpio.2  | 3D Touch (servo/PWM)   | I/O        | **Z- endstop** (conector 3D Touch)      |
| gpio.4  | EXP1 LCD_RS            | I/O        | **X+ endstop** (conector EXP1)          |
| gpio.5  | TF Card CS / EXP2      | I/O        | SD Card CS — fixo                       |
| gpio.6  | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.7  | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.8  | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.9  | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.10 | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.11 | Flash interna          | ⛔ PROIBIDO | Nunca usar                              |
| gpio.12 | EXP2 BTN_EN2           | I/O        | Livre                                   |
| gpio.13 | EXP1 BTN_ENC           | I/O        | **Y- motor1 endstop** (auto-squaring)   |
| gpio.14 | EXP2 BTN_EN1           | I/O        | **Y+ endstop** (compartilhado Y0/Y1)    |
| gpio.15 | EXP1 LCD_D6            | I/O        | **Spindle PWM** ⚠️ pulsos no boot       |
| gpio.16 | EXP1 LCD_D5 / UART2 RX | I/O        | Livre (reserva para UART/VFD)           |
| gpio.17 | EXP1 LCD_D7 / UART2 TX | I/O        | Livre (alt. spindle sem pulsos no boot) |
| gpio.18 | TF Card SCK / EXP2     | I/O        | SPI SCK — fixo                          |
| gpio.19 | TF Card MISO / EXP2    | I/O        | SPI MISO — fixo                         |
| gpio.20 | —                      | —          | Não exposto na placa                    |
| gpio.21 | EXP1 LCD_EN            | I/O        | Livre                                   |
| gpio.22 | Z_STOP                 | I/O        | **Z+ endstop** (conector Z-LIMIT)       |
| gpio.23 | TF Card MOSI / EXP2    | I/O        | SPI MOSI — fixo                         |
| gpio.24 | —                      | —          | Não exposto na placa                    |
| gpio.25 | I2S BCK                | I/O        | I2S Bit Clock — fixo, não alterar       |
| gpio.26 | I2S WS                 | I/O        | I2S Word Select — fixo, não alterar     |
| gpio.27 | I2S DATA               | I/O        | I2S Data — fixo, não alterar            |
| gpio.28 | —                      | —          | Não exposto na placa                    |
| gpio.29 | —                      | —          | Não exposto na placa                    |
| gpio.30 | —                      | —          | Não exposto na placa                    |
| gpio.31 | —                      | —          | Não exposto na placa                    |
| gpio.32 | Y_STOP                 | I/O        | **Y- motor0 endstop** (conector Y-LIMIT)|
| gpio.33 | X_STOP                 | I/O        | **X- endstop** (conector X-LIMIT)       |
| gpio.34 | TH2 / SD_DET           | ⚠️ INPUT   | SD Card Detect (jumper J2 em SDDET)     |
| gpio.35 | MT_DET                 | ⚠️ INPUT   | **Probe** (compartilhado c/ Reset)      |
| gpio.36 | TH1                    | ⚠️ INPUT   | **Feed Hold**                           |
| gpio.37 | —                      | —          | Não exposto na placa                    |
| gpio.38 | —                      | —          | Não exposto na placa                    |
| gpio.39 | TB                     | ⚠️ INPUT   | **Cycle Start**                         |

> ⚠️ **GPIOs 34, 35, 36, 39 são INPUT-ONLY** no ESP32 — sem pull-up interno de hardware, use `:pu` no config para ativar via software.

---

## Pinos I2S (Motores — virtuais, não são GPIOs físicos)

| I2SO   | Função           | Status no projeto              |
|--------|------------------|-------------------------------|
| I2SO.0 | X Enable         | X motor                       |
| I2SO.1 | X Step           | X motor                       |
| I2SO.2 | X Direction      | X motor                       |
| I2SO.3 | Y Enable         | Y motor0 (lado A do pórtico)  |
| I2SO.4 | Y Step           | Y motor0                      |
| I2SO.5 | Y Direction      | Y motor0                      |
| I2SO.6 | Z Enable         | Z motor                       |
| I2SO.7 | Z Step           | Z motor                       |
| I2SO.8 | Z Direction      | Z motor                       |
| I2SO.9 | E0 Enable        | **Y motor1** (lado B pórtico) |
| I2SO.10| E0 Step          | **Y motor1**                  |
| I2SO.11| E0 Direction     | **Y motor1**                  |
| I2SO.12| E1 Enable        | Livre                         |
| I2SO.13| E1 Step          | Livre                         |
| I2SO.14| E1 Direction     | Livre                         |
| I2SO.16| Heated Bed (saída)| **Flood coolant** (M8)       |
| I2SO.17| HE0 (saída)      | **Mist coolant** (M7)         |

---

## Conectores Físicos — Referência Rápida

### Endstops dedicados (conector 3 pinos: GND / Signal / +5V)
| Conector | GPIO    | Uso no projeto         |
|----------|---------|------------------------|
| X-LIMIT  | gpio.33 | X- endstop             |
| Y-LIMIT  | gpio.32 | Y- motor0 endstop      |
| Z-LIMIT  | gpio.22 | Z+ endstop (homing)    |

### Endstops extras (via GPIOs livres — fiação adaptada)
| GPIO    | Conector físico | Uso no projeto              |
|---------|-----------------|-----------------------------|
| gpio.4  | EXP1 pino LCD_RS| X+ endstop                  |
| gpio.13 | EXP1 pino BTN_ENC| Y- motor1 endstop (auto-sq.)|
| gpio.14 | EXP2 pino BTN_EN1| Y+ endstop (Y0 e Y1)       |
| gpio.2  | 3D Touch        | Z- endstop                  |

### Controles (input-only)
| Conector | GPIO    | Uso no projeto  |
|----------|---------|-----------------|
| MT_DET   | gpio.35 | Probe           |
| TH1      | gpio.36 | Feed Hold       |
| TB       | gpio.39 | Cycle Start     |

### Spindle
| Conector | GPIO    | Observação                              |
|----------|---------|-----------------------------------------|
| EXP1 LCD_D6 | gpio.15 | PWM ativo ⚠️ pulsos curtos no boot  |
| EXP1 LCD_D7 | gpio.17 | Alternativa recomendada (sem pulsos)  |

### Coolant / Relés
| Conector     | I2SO    | Uso no projeto  |
|--------------|---------|-----------------|
| Heated Bed   | i2so.16 | Flood (M8)      |
| HE0          | i2so.17 | Mist (M7)       |

### SD Card
| Função | GPIO    |
|--------|---------|
| CS     | gpio.5  |
| SCK    | gpio.18 |
| MISO   | gpio.19 |
| MOSI   | gpio.23 |
| DET    | gpio.34 |

---

## GPIOs Livres Restantes no Projeto

| GPIO    | Conector físico | Observação                              |
|---------|-----------------|-----------------------------------------|
| gpio.0  | EXP1 LCD_D4     | Livre — sem conflito                    |
| gpio.12 | EXP2 BTN_EN2    | Livre — sem conflito                    |
| gpio.16 | EXP1 LCD_D5     | Livre — reserva para UART2 RX (VFD)    |
| gpio.17 | EXP1 LCD_D7     | Livre — alternativa spindle / UART2 TX |
| gpio.21 | EXP1 LCD_EN     | Livre — sem conflito                    |

---

## Conflitos e Alertas do Projeto Atual

| Situação | Detalhe |
|----------|---------|
| ⚠️ gpio.2 em uso para Z- | Se ativar Laser, o `output_pin` do Laser precisa ser realocado para gpio.0, gpio.12 ou gpio.21 |
| ⚠️ gpio.15 para spindle | Pode dar pulsos curtos no boot — risco de ativar spindle. Considere migrar para gpio.17 |
| ⚠️ gpio.35 compartilhado | Probe e Reset usam o mesmo pino MT_DET — apenas um pode estar ativo por vez |
| ℹ️ Display LCD | EXP1/EXP2 não serão usados — GPIOs desses conectores estão disponíveis |
