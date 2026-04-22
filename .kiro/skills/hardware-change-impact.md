---
name: hardware-change-impact
description: Analisa o impacto de uma mudança de hardware na CNC — como adicionar endstops, segundo motor, relé ou trocar spindle — e gera o diff exato do config.yaml necessário.
inclusion: manual
---

# Skill: Análise de Impacto de Mudança de Hardware

Quando ativado com uma descrição de mudança de hardware, execute a análise completa abaixo.

## Exemplos de uso
- "Quero adicionar endstops individuais para os dois motores do eixo Y"
- "Quero adicionar um segundo motor no eixo Y para pórtico duplo"
- "Quero trocar o spindle PWM por um relé"
- "Quero adicionar um sensor de probe"
- "Quero usar gpio.17 em vez de gpio.15 para o spindle"

## Procedimento de Análise

### 1. Entender a mudança
Identifique claramente:
- O que está sendo adicionado/modificado/removido
- Qual componente físico está envolvido (motor, sensor, relé, etc.)

### 2. Consultar o mapa de hardware
Com base em `fluidnc-hardware.md`, determine:
- Qual GPIO ou I2SO será necessário
- Se esse pino está livre ou ocupado
- Qual conector físico na placa será usado

### 3. Verificar conflitos
Com base no `config.yaml` atual e nas regras de `fluidnc-config-rules.md`:
- O pino necessário já está em uso?
- A mudança viola alguma regra?
- Há restrições de hardware (input-only, reservado)?

### 4. Relatório de impacto

Apresente em seções:

#### Hardware físico necessário
- Componentes adicionais (cabos, conectores, adaptadores)
- Onde conectar na placa (conector específico)
- Jumpers a configurar (se aplicável)

#### Pinos envolvidos
| Pino | Status atual | Novo uso | Conflito? |
|------|-------------|----------|-----------|
| ... | ... | ... | ... |

#### Riscos
- O que pode dar errado se configurado incorretamente
- Nível de risco: 🔴 Alto / 🟡 Médio / 🟢 Baixo

#### Alterações no config.yaml
Mostre o diff exato — apenas os blocos que mudam:

```yaml
# ANTES
...

# DEPOIS
...
```

#### Procedimento de teste
Passos para verificar que a mudança funcionou corretamente antes de usar em produção.

### 5. Perguntas de esclarecimento
Se a mudança for ambígua, faça perguntas específicas antes de prosseguir. Exemplos:
- "O segundo motor Y vai ter endstop próprio ou compartilhar com o motor0?"
- "O sensor é NO (normalmente aberto) ou NC (normalmente fechado)?"
- "Qual GPIO livre você quer usar para o novo endstop?"
