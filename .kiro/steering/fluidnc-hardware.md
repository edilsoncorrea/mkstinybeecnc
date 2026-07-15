---
inclusion: always
---

# MKS TinyBee V1.0 — Mapa de Hardware

Este documento é a fonte de verdade sobre o hardware da placa MKS TinyBee V1.0 usada neste projeto. Sempre consulte este mapa antes de sugerir ou validar qualquer alteração no `config.yaml`.

## Plataforma

- **Firmware:** FluidNC (ESP32)
- **Placa:** MKS TinyBee V1.0
- **MCU:** ESP32-WROOM-32 (240MHz dual-core, 4MB Flash, 520KB SRAM)
- **Drivers de motor:** 5x TMC2209 integrados (X, Y, Z, E0, E1)
- **Alimentação:** 12–24V DC

---

## Pinos I2S — Controle de Motores

Os motores são controlados exclusivamente via barramento I2S. Os pinos I2SO são virtuais (não são GPIOs físicos).

| Eixo | Enable   | Step     | Direction |
|------|----------|----------|-----------|
| X    | I2SO.0   | I2SO.1   | I2SO.2    |
| Y    | I2SO.3   | I2SO.4   | I2SO.5    |
| Z    | I2SO.6   | I2SO.7   | I2SO.8    |
| E0   | I2SO.9   | I2SO.10  | I2SO.11   |
| E1   | I2SO.12  | I2SO.13  | I2SO.14   |

**Regra:** Nunca reutilize um I2SO para dois eixos. Nunca use I2SO para endstops ou controles.

**Inversão de direção em pinos I2SO:** Use `I2SO.X:low` (NÃO use `!I2SO.X` — sintaxe inválida para I2S). Exemplo: `direction_pin: I2SO.2:low`

### Pinos físicos do barramento I2S

| Função   | GPIO    |
|----------|---------|
| BCK      | gpio.25 |
| WS       | gpio.26 |
| DATA     | gpio.27 |

Estes três pinos são **reservados** e não podem ser usados para nenhuma outra função.

---

## GPIOs Físicos — Mapa Completo

### Endstops / Limites

| Função  | GPIO    | Conector físico | Observação                        |
|---------|---------|-----------------|-----------------------------------|
| X-      | gpio.33 | X-LIMIT         | Limite negativo do eixo X         |
| Y-      | gpio.32 | Y-LIMIT         | Limite negativo do eixo Y         |
| Z+      | gpio.22 | Z-LIMIT         | Limite positivo do eixo Z         |

**Pinos de limite disponíveis mas não usados por padrão:**
- Não há conector dedicado para X+, Y+, Z- na TinyBee V1.0
- Para endstops adicionais (ex: Y1 independente), é necessário usar GPIOs livres via adaptação

### Controles

| Função       | GPIO    | Conector físico | Observação                        |
|--------------|---------|-----------------|-----------------------------------|
| Reset/Door   | gpio.35 | MT_DET          | Input-only, sem pull-up interno   |
| Feed Hold    | gpio.36 | TH1             | Input-only, sem pull-up interno   |
| Cycle Start  | gpio.39 | TB              | Input-only, sem pull-up interno   |

> ⚠️ **GPIOs 34, 35, 36, 39 são input-only** no ESP32. Não podem ser usados como saída.

### Probe

| Função | GPIO    | Conector físico |
|--------|---------|-----------------|
| Probe  | gpio.35 | MT_DET          |

> Nota: gpio.35 é compartilhado entre Reset e Probe no config atual. Apenas um pode ser ativo por vez.

### SD Card

| Função       | GPIO    | Observação                              |
|--------------|---------|-----------------------------------------|
| CS           | gpio.5  | SPI Chip Select                         |
| Card Detect  | gpio.34 | Input-only. Requer jumper J2 em SDDET   |

### SPI (SD Card)

| Função | GPIO    |
|--------|---------|
| MISO   | gpio.19 |
| MOSI   | gpio.23 |
| SCK    | gpio.18 |

Estes pinos são **reservados para o SD Card** e não devem ser reutilizados.

### Spindle / Saídas

| Função      | GPIO    | Conector físico | Observação                          |
|-------------|---------|-----------------|-------------------------------------|
| Spindle PWM | gpio.15 | EXP1 IO15       | Pode dar pulsos curtos no boot      |
| Alternativo | gpio.17 | EXP1 IO17       | Preferível para evitar pulsos boot  |

### Saídas via I2S (Coolant/Relés)

| Função | I2SO    | Conector físico | Observação              |
|--------|---------|-----------------|-------------------------|
| Flood  | i2so.16 | Heated Bed      | Saída de relé/coolant   |
| Mist   | i2so.17 | HE0             | Saída de relé/coolant   |

---

## GPIOs Livres (disponíveis para expansão)

Os seguintes GPIOs não têm função atribuída por padrão e podem ser usados para expansões (endstops adicionais, relés, etc.):

| GPIO    | Tipo | Status atual                                         |
|---------|------|------------------------------------------------------|
| gpio.2  | I/O  | **EM USO** — Z- endstop (conector 3D Touch)          |
| gpio.4  | I/O  | **EM USO** — X+ endstop (fiação adaptada)            |
| gpio.13 | I/O  | **EM USO** — Y- motor1 endstop (fiação adaptada)     |
| gpio.14 | I/O  | **EM USO** — Y+ endstop compartilhado (fiação adapt.)|
| gpio.16 | I/O  | Disponível                                           |
| gpio.17 | I/O  | Disponível — alternativa ao gpio.15 para spindle     |
| gpio.21 | I/O  | Disponível                                           |

> ⚠️ GPIOs 6–11 são usados pela flash interna do ESP32 e **nunca devem ser usados**.

---

## Jumpers

| Jumper | Posição  | Efeito                                      |
|--------|----------|---------------------------------------------|
| J1     | 5V       | Saída PWM em 0–5V (para ESC)                |
| J1     | VIN      | Saída PWM em 0–24V (para VFD)               |
| J2     | SDDET    | Habilita detecção de SD Card (recomendado)  |
| J2     | GND      | Desabilita detecção de SD Card              |

---

## Capacidade de Eixos

O FluidNC suporta até **6 eixos** (X, Y, Z, A, B, C) e até **2 motores por eixo** (`motor0` e `motor1`).

| Slot driver | Uso padrão | Uso alternativo                                  |
|-------------|------------|--------------------------------------------------|
| X           | Eixo X     | —                                                |
| Y           | Eixo Y     | —                                                |
| Z           | Eixo Z     | —                                                |
| E0          | Disponível | `motor1` do eixo X ou Y (tandem), ou 4º eixo (A) |
| E1          | Disponível | `motor1` do eixo X ou Y (tandem), ou 5º eixo (B) |

**Máximo de 5 drivers na TinyBee** — com 3 eixos + 2 motores tandem, todos os slots estão ocupados.

Para usar dois motores em um eixo (ex: Y com dois motores independentes para auto-squaring), o segundo motor usa o slot E0 ou E1 como `motor1` dentro do mesmo bloco de eixo.

### Parâmetros globais de eixos (nível `axes:`)

```yaml
axes:
  shared_stepper_disable_pin: NO_PIN  # Pino único para desabilitar todos os motores
  shared_stepper_reset_pin: NO_PIN    # Pino único de reset para todos os drivers
  homing_runs: 1                      # Número de ciclos approach/pulloff no homing
```

### Parâmetros de eixo com `idle_disable`

Cada eixo suporta `idle_disable: true/false` para controlar se o motor é desabilitado quando ocioso (além do `idle_ms` global do stepping).

---

## Configuração Fixa (não alterar)

Estes blocos do `config.yaml` são determinados pelo hardware da TinyBee e **não devem ser modificados**:

```yaml
i2so:
  bck_pin: gpio.25
  data_pin: gpio.27
  ws_pin: gpio.26

spi:
  miso_pin: gpio.19
  mosi_pin: gpio.23
  sck_pin: gpio.18

stepping:
  engine: I2S_STATIC
```
