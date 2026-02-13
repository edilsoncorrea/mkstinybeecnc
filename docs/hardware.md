# Hardware - MKS TinyBee V1.0

## Visão Geral

A MKS TinyBee V1.0 é uma placa controladora CNC compacta baseada em ESP32, desenvolvida pela Makerbase. É uma excelente escolha para CNCs de hobby e pequenas máquinas profissionais.

## Especificações Técnicas

### Processador
- **MCU:** ESP32-WROOM-32
- **Clock:** 240 MHz dual-core
- **Flash:** 4MB
- **RAM:** 520KB SRAM

### Drivers de Motor
- **Quantidade:** 5 drivers integrados (X, Y, Z, E0, E1)
- **Tipo:** TMC2209
- **Corrente máxima:** 2A por motor (configurável)
- **Microstepping:** Até 1/256
- **StealthChop:** Operação silenciosa
- **SpreadCycle:** Modo de alta performance

### Alimentação
- **Tensão de entrada:** 12-24V DC
- **Corrente recomendada:** 15A mínimo (depende dos motores)
- **Proteção:** Fusível resetável
- **Conector:** Terminal de parafuso

### Conectividade
- **USB:** Type-C (programação e controle)
- **Wi-Fi:** IEEE 802.11 b/g/n (2.4GHz)
- **SD Card:** Suporte a cartões até 32GB (FAT32)

### Entradas/Saídas
- **Limites/Endstops:** 6 entradas (X±, Y±, Z±)
- **Controle:** 3 entradas (Reset, Feed Hold, Cycle Start)
- **Spindle PWM:** 1 saída (0-24V ou 0-5V configurável)
- **Sensores:** 2 entradas para termistores
- **Probe:** 1 entrada para sensor de altura (Z-probe)

## Layout da Placa

### Conectores Principais

#### Motores
```
┌─────────────────────────────────────┐
│  X-Motor  Y-Motor  Z-Motor          │
│  [====]   [====]   [====]           │
│                                     │
│  E0-Motor E1-Motor                  │
│  [====]   [====]                    │
└─────────────────────────────────────┘
```

Pinagem do conector (4 pinos):
1. B2 (Bobina B)
2. B1 (Bobina B)
3. A1 (Bobina A)
4. A2 (Bobina A)

#### Limites (Endstops)
```
Conector de 3 pinos:
- GND (-)
- Signal
- VCC (+5V)
```

Suporta sensores:
- Mecânicos (NO/NC)
- Indutivos (NPN/PNP)
- Ópticos

#### Spindle/PWM
```
Conector de 2 pinos:
- GND
- PWM (0-24V ou 0-5V)
```

### Jumpers e Configurações

#### J1 - Seleção de Tensão PWM
- **5V:** PWM de 0-5V (compatível com ESC/controladores lógica)
- **VIN:** PWM de 0-24V (compatível com VFDs)

#### J2 - Detecção SD Card
- **SDDET:** Habilita detecção de cartão (recomendado para FluidNC)
- **GND:** Desabilita detecção automática

#### J3 - Seleção UART/SPI
- Configuração para TMC2209 (geralmente já configurado de fábrica)

## Diagrama de Pinagem ESP32

### Pinos I2S (Motor Control)

Os motores são controlados via I2S, uma interface de comunicação serial que permite controle preciso:

```
I2SO.0  = X Enable
I2SO.1  = X Step
I2SO.2  = X Direction

I2SO.3  = Y Enable
I2SO.4  = Y Step
I2SO.5  = Y Direction

I2SO.6  = Z Enable
I2SO.7  = Z Step
I2SO.8  = Z Direction

I2SO.9  = E0 Enable
I2SO.10 = E0 Step
I2SO.11 = E0 Direction

I2SO.12 = E1 Enable
I2SO.13 = E1 Step
I2SO.14 = E1 Direction
```

### Pinos GPIO

```
GPIO.15 = Spindle PWM
GPIO.22 = Limit Z+
GPIO.32 = Limit Y-
GPIO.33 = Limit X-
GPIO.34 = SD Card Detect
GPIO.35 = MT_DET (Reset/Door)
GPIO.36 = TH1 (Feed Hold)
GPIO.39 = TB (Cycle Start)
```

### Pinos de Comunicação

#### I2S
```
GPIO.25 = BCK (Bit Clock)
GPIO.26 = WS (Word Select)
GPIO.27 = DATA
```

#### SPI (SD Card)
```
GPIO.5  = CS (Chip Select)
GPIO.18 = SCK (Clock)
GPIO.19 = MISO (Master In)
GPIO.23 = MOSI (Master Out)
```

## Esquema de Conexão Típico

### Exemplo de Máquina 3 Eixos

```
┌─────────────────────────────────────────────┐
│           MKS TinyBee V1.0                  │
│                                             │
│  [X] ───────────> Motor NEMA17 (X)         │
│  [Y] ───────────> Motor NEMA17 (Y)         │
│  [Z] ───────────> Motor NEMA23 (Z)         │
│                                             │
│  [X-Limit] ────> Switch X                   │
│  [Y-Limit] ────> Switch Y                   │
│  [Z-Limit] ────> Switch Z                   │
│                                             │
│  [PWM] ─────────> Spindle/VFD              │
│                                             │
│  [Power] <────── 24V 15A PSU               │
└─────────────────────────────────────────────┘
```

## Configuração dos Drivers TMC2209

### Ajuste de Corrente

Os drivers TMC2209 permitem ajuste de corrente via UART, mas a placa possui potenciômetros para ajuste manual:

**Fórmula para tensão de referência:**
```
Vref = Irms × 0.4
```

**Exemplos:**
- Motor de 1.0A RMS: Vref = 0.4V
- Motor de 1.5A RMS: Vref = 0.6V
- Motor de 2.0A RMS: Vref = 0.8V

**Nota:** A corrente de pico é aproximadamente 1.41× a corrente RMS.

### Microstepping

O FluidNC configura automaticamente os TMC2209 para:
- **Padrão:** 1/16 microstepping
- **StealthChop2:** Operação silenciosa até velocidades médias
- **SpreadCycle:** Automaticamente em velocidades altas

## Proteções e Limitações

### Proteções da Placa
- Fusível resetável na entrada de alimentação
- Proteção de sobrecorrente nos drivers
- Proteção térmica nos TMC2209

### Limitações
- **Corrente máxima por motor:** 2A RMS (2.8A pico)
- **Tensão máxima:** 24V
- **Temperatura operacional:** 0-50°C
- **Dissipação térmica:** Considere ventilação para correntes > 1.5A

## Dimensões

- **Tamanho:** 110mm × 85mm
- **Montagem:** 4 furos M3 nos cantos
- **Altura com componentes:** ~15mm

## Checklist de Instalação

- [ ] Verificar tensão da fonte (12-24V)
- [ ] Conectar motores respeitando polaridade
- [ ] Configurar jumper J1 (tensão PWM)
- [ ] Configurar jumper J2 (SD card detect)
- [ ] Conectar endstops nos terminais corretos
- [ ] Verificar continuidade dos cabos
- [ ] Ajustar Vref dos drivers (se necessário)
- [ ] Conectar spindle/PWM
- [ ] Instalar cartão SD formatado (FAT32)
- [ ] Conectar USB para programação
- [ ] Alimentar a placa (última etapa)

## Manutenção

### Preventiva
- Verificar aperto dos terminais de parafuso mensalmente
- Limpar poeira acumulada
- Verificar temperatura dos drivers durante operação
- Verificar conectores e cabos

### Solução de Problemas Físicos

**Motor não gira:**
- Verificar conexão do motor
- Verificar se driver está habilitado
- Medir tensão nos terminais do motor

**Aquecimento excessivo:**
- Reduzir corrente do driver
- Melhorar ventilação
- Verificar se motor não está preso

**Ruído nos motores:**
- Ajustar configuração de microstepping
- Verificar aterramento
- Ajustar parâmetros de interpolação

## Recursos Adicionais

- [Esquemático oficial](https://github.com/makerbase-mks/MKS-TinyBee/tree/main/hardware)
- [Datasheet TMC2209](https://www.trinamic.com/fileadmin/assets/Products/ICs_Documents/TMC2209_Datasheet_V103.pdf)
- [ESP32 Technical Reference](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)
