# MKS TinyBee CNC - Documentação FluidNC

Documentação completa para construção e configuração de CNC utilizando a controladora MKS TinyBee V1.0 com firmware FluidNC.

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Hardware](#hardware)
- [Instalação do FluidNC](#instalação-do-fluidnc)
- [Configuração](#configuração)
- [Pinagem](#pinagem)
- [Calibração](#calibração)
- [Solução de Problemas](#solução-de-problemas)
- [Recursos Úteis](#recursos-úteis)
- [Ecossistema de Validação com Kiro](#-ecossistema-de-validação-com-kiro)
- [Documentação Detalhada](#documentação-detalhada)

---

## 🎯 Sobre o Projeto

Este repositório documenta o processo de adaptação da controladora MKS TinyBee para uso com o firmware FluidNC, uma alternativa moderna e robusta ao Grbl tradicional. O FluidNC oferece recursos avançados como:

- Suporte a Wi-Fi e interface web integrada
- Configuração via arquivo YAML
- Suporte a múltiplos tipos de máquinas (cartesiana, CoreXY, SCARA, etc.)
- Controle via WebUI, telnet ou serial
- Sistema de arquivos SD Card para armazenar G-code

## 🔧 Hardware

### MKS TinyBee V1.0

A MKS TinyBee é uma placa controladora baseada em ESP32, ideal para CNCs de pequeno a médio porte.

**Especificações principais:**
- MCU: ESP32-WROOM-32
- 5 drivers de motor (X, Y, Z, E0, E1) - TMC2209 integrados
- Alimentação: 12-24V DC
- Conectividade: Wi-Fi, USB, SD Card
- Interface I2S para controle preciso dos motores
- Entradas para endstops/limites
- Saída PWM para spindle
- Entradas para controle (reset, feed hold, cycle start)

### Componentes Recomendados

- **Motores:** NEMA 17 ou NEMA 23 (dependendo do tamanho da máquina)
- **Spindle:** Spindle PWM ou VFD
- **Sensores de limite:** Switches mecânicos ou sensores indutivos
- **Fonte de alimentação:** 24V 15A (mínimo recomendado)

## 💾 Instalação do FluidNC

### 1. Download do Firmware

Baixe a versão mais recente do FluidNC:
```
https://github.com/bdring/FluidNC/releases
```

### 2. Flash do Firmware

**Opção 1 - Web Installer (Recomendado):**
1. Acesse: https://install.fluidnc.com/
2. Conecte a placa via USB
3. Clique em "Connect" e selecione a porta COM
4. Escolha a versão e clique em "Install"

**Opção 2 - ESP Flash Tool:**
1. Baixe o ESP Flash Download Tool
2. Configure:
   - Chip Type: ESP32
   - Flash Mode: DIO
   - Flash Speed: 40MHz
3. Selecione o arquivo .bin
4. Flash na placa

### 3. Verificação

Após o flash, conecte via terminal serial (115200 baud) e verifique se aparece a mensagem de boas-vindas do FluidNC.

## ⚙️ Configuração

Este repositório inclui dois arquivos de configuração:

### config.yaml (Template Base)

Arquivo modelo com marcadores de ajuste (`<ajuste>`, `<seu valor>`). Use este como ponto de partida para sua máquina específica.

**Parâmetros a ajustar:**
- `steps_per_mm`: Calcule baseado em pitch, microsteps e redução
- `max_travel_mm`: Dimensões de trabalho da sua máquina
- `speed_map` do spindle: Conforme especificações do seu spindle

### config2.yaml (Exemplo Configurado)

Exemplo de configuração completa com valores reais. Útil como referência.

### Upload da Configuração

**Via WebUI:**
1. Conecte ao Wi-Fi da placa (AP: "FluidNC" senha: "12345678")
2. Acesse: http://192.168.0.1
3. Vá em "SD Card" > Upload config.yaml

**Via Terminal Serial:**
```
$LocalFS/Upload
[Cole o conteúdo do config.yaml]
[Ctrl+D para finalizar]
```

## 📌 Pinagem

### Eixos e Motores

| Eixo | Step Pin | Direction Pin | Enable Pin | Limit Pin |
|------|----------|---------------|------------|-----------|
| X    | I2SO.1   | I2SO.2        | I2SO.0     | GPIO.33 (X-) |
| Y    | I2SO.4   | I2SO.5        | I2SO.3     | GPIO.32 (Y-) |
| Z    | I2SO.7   | I2SO.8        | I2SO.6     | GPIO.22 (Z+) |
| E0   | I2SO.10  | I2SO.11       | I2SO.9     | - |
| E1   | I2SO.13  | I2SO.14       | I2SO.12    | - |

### Comunicação I2S

- BCK Pin: GPIO.25
- WS Pin: GPIO.26
- DATA Pin: GPIO.27

### SPI (SD Card)

- MISO: GPIO.19
- MOSI: GPIO.23
- SCK: GPIO.18
- CS: GPIO.5
- Card Detect: GPIO.34 (requer jumper J2 em SDDET)

### Controle

- Reset: GPIO.35 (MT_DET)
- Feed Hold: GPIO.36 (TH1)
- Cycle Start: GPIO.39 (TB)

### Spindle

- PWM Output: GPIO.15

## 📐 Calibração

### 1. Cálculo de Steps per MM

```
steps_per_mm = (passos_por_revolução × microsteps) / (pitch × redução)
```

**Exemplo:**
- Motor: 200 passos/revolução (1.8°)
- Microsteps: 16 (configurado no driver)
- Pitch: 2mm (polia GT2 de 20 dentes)
- Redução: 1:1

```
steps_per_mm = (200 × 16) / 2 = 1600 steps/mm
```

### 2. Teste de Movimento

```gcode
G91          ; Modo relativo
G0 X10       ; Move 10mm em X
G0 Y10       ; Move 10mm em Y
G0 Z5        ; Move 5mm em Z
```

Meça o movimento real e ajuste `steps_per_mm` proporcionalmente.

### 3. Ajuste de Aceleração

Comece com valores conservadores:
- Aceleração: 50-100 mm/s²
- Velocidade máxima: 3000-5000 mm/min

Aumente gradualmente até encontrar o limite antes de perder passos.

### 4. Calibração do Spindle

1. Configure `speed_map` no config.yaml (ex: `0=0% 24000=100%`)
2. Teste com diferentes velocidades:
```gcode
M3 S12000    ; 50% da velocidade
M3 S24000    ; 100% da velocidade
M5           ; Desliga spindle
```

## 🔍 Solução de Problemas

### Placa não conecta ao Wi-Fi
- Verifique se o firmware FluidNC foi instalado corretamente
- Procure pela rede "FluidNC" nas redes Wi-Fi disponíveis
- Senha padrão: "12345678"

### Motor vibra mas não se move
- Verifique `steps_per_mm` (valor muito alto/baixo)
- Verifique alimentação da placa (tensão adequada)
- Ajuste `pulse_us` e `dir_delay_us` no config

### Limites não funcionam
- Verifique fiação dos endstops
- Confirme tipo de sensor (NO/NC) no config
- Teste continuidade com multímetro

### SD Card não é detectado
- Verifique jumper J2 na posição SDDET
- Formate o cartão em FAT32
- Use cartões até 32GB

### Movimentos invertidos
- Inverta o pin de direção no config: `!I2SO.2` (adicione !)
- Ou reconfigure fisicamente a fiação do motor

## 📚 Recursos Úteis

### Documentação Oficial

- [FluidNC Wiki](http://wiki.fluidnc.com/)
- [FluidNC - MKS TinyBee](http://wiki.fluidnc.com/en/hardware/3rd-party/MKS_TinyBee)
- [FluidNC GitHub](https://github.com/bdring/FluidNC)

### Comunidade

- [FluidNC Discord](https://discord.gg/fluidnc)
- [Fórum FluidNC](https://github.com/bdring/FluidNC/discussions)

### Hardware

- [MKS TinyBee - GitHub](https://github.com/makerbase-mks/MKS-TinyBee)
- [Documentação MKS](https://github.com/makerbase-mks)

### Vídeos

- [MKS TinyBee Setup](https://www.youtube.com/watch?v=mEYtFK7AFOs)

---

## 📖 Documentação Detalhada

Este repositório inclui documentação completa e detalhada em português:

### 🚀 [Início Rápido](docs/inicio-rapido.md)
Coloque sua CNC funcionando em ~1h30min com este guia passo-a-passo simplificado.

### 🔧 [Hardware](docs/hardware.md)
- Especificações técnicas completas da MKS TinyBee
- Layout da placa e conectores
- Diagrama de pinagem detalhado
- Configuração dos drivers TMC2209
- Esquemas de conexão

### 💾 [Instalação](docs/instalacao.md)
- Três métodos de instalação do firmware
- Configuração inicial via Wi-Fi ou serial
- Troubleshooting de instalação
- Comandos de atualização

### ⚙️ [Configuração](docs/configuracao.md)
- Explicação detalhada do config.yaml
- Cálculo de steps_per_mm para diferentes sistemas
- Configuração de homing e limites
- Setup de spindle (PWM, relay, VFD)
- Exemplos práticos

### 📌 [Referência de Pinagem](docs/pinagem-referencia.md)
- Mapa completo de pinos GPIO
- Templates para config.yaml
- Diagrama visual da placa
- Calculadoras rápidas
- Guia de troubleshooting rápido

### 🔍 [Solução de Problemas](docs/troubleshooting.md)
- Problemas de conexão (USB, Wi-Fi, WebUI)
- Problemas com motores
- Problemas de limites/homing
- Problemas de spindle
- Problemas com SD Card
- Ferramentas de diagnóstico

### 🔗 [Recursos Úteis](docs/recursos-uteis.md)
- Links para documentação oficial
- Ferramentas e software recomendados
- Comunidades e fóruns
- Vídeos e tutoriais
- Calculadoras online
- Fornecedores

---

## 🤖 Ecossistema de Validação com Kiro

Este repositório inclui um ecossistema completo para edição assistida e validação do `config.yaml` usando o [Kiro IDE](https://kiro.dev). O objetivo é evitar erros de wiring — como conflitos de GPIO, pinos incorretos por eixo, ou valores fora de range — antes de fazer upload para a placa.

### Como funciona

O Kiro carrega automaticamente o contexto de hardware da MKS TinyBee e as regras de configuração do FluidNC em todas as interações. Isso permite que o assistente raciocine sobre implicações de mudanças de hardware com precisão.

### Arquivos do ecossistema

#### Steering (contexto sempre ativo)

| Arquivo | Descrição |
|---------|-----------|
| `.kiro/steering/fluidnc-hardware.md` | Fonte de verdade do hardware: todos os GPIOs, I2SOs, conectores físicos, jumpers e GPIOs livres da TinyBee |
| `.kiro/steering/fluidnc-config-rules.md` | 15 regras de validação: unicidade de GPIO, pinos input-only, I2SOs corretos por eixo, ranges de valores |
| `.kiro/steering/fluidnc-capabilities.md` | Capacidades e limitações completas do FluidNC na TinyBee, baseadas no código-fonte oficial |

#### Skills (capacidades sob demanda)

| Skill | Como usar | O que faz |
|-------|-----------|-----------|
| `validate-fluidnc-config` | `#validate-fluidnc-config` no chat | Auditoria completa do config.yaml com relatório de críticos 🔴, avisos 🟡 e infos 🔵 |
| `hardware-change-impact` | `#hardware-change-impact` no chat | Dado "quero adicionar endstops individuais no Y", analisa pinos, riscos, gera diff exato do config e procedimento de teste |

#### Hook automático

Ao salvar qualquer `config*.yaml`, o Kiro executa automaticamente a validação e reporta problemas no chat.

#### Schema YAML

O arquivo `schemas/tinybee-config-validator.yaml-schema` fornece:
- Autocomplete para todas as seções do config
- Descrições inline de cada parâmetro
- Validação de valores (ranges, constantes fixas, padrões de pinos I2SO)

Requer a extensão **YAML (Red Hat)** no VS Code / Kiro. Já configurado em `.vscode/settings.json`.

### Exemplos de uso

**Validar o config atual:**
```
#validate-fluidnc-config
```

**Analisar impacto de uma mudança:**
```
#hardware-change-impact
Quero adicionar endstops individuais para os dois motores do eixo Y (auto-squaring)
```

**Perguntar sobre capacidades:**
```
Posso usar VFD Modbus e spindle PWM ao mesmo tempo nessa placa?
```

### O que o ecossistema conhece

Com base na análise do [código-fonte do FluidNC](https://github.com/bdring/FluidNC):

- Mapeamento completo de todos os GPIOs físicos e I2SOs da TinyBee
- Quais GPIOs são input-only (34, 35, 36, 39) e quais são reservados pelo sistema
- Capacidade de até 6 eixos e 2 motores por eixo (tandem/auto-squaring)
- Todos os tipos de spindle suportados (PWM, Laser, Relay, VFD Modbus, Huanyang, etc.)
- User Outputs/Inputs para controle de relés e válvulas via GCode (M62-M68)
- Macros configuráveis (`after_homing`, `after_reset`, `startup_line`)
- Parking automático do eixo Z
- Limitações específicas da TinyBee (DAC 0-10V indisponível, VFD Modbus consome gpio.16/17, etc.)

---

## 📝 Notas

- Este documento está em constante atualização
- Sempre faça backup da sua configuração antes de modificar
- Teste movimentos com a máquina desligada primeiro (dry run)
- Comece com velocidades e acelerações baixas

## 🤝 Contribuindo

Sinta-se à vontade para abrir issues ou pull requests com melhorias, correções ou adições à documentação.

## 📄 Licença

Este projeto de documentação é disponibilizado como está, para uso pessoal e educacional.
