---
inclusion: always
---

# FluidNC — Regras de Configuração e Validação

Este documento define as regras de negócio que governam o `config.yaml` para a MKS TinyBee V1.0. Use estas regras ao analisar, sugerir ou validar qualquer alteração de configuração.

---

## Regras de Pinos — Conflitos Proibidos

### R1 — Unicidade de GPIO
Cada GPIO físico só pode ter **uma função** em todo o config. Se o mesmo `gpio.XX` aparecer em dois lugares do config, isso é um conflito crítico — **o FluidNC rejeita o config e não carrega o motor1**.

**Isso inclui:** mesmo GPIO em motor0 e motor1 do mesmo eixo (ex: limit_pos_pin compartilhado). O FluidNC trata cada motor como uma entidade separada e exige GPIOs únicos.

**Exceção conhecida e documentada:** `gpio.35` é compartilhado entre `probe.pin` e `control.reset_pin` no config atual. Apenas um deve estar ativo (o outro deve ser `NO_PIN` ou comentado).

### R2 — GPIOs Input-Only
Os GPIOs 34, 35, 36 e 39 são **input-only** no ESP32. Nunca devem ser usados como saída (spindle, coolant, relés, etc.).

### R3 — GPIOs Reservados do Sistema
Os seguintes GPIOs nunca devem ser usados para nenhuma função de usuário:
- `gpio.25`, `gpio.26`, `gpio.27` — barramento I2S
- `gpio.18`, `gpio.19`, `gpio.23` — barramento SPI
- `gpio.5` — CS do SD Card
- `gpio.6` a `gpio.11` — flash interna do ESP32

### R4 — Unicidade de I2SO
Cada pino I2SO só pode ter uma função. Os pinos I2SO.0–I2SO.14 são para motores. I2SO.16 e I2SO.17 são para coolant/relés.

---

## Regras de Eixos

### R5 — Mapeamento Correto de I2SO por Eixo
Cada eixo deve usar o conjunto correto de pinos I2SO:

| Eixo | Enable | Step   | Direction |
|------|--------|--------|-----------|
| X    | I2SO.0 | I2SO.1 | I2SO.2    |
| Y    | I2SO.3 | I2SO.4 | I2SO.5    |
| Z    | I2SO.6 | I2SO.7 | I2SO.8    |
| E0   | I2SO.9 | I2SO.10| I2SO.11   |
| E1   | I2SO.12| I2SO.13| I2SO.14   |

Usar pinos de outro eixo causa comportamento imprevisível.

### R6 — Dois Motores no Mesmo Eixo (Tandem)
Para usar dois motores em um eixo (ex: Y com dois motores para pórtico duplo):
- O primeiro motor é `motor0` com os pinos do eixo Y (I2SO.3/4/5)
- O segundo motor é `motor1` usando os pinos do slot E0 (I2SO.9/10/11) ou E1 (I2SO.12/13/14)
- Ambos ficam dentro do mesmo bloco `axes.y`
- O `motor1` pode ter seu próprio `limit_neg_pin` para endstop independente

**Exemplo de Y com dois motores e endstops individuais:**
```yaml
axes:
  y:
    motor0:
      limit_neg_pin: gpio.32:low:pu   # endstop Y lado esquerdo
      stepstick:
        step_pin: I2SO.4
        direction_pin: I2SO.5
        disable_pin: I2SO.3
    motor1:
      limit_neg_pin: gpio.XX:low:pu   # endstop Y lado direito (GPIO livre)
      stepstick:
        step_pin: I2SO.10             # usando slot E0
        direction_pin: I2SO.11
        disable_pin: I2SO.9
```

### R7 — Endstops para Tandem (Dual Motor)
Quando um eixo usa dois motores, existem dois modos de operação:

**Modo 1 — Auto-squaring (2 endstops):**
Cada motor tem seu próprio `limit_neg_pin` ou `limit_pos_pin` apontando para GPIOs diferentes. Permite correção individual de posição.

**Modo 2 — POG style (1 endstop):**
Apenas motor0 tem endstop. O FluidNC detecta `motorsWithSwitches() == 1` e usa `makeDualSwitches()` para compartilhar o endstop com motor1 via máscara de bits. Ambos os motores param juntos.

**Warning cosmético:** No modo POG, o FluidNC emite `[MSG:WARN:: Motor1 switches do not support negative homing dir]`. Este warning é **cosmético** — a lógica do eixo aceita homing se pelo menos um motor suporta a direção (usa `break` ao encontrar o primeiro). O homing funciona normalmente.

**Fonte:** `FluidNC/src/Machine/Axis.cpp` linhas 54-76, `Motor.cpp` linhas 77-83.

**IMPORTANTE:** Não confundir o warning com falha de homing. Se Y não move após `$H`, verificar:
1. Status com `?` — se está em Alarm, usar `$X` para unlock
2. Se `must_home: true` e o homing falhou por outro motivo (ex: endstop não aciona), a máquina trava
3. Testar com `$X` + `G1 Y10 F500` para isolar se é problema de homing vs motor/driver

**Soluções para eliminar o warning:**
1. Adicionar endstop ao motor1 (ex: `limit_neg_pin: gpio.13:low:pu`) — ativa auto-squaring
2. Ignorar o warning — POG funciona com endstop único
3. Remover motor1 se o segundo motor não está conectado fisicamente

---

## Regras de Valores

### R8 — steps_per_mm
- Deve ser maior que zero
- Valores típicos: 40–80 (correias GT2), 200–640 (fuso de esferas), 800–1600 (fuso trapezoidal)
- Valores acima de 2000 são incomuns e merecem revisão
- Fórmula: `(passos_motor × microsteps) / (pitch × redução)`

### R9 — max_rate_mm_per_min
- Deve ser maior que zero
- Deve ser maior que `seek_mm_per_min` e `feed_mm_per_min` do homing
- Valores típicos: 3000–12000 mm/min
- Valores acima de 15000 mm/min são incomuns para máquinas hobby

### R10 — acceleration_mm_per_sec2
- Deve ser maior que zero
- Valores típicos: 30–150 mm/s²
- Eixo Z geralmente tem aceleração menor que X e Y

### R11 — Ciclos de Homing
- O ciclo 1 é executado primeiro (geralmente Z para levantar a ferramenta)
- Eixos com mesmo número de ciclo fazem homing simultaneamente
- Valores típicos: Z=1, X=2, Y=3

### R12 — pulloff_mm
- Deve ser maior que zero
- Deve ser menor que `max_travel_mm`
- Valores típicos: 2–10 mm

### R12b — soft_limits e range de movimento válido
Quando `soft_limits: true`, o FluidNC calcula o range permitido a partir de `mpos_mm` e `max_travel_mm`:

- **Regra:** O range válido é `[mpos_mm, mpos_mm + max_travel_mm]` quando `positive_direction: false` (homing no lado negativo, curso na direção positiva)
- **Regra:** O range válido é `[mpos_mm - max_travel_mm, mpos_mm]` quando `positive_direction: true` (homing no lado positivo, curso na direção negativa)
- Comando GCode que resulte em posição **exatamente igual** ao limite pode ser rejeitado — use margem de 1-2mm

**IMPORTANTE — G54 e soft_limits:**
O soft_limits avalia a posição **de máquina** (MPos), não a posição de trabalho (WPos). Se há um offset G54 ativo (ex: G54 Y=-320), um `G1 Y10` calcula MPos target = WPos + offset = 10 + (-320) = -310, que pode cair fora do range. Sintomas: `[MSG:INFO: Soft limit on Y target:-310.000]` quando o comando parece válido.

**Para macros de parking/posicionamento pós-homing, sempre usar `G53`** (coordenadas de máquina, one-shot), que ignora o G54:
```
$H
G53 G0 Y330
G53 G0 Z-50
```

**Requisitos do G53 no FluidNC (confirmado no código-fonte GCode.cpp):**
- G53 requer `G0` ou `G1` explícito **na mesma linha** — sem isso retorna erro `G53 invalid motion mode`
- G53 NÃO aplica offsets (G54/G92/TLO) ao target — usa coordenada de máquina direta
- Se o soft_limit reporta um target inesperado (ex: +4 quando mandou -40), o G53 não foi processado — o offset G54 está sendo aplicado
- **Possíveis causas de G53 não funcionar:**
  1. CNCjs quebrando a linha em duas (G53 separado do G0)
  2. Estado de Alarm impedindo parsing de GCode
  3. Modo modal ativo diferente de G0/G1 (ex: G38.x após probe)
- **Workaround:** Enviar `G0` sozinho na linha anterior para garantir modo modal, ou usar `G0 G53 Z-40`

Alternativa: zerar G54 com `G10 L2 P1 X0 Y0 Z0` antes dos movimentos.

**Fórmulas de range (confirmadas em FluidNC/src/Limit.cpp):**
```
limitsMaxPosition = positiveDirection ? mpos : mpos + maxTravel
limitsMinPosition = positiveDirection ? mpos - maxTravel : mpos
```

**Exemplo prático (Y no projeto atual):**
- `positive_direction: false` + `mpos_mm: 0` + `max_travel_mm: 334`
- Range válido: Y de 0 a ~333mm (334 pode ser rejeitado por ser o limite exato)
- `G53 G0 Y330` funciona sempre — não depende de G54

**Exemplo Z:**
- `positive_direction: true` + `mpos_mm: 0` + `max_travel_mm: 80`
- Range válido: Z de -80 a 0mm
- `G53 G0 Z-50` funciona sempre

---

## Regras de Spindle

### R13 — Pino de Saída do Spindle
- Deve ser um GPIO de saída (não input-only)
- Recomendado: `gpio.17` (evita pulsos no boot)
- Alternativo: `gpio.15` (pode dar pulsos curtos no boot — risco de ativar spindle)
- Nunca usar GPIOs 34, 35, 36, 39

### R14 — speed_map
- Deve começar com `0=0%`
- O valor máximo de RPM deve corresponder a `100%`
- Exemplo válido: `0=0.000% 24000=100.000%`

---

## Regras de Coolant / Relés

### R15 — Pinos de Coolant
- Devem usar I2SO.16 (flood) e I2SO.17 (mist) — conectores Heated Bed e HE0
- Não usar GPIOs físicos para coolant sem adaptação de hardware

---

## Análise de Impacto de Mudanças

Ao avaliar uma mudança de hardware ou configuração, sempre responda:

1. **Qual GPIO/I2SO será afetado?** — Está livre? Está reservado?
2. **Há conflito com outro uso?** — Verificar toda a config atual
3. **Qual conector físico na placa?** — Onde o cabo vai conectar
4. **Precisa de adaptação de hardware?** — Jumper, cabo extra, adaptador
5. **Qual o risco se configurado errado?** — Colisão, dano ao motor, falha de homing
6. **O que muda no config.yaml?** — Bloco exato que precisa ser alterado
7. **Precisa testar antes de usar em produção?** — Procedimento de teste recomendado

---

## Checklist de Validação do config.yaml

Antes de fazer upload de qualquer config.yaml para a placa, verificar:

- [ ] Nenhum GPIO repetido em funções diferentes
- [ ] GPIOs input-only (34, 35, 36, 39) usados apenas como entrada
- [ ] GPIOs reservados do sistema não reutilizados
- [ ] Pinos I2SO corretos para cada eixo
- [ ] `steps_per_mm` > 0 em todos os eixos ativos
- [ ] `max_travel_mm` > `pulloff_mm` em todos os eixos
- [ ] Ciclos de homing definidos para todos os eixos com endstop
- [ ] Spindle usando GPIO de saída válido
- [ ] Blocos fixos (i2so, spi, stepping) não alterados
- [ ] Sintaxe YAML válida (indentação com 2 espaços, sem tabs)
