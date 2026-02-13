# Solução de Problemas

Este guia cobre os problemas mais comuns ao usar FluidNC com MKS TinyBee e suas soluções.

## 🔌 Problemas de Conexão

### Placa não é reconhecida no USB

**Sintomas:**
- Computador não detecta dispositivo USB
- Nenhuma porta COM/tty aparece
- LED da placa não acende

**Possíveis causas:**
1. Cabo USB defeituoso ou apenas para carga
2. Driver CP210x não instalado
3. Porta USB sem energia suficiente
4. Problema de alimentação da placa

**Soluções:**

1. **Teste o cabo USB:**
   - Use cabo USB com fios de dados
   - Teste com outro cabo conhecido
   - Comprimento máximo recomendado: 1.5m

2. **Instale drivers CP210x:**
   - Windows/Mac: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers
   - Linux: Geralmente já incluído no kernel

3. **Teste outra porta USB:**
   - Use porta USB traseira (não frontal)
   - Evite hubs USB não alimentados
   - Desconecte outros dispositivos USB

4. **Verifique alimentação:**
   - LED PWR deve acender ao conectar USB
   - Se não acender, problema na placa ou fonte

### Wi-Fi Access Point não aparece

**Sintomas:**
- Rede "FluidNC" não aparece
- Impossível conectar via Wi-Fi

**Soluções:**

1. **Verifique primeira inicialização:**
   - Aguarde 30-60 segundos após boot
   - Procure rede "FluidNC" ou "FluidNC-XXXX"

2. **Force modo AP via serial:**
   ```
   $WiFi/Mode=AP
   $RST
   ```

3. **Verifique configuração:**
   ```
   $I
   ```
   Deve mostrar: `WiFi Mode: AP`

4. **Reset de fábrica:**
   - Mantenha GPIO 0 conectado ao GND
   - Ligue a placa
   - Solte após 5 segundos

### WebUI não carrega

**Sintomas:**
- http://192.168.0.1 não abre
- Timeout ou erro de conexão

**Soluções:**

1. **Verifique conexão Wi-Fi:**
   - Certifique-se de estar conectado à rede FluidNC
   - Sem acesso à internet nesta rede (esperado)

2. **Teste outros endereços:**
   - http://fluidnc.local
   - http://192.168.0.1:80

3. **Limpe cache do navegador:**
   - Ctrl+Shift+Del
   - Tente modo anônimo

4. **Teste via telnet:**
   ```bash
   telnet 192.168.0.1 23
   ```
   Se conectar, problema é no navegador/HTTP

5. **Verifique sistema de arquivos:**
   - Conecte via serial
   - `$LocalFS/List`
   - Se não mostrar arquivos, sistema corrompido

## ⚡ Problemas com Motores

### Motor não se move

**Sintomas:**
- Comando enviado mas motor não responde
- Sem ruído ou vibração

**Diagnóstico:**

1. **Verifique unlock:**
   ```
   $X
   ```

2. **Teste habilitação:**
   ```gcode
   $Motors/Enable
   G91
   G0 X1 F100
   ```

3. **Verifique LED do driver:**
   - Driver deve ter LED indicador aceso
   - Se apagado, problema no enable

**Soluções:**

1. **Problema de configuração:**
   - Verifique `steps_per_mm` não é zero
   - Confirme pins corretos no config.yaml

2. **Problema de fiação:**
   - Teste continuidade dos cabos
   - Verifique polaridade do motor (não importa muito)
   - Teste com outro motor

3. **Driver desabilitado:**
   - Verifique `disable_pin` no config
   - Teste inverte lógica: `!I2SO.0`

### Motor vibra mas não gira

**Sintomas:**
- Motor treme/vibra
- Ruído mas sem movimento
- Perde passos imediatamente

**Causas comuns:**
1. `steps_per_mm` muito alto
2. Velocidade muito alta
3. Aceleração muito alta
4. Corrente do driver muito baixa
5. Fiação incorreta

**Soluções:**

1. **Teste com valores conservadores:**
   ```yaml
   steps_per_mm: 40.000
   max_rate_mm_per_min: 1000.000
   acceleration_mm_per_sec2: 20.000
   ```

2. **Verifique fiação do motor:**
   - 4 fios conectados corretamente
   - Pares de bobinas corretos
   - Use multímetro para identificar pares

3. **Ajuste corrente do driver:**
   - Aumente Vref gradualmente
   - Vref = Irms × 0.4
   - Não exceda corrente nominal do motor

4. **Reduza velocidade:**
   ```gcode
   G91
   G0 X10 F100  # Muito lento
   ```

### Motor gira na direção errada

**Sintomas:**
- Comando X+ move para X-
- Homing vai para direção errada

**Solução simples:**

Adicione `!` antes do direction_pin:

```yaml
stepstick:
  step_pin: I2SO.1
  direction_pin: !I2SO.2  # Inverte
  disable_pin: I2SO.0
```

Ou reconecte motor fisicamente (troque pares de fios).

### Motor aquece demais

**Sintomas:**
- Temperatura > 60°C
- Motor quente ao tocar

**Soluções:**

1. **Reduza corrente:**
   - Ajuste Vref para 80% da corrente nominal
   - `Vref = (Irms × 0.8) × 0.4`

2. **Melhore ventilação:**
   - Adicione ventoinhas na caixa de controle
   - Aponte para os drivers

3. **Verifique idle_ms:**
   ```yaml
   stepping:
     idle_ms: 30000  # Desabilita após 30s
   ```

4. **Use StealthChop:**
   - TMC2209 já usa por padrão
   - Menos aquecimento que modos force

### Perda de passos

**Sintomas:**
- Posição incorreta após movimento
- Camadas desalinhadas
- Furos fora de posição

**Diagnóstico:**

1. **Teste movimento lento:**
   ```gcode
   G91
   G0 X100 F500  # 500mm/min
   ```
   Se OK, problema é velocidade/aceleração

2. **Teste carga:**
   - Desconecte correias/fusos
   - Teste só o motor
   - Se OK, problema é mecânico

**Soluções:**

1. **Reduza velocidade:**
   ```yaml
   max_rate_mm_per_min: 5000.000  # Era 10000
   ```

2. **Reduza aceleração:**
   ```yaml
   acceleration_mm_per_sec2: 50.000  # Era 100
   ```

3. **Aumente corrente:**
   - Até 100% da corrente nominal
   - Monitore temperatura

4. **Problemas mecânicos:**
   - Lubrificação inadequada
   - Correias muito tensionadas
   - Atrito excessivo
   - Desalinhamento

## 🎯 Problemas de Limites/Homing

### Limites não funcionam

**Sintomas:**
- Homing não para no endstop
- `?` não mostra limite acionado
- Máquina continua após acionar limite

**Diagnóstico:**

Teste manual dos limites:
```
?
```

Acione o endstop fisicamente e verifique se aparece letra maiúscula (ex: `X` para X acionado).

**Soluções:**

1. **Verifique fiação:**
   - GND conectado
   - Signal conectado
   - VCC conectado (se necessário)
   - Continuidade com multímetro

2. **Verifique tipo de sensor:**
   - NO (Normalmente Aberto): `gpio.33:low:pu`
   - NC (Normalmente Fechado): `gpio.33:high:pu`

3. **Teste polaridade:**
   - Tente `:low` se estava `:high`
   - Tente `:high` se estava `:low`

4. **Verifique pull-up:**
   - `:pu` = pull-up interno
   - `:pd` = pull-down interno
   - Sensores NPN geralmente precisam `:pu`

### Homing falha

**Sintomas:**
- Homing não encontra limite
- Alarme HARD_LIMIT
- Motor continua batendo

**Soluções:**

1. **Verifique direção:**
   ```yaml
   homing:
     positive_direction: false  # Tente true
   ```

2. **Verifique ciclo:**
   ```yaml
   homing:
     cycle: 1  # Primeiro a fazer homing
   ```

3. **Aumente seek_scaler:**
   ```yaml
   homing:
     seek_scaler: 2.000  # Busca mais longe
   ```

4. **Desabilite temporariamente:**
   ```yaml
   start:
     must_home: false  # Para teste
   ```

### Limite aciona aleatoriamente

**Sintomas:**
- Alarme HARD_LIMIT sem razão
- Limite acionado sem tocar sensor

**Causas:**
- Interferência elétrica
- Fiação mal blindada
- Ruído do spindle/VFD

**Soluções:**

1. **Use cabos blindados:**
   - Malha conectada ao GND
   - Separe de cabos de potência

2. **Adicione capacitor:**
   - 100nF entre signal e GND
   - Próximo ao conector da placa

3. **Filtro RC:**
   - Resistor 1kΩ em série
   - Capacitor 100nF para GND

4. **Desabilite hard_limits:**
   ```yaml
   motor0:
     hard_limits: false  # Só para teste
   ```

## 🔄 Problemas de Spindle

### Spindle não liga

**Sintomas:**
- Comando M3 não liga spindle
- PWM não muda

**Diagnóstico:**

```gcode
M3 S12000  # Liga a 50%
M5         # Desliga
```

Meça tensão no pino PWM com multímetro.

**Soluções:**

1. **Verifique configuração:**
   ```yaml
   spindle:
     PWM:
       output_pin: gpio.15:high  # Ou :low
   ```

2. **Teste polaridade:**
   - `:high` = high por padrão
   - `:low` = low por padrão

3. **Verifique jumper J1:**
   - 5V para ESC/controladores lógicos
   - VIN para 24V (VFDs)

4. **Teste direto:**
   ```
   $Spindle/On=50
   $Spindle/Off
   ```

### Spindle não varia velocidade

**Sintomas:**
- S0 e S24000 têm mesma velocidade
- PWM parece constante

**Soluções:**

1. **Verifique speed_map:**
   ```yaml
   speed_map: 0=0.000% 24000=100.000%
   ```

2. **Teste range:**
   ```gcode
   M3 S1000   # Mínimo
   G4 P3
   M3 S24000  # Máximo
   G4 P3
   M5
   ```

3. **Verifique frequência PWM:**
   ```yaml
   pwm_hz: 2500  # ESC geralmente 1000-5000Hz
   ```

4. **Calibre ESC:**
   - Alguns ESCs precisam calibração
   - Consulte manual do ESC

### Spindle demora a desligar

**Sintomas:**
- M5 não desliga imediatamente
- Atraso de vários segundos

**Esperado:**

```yaml
spindle:
  PWM:
    spindown_ms: 4000  # 4 segundos
```

Para desligar mais rápido:
```yaml
spindown_ms: 1000  # 1 segundo
```

## 💾 Problemas com SD Card

### SD Card não detectado

**Sintomas:**
- Arquivos não aparecem na WebUI
- `$SD/List` retorna erro

**Soluções:**

1. **Verifique jumper J2:**
   - Deve estar em posição SDDET
   - Não em GND

2. **Formate cartão:**
   - FAT32 obrigatório
   - Máximo 32GB
   - Tente outro cartão

3. **Verifique card_detect_pin:**
   ```yaml
   sdcard:
     card_detect_pin: gpio.34:low
   ```

4. **Desabilite detecção:**
   ```yaml
   sdcard:
     card_detect_pin: NO_PIN
   ```

### Arquivo G-code não executa

**Sintomas:**
- Upload OK mas não roda
- Erro ao iniciar arquivo

**Soluções:**

1. **Verifique formato:**
   - Arquivo .nc ou .gcode
   - Texto puro (não binário)
   - Line endings: LF ou CRLF

2. **Teste arquivo simples:**
   ```gcode
   G90
   G0 X10 Y10
   G0 X0 Y0
   M30
   ```

3. **Execute via comando:**
   ```
   $SD/Run=teste.nc
   ```

## 🖥️ Problemas de Software

### Firmware não inicia após flash

**Sintomas:**
- Placa não responde
- Sem saída serial
- LEDs estranhos

**Soluções:**

1. **Erase completo:**
   ```bash
   esptool.py --port COM3 erase_flash
   ```

2. **Re-flash com endereços corretos:**
   ```
   0x1000  bootloader.bin
   0x8000  partitions.bin
   0x10000 fluidnc.bin
   ```

3. **Teste outro firmware:**
   - Baixe versão estável anterior
   - Versões Beta podem ter bugs

### Config.yaml não carrega

**Sintomas:**
- Placa inicia mas usa configuração padrão
- Erros de sintaxe no boot

**Diagnóstico:**

```
$LocalFS/Show=config.yaml
```

Mostra o conteúdo do arquivo.

**Soluções:**

1. **Valide sintaxe YAML:**
   - Use https://www.yamllint.com/
   - Verifique indentação (2 espaços)
   - Sem tabs

2. **Verifique caracteres:**
   - UTF-8 sem BOM
   - Sem caracteres especiais

3. **Re-upload:**
   ```
   $LocalFS/Upload
   [Cole config.yaml]
   Ctrl+D
   ```

### Comandos não funcionam

**Sintomas:**
- GCode ignorado
- Resposta "error:..." 

**Soluções:**

1. **Unlock a máquina:**
   ```
   $X
   ```

2. **Verifique modo:**
   ```
   $G
   ```
   Deve mostrar modo atual (G90, G91, etc)

3. **Veja último erro:**
   Mensagens `[MSG:ERR: ...]` explicam problemas

## 📊 Problemas de Performance

### Movimentos irregulares

**Sintomas:**
- Vibração excessiva
- Ruído durante movimento
- Acabamento ruim

**Soluções:**

1. **Reduza aceleração:**
   ```yaml
   acceleration_mm_per_sec2: 40.000
   ```

2. **Suavização no CAM:**
   - Aumente tolerância de arco
   - Use G2/G3 em vez de segmentos

3. **Ajuste jerk (se disponível):**
   - FluidNC não tem jerk
   - Use aceleração menor

### Perda de conexão Wi-Fi

**Sintomas:**
- WebUI desconecta
- Comandos travados

**Soluções:**

1. **Aproxime da placa:**
   - Sinal Wi-Fi fraco
   - Use antena externa (se disponível)

2. **Reduza interferência:**
   - Afaste de VFDs/spindles
   - Use cabo de rede (futuras versões)

3. **Conexão via USB:**
   - Mais confiável para operação
   - Wi-Fi só para configuração

## 🔧 Ferramentas de Diagnóstico

### Comandos Úteis

```bash
$I              # Informações do sistema
$$              # Configurações EEPROM
$G              # Estados G-code
?               # Status em tempo real
$Limits/Check   # Teste de limites
$Alarm/List     # Lista alarmes
$Error/List     # Lista erros
```

### Modo Verbose

```
$Verbose/Errors=true
$Verbose/Info=true
```

Mostra mais informações de debug.

### Log de Erros

Conecte via serial e salve log:
```bash
screen -L /dev/ttyUSB0 115200
```

## 📞 Obtendo Ajuda

### Informações para Suporte

Ao pedir ajuda, forneça:

1. **Versão do FluidNC:**
   ```
   $I
   ```

2. **Configuração:**
   ```
   $LocalFS/Show=config.yaml
   ```

3. **Erro específico:**
   - Mensagem de erro completa
   - Comando que causou erro

4. **Hardware:**
   - MKS TinyBee V1.0
   - Motores (modelo, corrente)
   - Fonte de alimentação

### Comunidade

- [Discord FluidNC](https://discord.gg/fluidnc)
- [GitHub Discussions](https://github.com/bdring/FluidNC/discussions)
- [Reddit r/hobbycnc](https://reddit.com/r/hobbycnc)

### Recursos

- [FluidNC Wiki](http://wiki.fluidnc.com/)
- [FAQ Oficial](http://wiki.fluidnc.com/en/support/faq)
- [GitHub Issues](https://github.com/bdring/FluidNC/issues)
