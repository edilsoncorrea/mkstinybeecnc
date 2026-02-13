# Guia de Configuração FluidNC

Este guia explica em detalhes como configurar o arquivo `config.yaml` para a MKS TinyBee com FluidNC.

## Estrutura do Arquivo config.yaml

O FluidNC utiliza arquivos YAML para configuração. O YAML é sensível a indentação (use 2 espaços, não tabs).

### Seções Principais

1. **board** - Identificação da placa
2. **name** - Nome da sua máquina
3. **kinematics** - Tipo de cinemática
4. **i2so** - Configuração I2S para motores
5. **spi** - Configuração SPI para SD card
6. **sdcard** - Configuração do cartão SD
7. **stepping** - Parâmetros de controle de passo
8. **axes** - Configuração dos eixos
9. **control** - Entradas de controle
10. **spindle** - Configuração do spindle
11. **start** - Comandos de inicialização

## Configuração Básica

### Header do Arquivo

```yaml
board: MKS TinyBee V1.0 XXYYZ
name: MinhaCooltoolCNC
```

- **board**: Identificação da placa (não alterar)
- **name**: Nome personalizado da sua máquina

### Cinemática

```yaml
kinematics:
  Cartesian:
```

Tipos disponíveis:
- `Cartesian`: XYZ tradicional
- `CoreXY`: Sistema CoreXY
- `WallPlotter`: Plotter de parede
- `SCARA`: Braço SCARA

### Configuração I2S

```yaml
i2so:
  bck_pin: gpio.25
  data_pin: gpio.27
  ws_pin: gpio.26
```

**Não altere estes valores** - são específicos da MKS TinyBee.

### Configuração SPI e SD Card

```yaml
spi:
  miso_pin: gpio.19
  mosi_pin: gpio.23
  sck_pin: gpio.18

sdcard:
  cs_pin: gpio.5
  card_detect_pin: gpio.34:low
```

**Importante**: Certifique-se de que o jumper J2 está em posição SDDET.

### Parâmetros de Stepping

```yaml
stepping:
  engine: I2S_STATIC
  idle_ms: 255
  pulse_us: 4
  dir_delay_us: 1
  disable_delay_us: 2
```

**Parâmetros:**
- `engine`: Tipo de engine (I2S_STATIC para MKS TinyBee)
- `idle_ms`: Tempo antes de desabilitar motores (255 = nunca)
- `pulse_us`: Duração do pulso de step (4µs é adequado)
- `dir_delay_us`: Delay após mudar direção (1µs)
- `disable_delay_us`: Delay antes de desabilitar (2µs)

## Configuração dos Eixos

A configuração de cada eixo é a parte mais importante e requer ajustes específicos para sua máquina.

### Estrutura de um Eixo

```yaml
axes:
  x:
    steps_per_mm: 40.000
    max_rate_mm_per_min: 8000.000
    acceleration_mm_per_sec2: 80.000
    max_travel_mm: 2500.000
    soft_limits: false
    
    homing:
      cycle: 2
      positive_direction: false
      mpos_mm: 0.000
      feed_mm_per_min: 300.000
      seek_mm_per_min: 1500.000
      settle_ms: 500
      seek_scaler: 1.100
      feed_scaler: 1.100
    
    motor0:
      limit_neg_pin: gpio.33:low:pu
      hard_limits: true
      pulloff_mm: 4.000
      
      stepstick:
        step_pin: I2SO.1
        direction_pin: I2SO.2
        disable_pin: I2SO.0
```

### Parâmetros do Eixo

#### steps_per_mm

**Fórmula:**
```
steps_per_mm = (motor_steps × microstepping) / (pitch × redução)
```

**Exemplos práticos:**

**Pinhão e correia (GT2):**
- Motor: 200 steps/rev (1.8°)
- Microstepping: 16
- Polia: 20 dentes GT2 (pitch 2mm)
- Redução: 1:1

```
steps_per_mm = (200 × 16) / (20 × 2) = 80
```

**Fuso de esferas:**
- Motor: 200 steps/rev
- Microstepping: 16
- Pitch do fuso: 5mm
- Redução: 1:1

```
steps_per_mm = (200 × 16) / 5 = 640
```

**Fuso trapezoidal:**
- Motor: 200 steps/rev
- Microstepping: 16
- Pitch do fuso: 2mm (T8)
- Redução: 1:1

```
steps_per_mm = (200 × 16) / 2 = 1600
```

#### max_rate_mm_per_min

Velocidade máxima do eixo em mm/min.

**Recomendações:**
- Correias: 8000-12000 mm/min
- Fusos de esferas: 5000-8000 mm/min
- Fusos trapezoidais: 3000-5000 mm/min

**Como determinar:**
1. Comece com valor conservador (3000)
2. Teste movimentos rápidos (G0)
3. Aumente gradualmente até perder passos
4. Use 70-80% do valor máximo encontrado

#### acceleration_mm_per_sec2

Aceleração do eixo em mm/s².

**Recomendações:**
- Eixos leves (X, Y com correias): 80-150 mm/s²
- Eixos pesados (pórtico): 50-80 mm/s²
- Eixo Z: 30-60 mm/s²

**Fatores que afetam:**
- Peso da estrutura
- Rigidez mecânica
- Torque do motor
- Atrito do sistema

#### max_travel_mm

Curso máximo do eixo em mm. Meça fisicamente o deslocamento total.

#### soft_limits

- `true`: FluidNC previne movimentos além do max_travel
- `false`: Sem verificação por software (use hard limits)

### Configuração de Homing

```yaml
homing:
  cycle: 2
  positive_direction: false
  mpos_mm: 0.000
  feed_mm_per_min: 300.000
  seek_mm_per_min: 1500.000
  settle_ms: 500
  seek_scaler: 1.100
  feed_scaler: 1.100
```

**Parâmetros:**

- **cycle**: Ordem de homing (1=primeiro, 2=segundo, 3=terceiro)
  - Típico: Z=1, X=2, Y=3
  
- **positive_direction**: 
  - `true`: Procura limite na direção positiva (+)
  - `false`: Procura limite na direção negativa (-)
  
- **mpos_mm**: Posição de máquina após homing (geralmente 0.000)

- **seek_mm_per_min**: Velocidade rápida de busca do limite

- **feed_mm_per_min**: Velocidade lenta de aproximação

- **settle_ms**: Tempo de estabilização após acionar limite

- **seek_scaler**: Distância extra na busca (1.1 = 110%)

- **feed_scaler**: Distância extra na aproximação (1.1 = 110%)

### Configuração do Motor

```yaml
motor0:
  limit_neg_pin: gpio.33:low:pu
  hard_limits: true
  pulloff_mm: 4.000
  
  stepstick:
    step_pin: I2SO.1
    direction_pin: I2SO.2
    disable_pin: I2SO.0
```

**Parâmetros:**

- **limit_neg_pin**: Pino do limite negativo
  - Formato: `gpio.XX:low:pu`
  - `:low` = acionado em nível baixo
  - `:pu` = pull-up interno habilitado
  
- **limit_pos_pin**: Pino do limite positivo (se houver)

- **hard_limits**: 
  - `true`: Para imediatamente ao acionar limite
  - `false`: Apenas registra, não para

- **pulloff_mm**: Distância de recuo após homing

### Inversão de Direção

Se o motor gira na direção errada, adicione `!` antes do pino:

```yaml
direction_pin: !I2SO.2  # Inverte direção
```

## Pinagem dos Eixos na MKS TinyBee

### Eixo X
```yaml
stepstick:
  step_pin: I2SO.1
  direction_pin: I2SO.2
  disable_pin: I2SO.0
```
Limite: `gpio.33` (X-)

### Eixo Y
```yaml
stepstick:
  step_pin: I2SO.4
  direction_pin: I2SO.5
  disable_pin: I2SO.3
```
Limite: `gpio.32` (Y-)

### Eixo Z
```yaml
stepstick:
  step_pin: I2SO.7
  direction_pin: I2SO.8
  disable_pin: I2SO.6
```
Limite: `gpio.22` (Z+)

### Eixo E0 (4º eixo)
```yaml
stepstick:
  step_pin: I2SO.10
  direction_pin: I2SO.11
  disable_pin: I2SO.9
```

### Eixo E1 (5º eixo)
```yaml
stepstick:
  step_pin: I2SO.13
  direction_pin: I2SO.14
  disable_pin: I2SO.12
```

## Controles

```yaml
control:
  safety_door_pin: NO_PIN
  reset_pin: gpio.35:low
  feed_hold_pin: gpio.36:low
  cycle_start_pin: gpio.39:low
```

**Funções:**
- **safety_door**: Pausa ao abrir porta/tampa
- **reset**: Reset/Emergency stop
- **feed_hold**: Pausa temporária
- **cycle_start**: Inicia/resume ciclo

**Nota**: Se não usar, defina como `NO_PIN`.

## Configuração do Spindle

### Spindle PWM

```yaml
spindle:
  PWM:
    pwm_hz: 2500
    output_pin: gpio.15:high
    s0_with_disable: true
    tool_num: 0
    spinup_ms: 4000
    spindown_ms: 4000
    speed_map: 0=0.000% 24000=100.000%
```

**Parâmetros:**

- **pwm_hz**: Frequência PWM (2500Hz típico para ESC)

- **output_pin**: Pino de saída (`gpio.15` na TinyBee)

- **s0_with_disable**: 
  - `true`: S0 desliga completamente
  - `false`: S0 mantém velocidade mínima

- **spinup_ms**: Tempo de aceleração (ms)

- **spindown_ms**: Tempo de desaceleração (ms)

- **speed_map**: Mapeamento velocidade/duty cycle
  - Formato: `RPM=duty% RPM=duty%`
  - Exemplo: `0=0% 12000=50% 24000=100%`

### Cálculo do speed_map

**Para ESC:**
- 0 RPM = 0% duty cycle
- RPM máximo = 100% duty cycle

**Para VFD com controle 0-10V:**
Se usar conversor PWM→0-10V:
- 0 RPM = 0%
- RPM máximo do spindle = 100%

**Exemplo prático:**
Spindle de 24000 RPM máximo:
```yaml
speed_map: 0=0% 6000=25% 12000=50% 18000=75% 24000=100%
```

### Spindle Relay

Se usar relé simples (liga/desliga):

```yaml
spindle:
  Relay:
    output_pin: gpio.15
    tool_num: 0
    spinup_ms: 2000
    spindown_ms: 2000
```

### Spindle com VFD (Modbus)

Para controle via Modbus RTU:

```yaml
spindle:
  VFD:
    modbus:
      uart_num: 1
      txd_pin: gpio.17
      rxd_pin: gpio.16
      rts_pin: NO_PIN
      baud: 9600
    tool_num: 0
    spinup_ms: 3000
    spindown_ms: 3000
    speed_map: 0=0% 24000=100%
```

## Comandos de Inicialização

```yaml
start:
  must_home: true
  gcode:
    - G90
    - G92 X0 Y0 Z0
```

**Parâmetros:**

- **must_home**: 
  - `true`: Requer homing antes de usar
  - `false`: Permite uso sem homing (cuidado!)

- **gcode**: Lista de comandos GCode executados após boot
  - `G90`: Modo absoluto
  - `G92`: Zera coordenadas de trabalho

## Exemplos de Configurações Completas

### Router CNC Pequeno (Correia)

```yaml
axes:
  x:
    steps_per_mm: 80.000
    max_rate_mm_per_min: 10000.000
    acceleration_mm_per_sec2: 100.000
    max_travel_mm: 400.000
    soft_limits: false
```

### CNC Médio (Fuso de Esferas)

```yaml
axes:
  x:
    steps_per_mm: 640.000
    max_rate_mm_per_min: 6000.000
    acceleration_mm_per_sec2: 80.000
    max_travel_mm: 800.000
    soft_limits: true
```

### CNC Grande (Fuso Trapezoidal)

```yaml
axes:
  x:
    steps_per_mm: 1600.000
    max_rate_mm_per_min: 4000.000
    acceleration_mm_per_sec2: 50.000
    max_travel_mm: 1500.000
    soft_limits: true
```

## Validação da Configuração

### Verificar Sintaxe YAML

Use validador online: https://www.yamllint.com/

Ou instale yamllint:
```bash
pip install yamllint
yamllint config.yaml
```

### Testar Configuração

1. Upload para a placa
2. Reset
3. Conecte via serial
4. Digite `$$` para ver configurações carregadas
5. Verifique erros no boot

### Comandos de Teste

```gcode
$X              # Unlock
G91             # Relativo
G0 X1 F500      # 1mm em X
G0 Y1 F500      # 1mm em Y
G0 Z0.5 F200    # 0.5mm em Z
```

## Dicas de Ajuste Fino

### Calibração de steps_per_mm

1. Marque posição inicial
2. Mova 100mm: `G91 G0 X100`
3. Meça distância real
4. Ajuste:

```
steps_per_mm_novo = steps_per_mm_atual × (100 / distância_real)
```

### Otimização de Velocidade

1. Comece com 50% da velocidade teórica
2. Faça movimentos rápidos repetidos
3. Aumente 10% por vez
4. Pare quando perder passos
5. Use 80% do valor máximo

### Otimização de Aceleração

1. Comece com valores baixos
2. Execute padrões de zigue-zague
3. Aumente gradualmente
4. Ouça ruídos anormais
5. Procure por perda de passos em cantos

### Tuning do Homing

**Homing muito lento:**
- Aumente `seek_mm_per_min`

**Homing impreciso:**
- Reduza `feed_mm_per_min`
- Aumente `settle_ms`

**Homing bate forte no limite:**
- Reduza `seek_mm_per_min`

## Backup e Versionamento

### Backup Regular

```bash
# Salve com data
cp config.yaml config_2024-01-15.yaml
```

### Git para Versionamento

```bash
git init
git add config.yaml
git commit -m "Configuração inicial"
```

## Recursos Adicionais

- [Referência Completa YAML](http://wiki.fluidnc.com/en/config/overview)
- [Exemplos de Configuração](http://wiki.fluidnc.com/en/config/examples)
- [Calculadora steps_per_mm](https://blog.prusaprinters.org/calculator_3416/)
