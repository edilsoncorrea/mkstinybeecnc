# Referência Rápida de Pinagem - MKS TinyBee V1.0

Este documento serve como referência rápida para a pinagem da MKS TinyBee ao configurar o FluidNC.

## 📍 Mapa de Pinos GPIO

### Motores (via I2S)

```
Motor X:
  Enable:    I2SO.0
  Step:      I2SO.1
  Direction: I2SO.2

Motor Y:
  Enable:    I2SO.3
  Step:      I2SO.4
  Direction: I2SO.5

Motor Z:
  Enable:    I2SO.6
  Step:      I2SO.7
  Direction: I2SO.8

Motor E0 (4º eixo):
  Enable:    I2SO.9
  Step:      I2SO.10
  Direction: I2SO.11

Motor E1 (5º eixo):
  Enable:    I2SO.12
  Step:      I2SO.13
  Direction: I2SO.14
```

### Interface I2S

```
BCK (Bit Clock):  gpio.25
WS (Word Select): gpio.26
DATA:             gpio.27
```

### SPI (SD Card)

```
MISO: gpio.19
MOSI: gpio.23
SCK:  gpio.18
CS:   gpio.5
```

### Limites/Endstops

```
X-Limit: gpio.33
Y-Limit: gpio.32
Z-Limit: gpio.22
```

### Controles

```
Reset/Emergency: gpio.35 (MT_DET no conector)
Feed Hold:       gpio.36 (TH1 no conector)
Cycle Start:     gpio.39 (TB no conector)
```

### Outros

```
Spindle PWM:     gpio.15
SD Card Detect:  gpio.34
```

## 🔌 Conectores Físicos

### Endstops (3 pinos cada)

```
┌─────────────┐
│  -  S  +    │  Conector Limit X
│  -  S  +    │  Conector Limit Y
│  -  S  +    │  Conector Limit Z
└─────────────┘

- = GND
S = Signal
+ = VCC (+5V)
```

**Configuração típica:**
- Sensor NO (Normalmente Aberto): `gpio.XX:low:pu`
- Sensor NC (Normalmente Fechado): `gpio.XX:high:pu`

### Motores (4 pinos cada)

```
┌─────────────┐
│ B2 B1 A1 A2 │
└─────────────┘

A1-A2 = Bobina A (par)
B1-B2 = Bobina B (par)
```

**Como identificar pares:**
- Use multímetro em modo continuidade
- Pares têm resistência (2-4Ω típico)
- Não-pares têm resistência infinita

### Spindle/PWM (2 pinos)

```
┌───────┐
│ PWM - │
└───────┘

PWM = Sinal PWM
-   = GND
```

**Jumper J1 seleciona tensão:**
- Posição 5V: 0-5V (para ESC)
- Posição VIN: 0-24V (para VFD)

### Alimentação

```
┌────────────┐
│  +24V  GND │
└────────────┘

Tensão: 12-24V DC
Corrente: Mínimo 15A recomendado
```

## 📋 Template para Config.yaml

### Estrutura Básica de Eixo

```yaml
axes:
  x:
    steps_per_mm: 80.000
    max_rate_mm_per_min: 8000.000
    acceleration_mm_per_sec2: 80.000
    max_travel_mm: 400.000
    
    motor0:
      limit_neg_pin: gpio.33:low:pu
      
      stepstick:
        step_pin: I2SO.1
        direction_pin: I2SO.2
        disable_pin: I2SO.0
```

### Todos os Eixos (Copy-Paste)

```yaml
axes:
  x:
    motor0:
      stepstick:
        step_pin: I2SO.1
        direction_pin: I2SO.2
        disable_pin: I2SO.0
  
  y:
    motor0:
      stepstick:
        step_pin: I2SO.4
        direction_pin: I2SO.5
        disable_pin: I2SO.3
  
  z:
    motor0:
      stepstick:
        step_pin: I2SO.7
        direction_pin: I2SO.8
        disable_pin: I2SO.6
  
  a:  # 4º eixo (opcional)
    motor0:
      stepstick:
        step_pin: I2SO.10
        direction_pin: I2SO.11
        disable_pin: I2SO.9
```

## 🔧 Configurações Comuns

### I2S (sempre o mesmo)

```yaml
i2so:
  bck_pin: gpio.25
  data_pin: gpio.27
  ws_pin: gpio.26
```

### SPI (sempre o mesmo)

```yaml
spi:
  miso_pin: gpio.19
  mosi_pin: gpio.23
  sck_pin: gpio.18
```

### SD Card

```yaml
sdcard:
  cs_pin: gpio.5
  card_detect_pin: gpio.34:low  # Requer jumper J2 em SDDET
```

### Limites

```yaml
# X negativo
limit_neg_pin: gpio.33:low:pu

# Y negativo  
limit_neg_pin: gpio.32:low:pu

# Z positivo
limit_pos_pin: gpio.22:low:pu
```

### Controles

```yaml
control:
  reset_pin: gpio.35:low
  feed_hold_pin: gpio.36:low
  cycle_start_pin: gpio.39:low
```

### Spindle PWM

```yaml
spindle:
  PWM:
    output_pin: gpio.15:high
```

## 🎨 Diagrama Visual

```
                MKS TinyBee V1.0
    ╔════════════════════════════════════════╗
    ║                                        ║
    ║  [X]  [Y]  [Z]  [E0] [E1]             ║  Motores
    ║                                        ║
    ║  ┌──┐  ┌──┐  ┌──┐                    ║
    ║  │X-│  │Y-│  │Z+│                    ║  Limites
    ║  └──┘  └──┘  └──┘                    ║
    ║                                        ║
    ║  [PWM]  [MT] [TH] [TB]                ║  Spindle/Controles
    ║                                        ║
    ║         [microSD]                      ║  SD Card
    ║                                        ║
    ║  [USB-C]              [24V] [GND]     ║  Conectores
    ╚════════════════════════════════════════╝

Legenda:
X-/Y-/Z+ = Endstops
PWM = Spindle
MT = MT_DET (Reset)
TH = TH1 (Feed Hold)
TB = TB (Cycle Start)
```

## ⚙️ Jumpers

### J1 - Tensão PWM

```
┌─────┬─────┐
│ 5V  │ VIN │
└─────┴─────┘
  ↑      ↑
  │      └── Selecione para PWM 0-24V (VFD)
  └───────── Selecione para PWM 0-5V (ESC)
```

### J2 - SD Card Detect

```
┌────────┬─────┐
│ SDDET  │ GND │
└────────┴─────┘
    ↑       ↑
    │       └── Desabilita detecção
    └────────── Habilita detecção (recomendado)
```

## 🧮 Calculadora Rápida

### Steps per MM

```
steps_per_mm = (passos_motor × microsteps) / pitch

Exemplos:
- GT2 20T:  (200 × 16) / 40  = 80
- Fuso 5mm: (200 × 16) / 5   = 640
- Fuso T8:  (200 × 16) / 2   = 1600
```

### Vref (Corrente do Driver)

```
Vref = Irms × 0.4

Exemplos:
- Motor 1.0A: 0.4V
- Motor 1.5A: 0.6V
- Motor 2.0A: 0.8V
```

## 🔍 Troubleshooting Rápido

### Motor não gira
1. Verifique steps_per_mm > 0
2. Verifique pins: `I2SO.X`
3. Teste com outro motor

### Motor gira errado
- Adicione `!` no direction_pin
- Ex: `direction_pin: !I2SO.2`

### Limite não funciona
1. Teste tipo: `:low` ou `:high`
2. Adicione pull-up: `:pu`
3. Teste fiação

### Spindle não liga
1. Verifique jumper J1
2. Teste polaridade: `:high` ou `:low`
3. Teste PWM com multímetro

## 📚 Referências Cruzadas

- [README completo](../README.md)
- [Guia de Hardware](hardware.md)
- [Guia de Configuração](configuracao.md)
- [Início Rápido](inicio-rapido.md)
- [Troubleshooting](troubleshooting.md)

---

**Dica:** Imprima este documento para ter à mão durante a configuração!
