# Início Rápido - MKS TinyBee + FluidNC

Guia rápido para colocar sua CNC em funcionamento o mais rápido possível.

## ⚡ Checklist Rápido

### Antes de Começar

- [ ] MKS TinyBee V1.0
- [ ] Fonte 12-24V (mínimo 15A recomendado)
- [ ] Cabo USB Type-C
- [ ] Motores NEMA 17 ou 23
- [ ] Cartão microSD formatado FAT32 (opcional, mas recomendado)
- [ ] Endstops/sensores de limite (opcional para início)

### Passo 1: Hardware (15 minutos)

1. **NÃO conecte a fonte ainda**
2. Conecte os motores nos conectores X, Y, Z
3. Se tiver endstops, conecte-os (pode fazer depois)
4. Configure jumper J2 em posição SDDET (se usar SD card)
5. Configure jumper J1:
   - **5V** para ESC/controladores lógicos
   - **VIN** para spindle 24V

### Passo 2: Firmware (10 minutos)

1. Conecte USB ao computador (sem alimentação externa ainda)
2. Abra navegador Chrome/Edge
3. Acesse: **https://install.fluidnc.com/**
4. Clique em "Connect" e selecione a porta
5. Clique em "INSTALL FLUIDNC"
6. Aguarde finalizar (3-5 min)

### Passo 3: Configuração (20 minutos)

1. Copie `config.yaml` para seu editor favorito
2. Ajuste os valores obrigatórios:

```yaml
# AJUSTE ESTES VALORES:
axes:
  x:
    steps_per_mm: 80.000      # Calcule para seu sistema!
    max_travel_mm: 400.000    # Curso máximo em mm
  y:
    steps_per_mm: 80.000      # Calcule para seu sistema!
    max_travel_mm: 300.000    # Curso máximo em mm
  z:
    steps_per_mm: 1600.000    # Calcule para seu sistema!
    max_travel_mm: 100.000    # Curso máximo em mm
```

3. **Cálculo rápido de steps_per_mm:**

**Para correias GT2 com polia 20 dentes:**
```
steps_per_mm = (200 × 16) / 40 = 80
```

**Para fuso T8 (2mm pitch):**
```
steps_per_mm = (200 × 16) / 2 = 1600
```

### Passo 4: Upload da Configuração (5 minutos)

**Opção A - Via Wi-Fi:**
1. Procure rede Wi-Fi "FluidNC"
2. Senha: `12345678`
3. Acesse: http://192.168.0.1
4. Aba "Files" > Upload config.yaml
5. Reset da placa

**Opção B - Via Serial:**
1. Abra terminal serial (115200 baud)
2. Digite: `$LocalFS/Upload`
3. Cole todo conteúdo do config.yaml
4. Pressione Ctrl+D
5. Digite: `$RST`

### Passo 5: Primeiro Teste (10 minutos)

**⚠️ IMPORTANTE: Desconecte correias/fusos dos motores para teste inicial!**

1. **Agora sim**, conecte a fonte de alimentação 12-24V
2. LED PWR deve acender
3. Conecte via WebUI ou serial
4. Digite: `$X` (unlock)
5. Teste cada motor:

```gcode
G91          ; Modo relativo
G0 X10 F500  ; Move X 10mm
G0 Y10 F500  ; Move Y 10mm
G0 Z5 F200   ; Move Z 5mm
```

6. Verifique se motores giram (direção não importa agora)

### Passo 6: Ajustes (conforme necessário)

**Motor gira na direção errada?**

Adicione `!` no direction_pin:
```yaml
direction_pin: !I2SO.2
```

**Motor não gira?**
- Verifique se steps_per_mm não é zero
- Verifique conexão do motor
- Tente outro motor para testar

**Motor vibra mas não gira?**
- Reduza velocidade: `G0 X10 F100`
- Reduza steps_per_mm temporariamente
- Verifique fiação do motor

### Passo 7: Calibração (30 minutos)

1. Reconecte as correias/fusos
2. Marque posição inicial com fita
3. Comando: `G91 G0 X100 F1000`
4. Meça distância real percorrida
5. Ajuste steps_per_mm:

```
steps_novo = steps_atual × (100 / distância_medida)
```

6. Repita para Y e Z

### Passo 8: Endstops (se instalados)

1. Teste manualmente:
```
?
```
Acione cada endstop e veja se aparece letra maiúscula (X, Y, Z)

2. Se não funcionar, ajuste:
```yaml
limit_neg_pin: gpio.33:high:pu  # Troque :low por :high
```

3. Teste homing:
```gcode
$X
$H
```

## 🎯 Comandos Essenciais

```bash
$X              # Unlock/destravar máquina
$H              # Homing (se configurado)
?               # Status em tempo real
$$              # Listar configurações
$I              # Info do sistema
$RST            # Reset/reiniciar
M3 S12000       # Liga spindle a 50%
M5              # Desliga spindle
```

## 🚨 Em Caso de Emergência

**Máquina não para:**
1. Botão de emergência físico (se instalado)
2. Desconecte alimentação
3. Via software: `!` (exclamação) ou Ctrl+X

**Alarme HARD_LIMIT:**
1. Desconecte alimentação
2. Mova manualmente para longe do limite
3. Conecte novamente
4. Digite: `$X` para unlock

## 📱 Conectividade

### Modo Access Point (padrão)
- SSID: FluidNC
- Senha: 12345678
- IP: http://192.168.0.1

### Modo Station (conectar à sua rede)
```
$WiFi/SSID=SuaRede
$WiFi/Password=SuaSenha
$WiFi/Mode=STA
$RST
```

Descubra o IP via serial: `$I`

## 📖 Próximos Passos

1. ✅ Leia o [README completo](README.md)
2. ✅ Estude [Configuração detalhada](docs/configuracao.md)
3. ✅ Configure spindle conforme seu hardware
4. ✅ Ajuste velocidades e acelerações
5. ✅ Teste com arquivo G-code simples
6. ✅ Instale sender (bCNC, UGS, CNCjs)

## 🆘 Problemas?

Consulte [Solução de Problemas](docs/troubleshooting.md) para ajuda detalhada.

## ⏱️ Tempo Total Estimado

- Hardware: 15 min
- Firmware: 10 min
- Configuração: 20 min
- Testes: 10 min
- Calibração: 30 min
- **Total: ~1h30min**

---

**Dica:** Não apresse! Cada passo bem feito evita problemas depois.
