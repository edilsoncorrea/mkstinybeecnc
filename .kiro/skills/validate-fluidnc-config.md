---
name: validate-fluidnc-config
description: Valida o config.yaml do FluidNC para a MKS TinyBee V1.0, verificando conflitos de pinos, valores fora de range e inconsistências de wiring.
inclusion: manual
---

# Skill: Validar config.yaml FluidNC

Quando ativado, analise o arquivo `config.yaml` (ou o arquivo especificado) aplicando todas as regras definidas em `fluidnc-config-rules.md` e o mapa de hardware em `fluidnc-hardware.md`.

## Procedimento de Validação

### 1. Leitura do arquivo
Leia o arquivo config.yaml completo antes de qualquer análise.

### 2. Inventário de pinos
Monte uma tabela de todos os GPIOs e I2SOs usados no arquivo, com sua função:

| Pino | Função | Localização no config |
|------|--------|----------------------|
| ... | ... | ... |

### 3. Verificações obrigatórias

Execute cada verificação e classifique como:
- 🔴 **CRÍTICO** — pode causar dano ao hardware ou comportamento perigoso
- 🟡 **AVISO** — pode causar mau funcionamento
- 🔵 **INFO** — sugestão de melhoria

**Verificações de pinos:**
- [ ] R1: Algum GPIO físico aparece em mais de uma função? (exceto gpio.35 documentado)
- [ ] R2: GPIOs input-only (34, 35, 36, 39) usados como saída?
- [ ] R3: GPIOs reservados do sistema usados (25, 26, 27, 18, 19, 23, 5, 6-11)?
- [ ] R4: Algum I2SO usado em mais de uma função?

**Verificações de eixos:**
- [ ] R5: Cada eixo usa o conjunto correto de I2SOs?
- [ ] R6/R7: Se há motor1 em algum eixo, os pinos são do slot correto (E0 ou E1)?

**Verificações de valores:**
- [ ] R8: steps_per_mm > 0 em todos os eixos? Valores suspeitos (>2000)?
- [ ] R9: max_rate_mm_per_min > seek_mm_per_min e > feed_mm_per_min?
- [ ] R10: acceleration_mm_per_sec2 > 0?
- [ ] R11: Ciclos de homing fazem sentido (Z=1 primeiro)?
- [ ] R12: pulloff_mm < max_travel_mm?

**Verificações de spindle:**
- [ ] R13: output_pin não é input-only?
- [ ] R14: speed_map começa com 0=0%?

**Verificações de coolant:**
- [ ] R15: flood_pin e mist_pin usam i2so.16/17?

**Verificações de blocos fixos:**
- [ ] i2so usa gpio.25/27/26?
- [ ] spi usa gpio.19/23/18?
- [ ] stepping.engine é I2S_STATIC?

### 4. Relatório final

Apresente:
1. Resumo: X críticos, Y avisos, Z infos
2. Lista detalhada de cada problema encontrado com:
   - Regra violada
   - Localização exata no config
   - Explicação do risco
   - Correção sugerida
3. Se nenhum problema: confirmação de que o config passou em todas as verificações
