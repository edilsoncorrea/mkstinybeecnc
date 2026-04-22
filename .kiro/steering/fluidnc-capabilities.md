---
inclusion: always
---

# FluidNC — Capacidades e Limitações na MKS TinyBee V1.0

Este documento descreve o que é possível e o que não é possível fazer com o FluidNC na MKS TinyBee V1.0. Use como referência ao planejar configurações complexas.

Fonte: análise do código-fonte em https://github.com/bdring/FluidNC

---

## Eixos e Motores

### Capacidades
- **Até 6 eixos** nomeados X, Y, Z, A, B, C
- **Até 2 motores por eixo** (`motor0` e `motor1`) — permite tandem/pórtico duplo
- Cada motor pode ter seu próprio endstop independente (auto-squaring)
- Inversão de direção por software com `!` no `direction_pin`
- `limit_neg_pin`, `limit_pos_pin` e `limit_all_pin` por motor
- `shared_stepper_disable_pin` — um único pino para desabilitar todos os motores
- `homing_runs` — número de ciclos approach/pulloff (padrão: 1, recomendado: 2 para precisão)

### Limitações na TinyBee
- **5 drivers físicos** (X, Y, Z, E0, E1) — máximo de 5 motores simultâneos
- Com 3 eixos + 2 motores tandem = todos os 5 slots ocupados, sem sobrar para 4º eixo
- Com 3 eixos + 1 motor tandem = 1 slot livre para 4º eixo
- **Não há pinos de endstop dedicados para X+, Y+, Z-** — requer GPIOs livres para endstops adicionais
- GPIOs livres disponíveis para endstops extras: gpio.2, gpio.4, gpio.13, gpio.14, gpio.16, gpio.17, gpio.21

### Configuração de tandem (dois motores no mesmo eixo)

```yaml
axes:
  y:
    motor0:
      limit_neg_pin: gpio.32:low:pu    # endstop Y lado A
      stepstick:
        step_pin: I2SO.4
        direction_pin: I2SO.5
        disable_pin: I2SO.3
    motor1:
      limit_neg_pin: gpio.13:low:pu    # endstop Y lado B (GPIO livre)
      stepstick:
        step_pin: I2SO.10              # slot E0
        direction_pin: I2SO.11
        disable_pin: I2SO.9
```

---

## Endstops / Limites

### Capacidades
- `limit_neg_pin` — endstop no lado negativo do eixo
- `limit_pos_pin` — endstop no lado positivo do eixo
- `limit_all_pin` — um único pino que aciona para qualquer direção
- `hard_limits: true/false` — para imediatamente ou apenas registra
- Suporte a sensores NO e NC (`:low` ou `:high`)
- Pull-up interno (`:pu`) e pull-down (`:pd`)
- Cada motor pode ter endstop independente (essencial para auto-squaring)

### Limitações na TinyBee
- Apenas 3 conectores físicos dedicados de endstop: X- (gpio.33), Y- (gpio.32), Z+ (gpio.22)
- Para endstops adicionais (X+, Y+, Z-, ou segundo endstop Y para tandem), é necessário usar GPIOs livres com fiação adaptada
- GPIOs 34, 35, 36, 39 são input-only — podem ser usados como endstop mas não como saída

---

## Spindle / Ferramentas

### Tipos suportados pelo FluidNC
- `PWM` — controle por duty cycle (ESC, VFD com entrada PWM)
- `Laser` — PWM otimizado para laser (com `off_on_alarm: true`)
- `Relay` — liga/desliga simples
- `VFD` — controle Modbus RTU (requer UART)
- `BESC` — ESC de brushless com protocolo específico
- `10V` — saída 0-10V via DAC do ESP32
- `Huanyang` — VFD Huanyang via Modbus
- `H2A` — VFD H2A via Modbus
- `YL620` — VFD YL620 via Modbus
- `Nowforever` — VFD Nowforever via Modbus

### Múltiplos spindles
O FluidNC suporta múltiplos spindles com `tool_num` diferente. Troca via `M6 Tx`.

```yaml
PWM:
  tool_num: 0
  speed_map: 0=0% 24000=100%
  output_pin: gpio.15:high

Laser:
  tool_num: 1
  speed_map: 0=0% 1000=100%
  output_pin: gpio.2:high:pd
```

### Parâmetros adicionais do spindle (não documentados nos configs básicos)
- `off_on_alarm: true/false` — desliga spindle em caso de alarme
- `M6_macro: G28` — GCode executado ao trocar ferramenta (M6)
- `direction_pin` — pino para controle de direção (CW/CCW)
- `enable_pin` — pino de enable separado do PWM
- `disable_with_s0: true/false` — desabilita enable_pin quando S0

### Limitações na TinyBee
- Pino PWM padrão: gpio.15 (pode dar pulsos no boot — risco de ativar spindle)
- Alternativa recomendada: gpio.17 (sem pulsos no boot)
- Para VFD Modbus: requer UART livre (gpio.16/17 para TX/RX)
- DAC (saída 0-10V) usa gpio.25 ou gpio.26 — **ambos reservados para I2S na TinyBee** — não disponível

---

## Coolant / Relés / Saídas Digitais

### Coolant (M7/M8/M9)
```yaml
coolant:
  flood_pin: i2so.16    # M8 — Heated Bed terminal
  mist_pin: i2so.17     # M7 — HE0 terminal
  delay_ms: 0
```

### User Outputs (M62-M68) — saídas genéricas via GCode
Permite controlar saídas digitais e analógicas diretamente do GCode:

```yaml
user_outputs:
  digital0_pin: gpio.XX    # M62/M63/M64/M65 P0
  digital1_pin: gpio.XX    # M62/M63/M64/M65 P1
  digital2_pin: gpio.XX    # M62/M63/M64/M65 P2
  digital3_pin: gpio.XX    # M62/M63/M64/M65 P3
  analog0_pin: gpio.XX     # M67/M68 E0 (PWM)
  analog0_hz: 5000         # frequência PWM
```

**Comandos GCode:**
- `M62 P0` — liga saída digital 0 (sincronizado com movimento)
- `M63 P0` — desliga saída digital 0 (sincronizado)
- `M64 P0` — liga imediatamente
- `M65 P0` — desliga imediatamente
- `M67 E0 Q50` — define saída analógica 0 a 50% duty (sincronizado)
- `M68 E0 Q50` — define imediatamente

**Uso típico:** controle de válvulas, ventoinhas, fixadores, servos RC.

### User Inputs — entradas genéricas
```yaml
user_inputs:
  digital0_pin: gpio.XX
  analog0_pin: gpio.XX
```

### Limitações na TinyBee
- GPIOs livres para saídas digitais: gpio.2, gpio.4, gpio.13, gpio.14, gpio.16, gpio.17, gpio.21
- GPIOs 34, 35, 36, 39 são input-only — não podem ser saídas digitais
- Saídas analógicas (PWM) podem usar qualquer GPIO de saída livre

---

## Probe / Sensor de Altura

### Capacidades
- `probe.pin` — sensor de probe principal (G38.x)
- `probe.toolsetter_pin` — segundo sensor para troca automática de ferramenta
- `probe.check_mode_start` — verifica estado do probe ao iniciar
- `probe.hard_stop` — para imediatamente ao acionar probe

```yaml
probe:
  pin: gpio.35:low
  toolsetter_pin: gpio.XX:low:pu    # GPIO livre para toolsetter
  check_mode_start: true
  hard_stop: false
```

### Limitações na TinyBee
- gpio.35 (MT_DET) é o pino padrão para probe — input-only, sem pull-up interno
- Se usar reset_pin em gpio.35, o probe não pode usar o mesmo pino simultaneamente
- Para toolsetter, precisa de GPIO livre adicional

---

## Controles (Botões)

### Capacidades
```yaml
control:
  safety_door_pin: NO_PIN
  reset_pin: gpio.35:low
  feed_hold_pin: gpio.36:low
  cycle_start_pin: gpio.39:low
  macro0_pin: NO_PIN    # executa Macro0 ao acionar
  macro1_pin: NO_PIN
  macro2_pin: NO_PIN
  macro3_pin: NO_PIN
  fault_pin: NO_PIN     # entrada de falha externa
  estop_pin: NO_PIN     # e-stop externo
```

### Macros configuráveis
```yaml
macros:
  startup_line0: G90    # executado no boot
  startup_line1: G21
  Macro0: G28           # executado quando macro0_pin é acionado
  Macro1: M5
  after_homing: G92 X0 Y0 Z0
  after_reset:
  after_unlock:
```

### Limitações na TinyBee
- gpio.35, 36, 39 são input-only — correto para controles
- gpio.34 (SD detect) também é input-only
- Sem pull-up interno nos GPIOs 34-39 — use `:pu` no config para ativar pull-up via software

---

## Homing

### Capacidades
- Ciclos numerados (1, 2, 3...) — mesmo número = simultâneo
- `homing_runs: N` — número de ciclos approach/pulloff (nível `axes:`)
- `positive_direction: true/false` — direção de busca
- `seek_mm_per_min` — velocidade rápida de busca
- `feed_mm_per_min` — velocidade lenta de aproximação
- `settle_ms` — tempo de estabilização
- `seek_scaler` / `feed_scaler` — distância extra de busca
- Auto-squaring automático quando eixo tem `motor0` e `motor1` com endstops independentes

### Sequência típica para máquina com pórtico duplo Y
```yaml
axes:
  homing_runs: 2    # 2 ciclos para melhor precisão
  z:
    homing:
      cycle: 1      # Z primeiro (levanta ferramenta)
  x:
    homing:
      cycle: 2      # X segundo
  y:
    homing:
      cycle: 3      # Y terceiro (com auto-squaring se motor1 tiver endstop)
```

---

## Parking

### Capacidades
Move o eixo Z para posição segura durante feed hold ou safety door:

```yaml
parking:
  enable: true
  axis: Z
  target_mpos_mm: -5.000        # posição de parking (machine coords)
  rate_mm_per_min: 800.000      # velocidade de subida
  pullout_distance_mm: 5.000    # distância de recuo antes de subir
  pullout_rate_mm_per_min: 250.000
```

---

## Configurações Globais

### Parâmetros de nível raiz
```yaml
arc_tolerance_mm: 0.002         # tolerância de arco G2/G3
junction_deviation_mm: 0.010    # desvio em junções (afeta velocidade em cantos)
verbose_errors: true            # mensagens de erro detalhadas
report_inches: false            # reportar em polegadas
planner_blocks: 16              # blocos do planejador de movimento
enable_parking_override_control: false
use_line_numbers: false
meta: "texto livre"             # metadados da configuração
```

---

## Resumo: Slots Disponíveis na TinyBee para Configurações Complexas

| Recurso | Disponível | Observação |
|---------|-----------|------------|
| Motores | 5 | X, Y, Z, E0, E1 |
| Eixos independentes | até 5 | se 1 motor por eixo |
| Motores tandem | até 2 | usa E0 e/ou E1 |
| Endstops dedicados | 3 | X- (33), Y- (32), Z+ (22) |
| Endstops extras | 7 | GPIOs livres: 2,4,13,14,16,17,21 |
| Spindle PWM | 1 | gpio.15 ou gpio.17 |
| Spindle Laser | 1 | gpio.2 (3D Touch) |
| Coolant/Relés | 2 | i2so.16, i2so.17 |
| User Outputs digitais | até 7 | GPIOs livres |
| User Outputs analógicos | até 7 | GPIOs livres |
| Probe | 1 | gpio.35 |
| Toolsetter | 1 | GPIO livre |
| Controles (botões) | 3 fixos + 4 macro | 35, 36, 39 + GPIOs livres |
| VFD Modbus | 1 | requer UART (gpio.16/17) |
| DAC 0-10V | ❌ | gpio.25/26 reservados para I2S |
