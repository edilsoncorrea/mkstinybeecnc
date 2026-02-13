# Guia de Instalação FluidNC

Este guia fornece instruções detalhadas para instalar e configurar o firmware FluidNC na controladora MKS TinyBee.

## Pré-requisitos

### Hardware
- MKS TinyBee V1.0
- Cabo USB Type-C
- Fonte 12-24V DC (15A recomendado)
- Cartão microSD (até 32GB, formatado em FAT32)

### Software
- Navegador web moderno (Chrome, Edge, Opera)
- OU ESP Flash Download Tool (alternativa)
- Terminal serial (opcional): PuTTY, CoolTerm, ou Arduino IDE Serial Monitor

## Método 1: Web Installer (Recomendado)

Este é o método mais simples e rápido.

### Passo 1: Preparar a Placa

1. **Desconecte** a alimentação da placa
2. **Conecte** o cabo USB ao computador
3. A placa deve ser reconhecida como porta serial/COM

**Verificar porta no Windows:**
```
Gerenciador de Dispositivos > Portas (COM & LPT)
Procure por "USB Serial Device" ou "CP210x"
```

**Verificar porta no Linux:**
```bash
ls /dev/ttyUSB*
# ou
dmesg | grep tty
```

**Verificar porta no macOS:**
```bash
ls /dev/cu.usbserial*
```

### Passo 2: Acessar o Web Installer

1. Abra o navegador (Chrome/Edge/Opera)
2. Acesse: **https://install.fluidnc.com/**
3. A página deve mostrar o instalador FluidNC

### Passo 3: Conectar à Placa

1. Clique no botão **"Connect"**
2. Uma janela popup aparecerá listando portas seriais
3. Selecione a porta da MKS TinyBee
4. Clique em **"Conectar"**

**Nota:** Se não aparecer nenhuma porta:
- Tente outro cabo USB (alguns são apenas para carga)
- Instale drivers CP210x se necessário
- Tente outra porta USB do computador

### Passo 4: Instalar Firmware

1. Selecione a versão mais recente (ex: "v3.7.x")
2. Clique em **"INSTALL FLUIDNC"**
3. Aguarde o processo (2-5 minutos)
4. Uma mensagem de sucesso aparecerá ao concluir

### Passo 5: Verificação

1. Clique em **"Connect"** novamente
2. Abra o console/terminal
3. Digite `$I` e pressione Enter
4. Deve aparecer informações do sistema:

```
[MSG:INFO: FluidNC v3.7.x]
[MSG:INFO: Compiled with ESP32 SDK:...]
[MSG:INFO: Board: MKS TinyBee V1.0]
```

## Método 2: ESP Flash Download Tool

Para usuários avançados ou quando o Web Installer não funciona.

### Passo 1: Download dos Arquivos

1. Baixe a versão mais recente:
   - Acesse: https://github.com/bdring/FluidNC/releases
   - Baixe: `fluidnc-vX.X.X.zip`

2. Extraia o arquivo ZIP
3. Localize os arquivos .bin:
   - `bootloader.bin`
   - `partitions.bin`
   - `fluidnc.bin`

### Passo 2: Download da Ferramenta

1. Baixe o ESP Flash Download Tool:
   - https://www.espressif.com/en/support/download/other-tools
   
2. Extraia e execute `flash_download_tool_xxx.exe`

### Passo 3: Configuração

1. Selecione **ESP32** como chip type
2. Configure os arquivos e endereços:

```
bootloader.bin     @ 0x1000
partitions.bin     @ 0x8000
fluidnc.bin        @ 0x10000
```

3. Configurações adicionais:
   - **SPI Speed:** 40MHz
   - **SPI Mode:** DIO
   - **Flash Size:** 4MB

### Passo 4: Flash

1. Selecione a porta COM
2. Clique em **START**
3. Aguarde até aparecer **FINISH**

## Método 3: esptool.py (Linux/macOS)

Para usuários de linha de comando.

### Instalação do esptool

```bash
pip install esptool
```

### Download do Firmware

```bash
wget https://github.com/bdring/FluidNC/releases/download/v3.7.x/fluidnc-v3.7.x.zip
unzip fluidnc-v3.7.x.zip
cd fluidnc-v3.7.x
```

### Apagar Flash Anterior

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash
```

### Flash do Firmware

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 921600 \
  --before default_reset --after hard_reset write_flash -z \
  --flash_mode dio --flash_freq 40m --flash_size detect \
  0x1000 bootloader.bin \
  0x8000 partitions.bin \
  0x10000 fluidnc.bin
```

**Nota:** Substitua `/dev/ttyUSB0` pela sua porta.

## Primeira Conexão

### Via Wi-Fi (Recomendado)

1. **Após o primeiro boot**, a placa cria um Access Point
2. **SSID:** FluidNC
3. **Senha:** 12345678

4. Conecte seu dispositivo a esta rede
5. Acesse: **http://192.168.0.1**
6. A WebUI do FluidNC deve abrir

### Via Serial

1. Abra terminal serial (115200 baud)
2. Conecte na porta COM da placa
3. Digite comandos GCode ou comandos $

**Comandos úteis:**
```
$I           - Informações do sistema
$$           - Mostrar configurações
$G           - Mostrar estados GCode
$X           - Destravar/Unlock
```

## Configuração Inicial

### 1. Upload do Arquivo de Configuração

**Via WebUI:**
1. Acesse a WebUI (http://192.168.0.1)
2. Vá para aba **"SD Card"** ou **"Files"**
3. Clique em **"Upload"**
4. Selecione o arquivo `config.yaml`
5. Aguarde o upload
6. **Reset** a placa (botão físico ou comando `$RST`)

**Via Serial:**
```
$LocalFS/Upload
[Cole todo o conteúdo do config.yaml]
[Pressione Ctrl+D no final]
```

### 2. Configurar Wi-Fi (Opcional)

**Modo Station (conectar à rede existente):**

```
$WiFi/SSID=MinhaRede
$WiFi/Password=MinhaSenha
$WiFi/Mode=STA
```

Após reset, a placa conectará à sua rede. Descubra o IP:
- Verifique no roteador
- Use app de scanner de rede
- Conecte via serial e digite `$I`

### 3. Testar Motores

**ATENÇÃO:** Desconecte as correias/fusos antes de testar!

```gcode
$X           ; Unlock da máquina
G91          ; Modo relativo
G0 X10       ; Move 10mm em X
G0 Y10       ; Move 10mm em Y
G0 Z5        ; Move 5mm em Z
```

Observe se os motores giram na direção correta.

### 4. Verificar Limites

Acione cada endstop manualmente e execute:
```
?
```

O status deve mostrar os limites acionados (letras maiúsculas X, Y, Z).

## Atualizações Futuras

### Via WebUI (Mais Fácil)

1. Baixe o novo firmware (.bin)
2. Na WebUI, vá em **"System"** > **"Update"**
3. Selecione o arquivo .bin
4. Clique em **"Update"**
5. Aguarde reinicialização

### Via Serial (OTA)

```
$Firmware/Update
[Cole a URL do firmware ou envie o arquivo]
```

## Solução de Problemas

### Placa não é reconhecida no USB

**Possíveis causas:**
1. Cabo USB defeituoso ou apenas para carga
2. Driver CP210x não instalado
3. Porta USB sem energia suficiente

**Soluções:**
- Tente outro cabo USB
- Instale drivers: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers
- Use porta USB traseira do computador (não hub)

### Flash falha no meio

**Possíveis causas:**
1. Cabo USB ruim
2. Energia insuficiente
3. Flash corrompida

**Soluções:**
- Use cabo USB curto e de boa qualidade
- Conecte USB diretamente no computador
- Execute `erase_flash` primeiro:
```bash
esptool.py --port COM3 erase_flash
```

### Firmware não inicia após flash

**Possíveis causas:**
1. Endereços incorretos
2. Arquivos .bin errados
3. Configurações SPI incorretas

**Soluções:**
- Verifique endereços: 0x1000, 0x8000, 0x10000
- Baixe o pacote completo novamente
- Use configurações: DIO, 40MHz

### Wi-Fi não aparece

**Possíveis causas:**
1. Firmware não iniciou corretamente
2. Região Wi-Fi incompatível
3. Antena desconectada (raro)

**Soluções:**
- Conecte via serial e verifique mensagens de erro
- Execute `$I` para ver status
- Reset de fábrica: conecte GPIO 0 ao GND durante boot

### WebUI não carrega

**Possíveis causas:**
1. Endereço IP incorreto
2. Firewall bloqueando
3. Sistema de arquivos corrompido

**Soluções:**
- Verifique IP com `$I` via serial
- Desabilite firewall temporariamente
- Re-flash do firmware

## Backup e Restauração

### Backup da Configuração

**Via WebUI:**
- Baixe o arquivo `config.yaml` da aba Files

**Via Serial:**
```
$SD/Show=config.yaml
```
Copie o conteúdo exibido.

### Backup Completo do Flash

```bash
esptool.py --port COM3 read_flash 0x0 0x400000 backup.bin
```

### Restaurar Backup

```bash
esptool.py --port COM3 write_flash 0x0 backup.bin
```

## Comandos Úteis

### Sistema
```
$I           - Informações do sistema
$$           - Listar configurações EEPROM
$RST         - Reset do sistema
$X           - Unlock/destravar
```

### Wi-Fi
```
$WiFi/SSID=nome          - Configurar SSID
$WiFi/Password=senha     - Configurar senha
$WiFi/Mode=STA           - Modo Station
$WiFi/Mode=AP            - Modo Access Point
```

### Arquivos
```
$SD/List                 - Listar arquivos
$SD/Show=arquivo.nc      - Mostrar conteúdo
$SD/Run=arquivo.nc       - Executar G-code
$LocalFS/List            - Listar arquivos locais
```

### Debug
```
$Limits/Check            - Testar limites
$Motors/Enable           - Habilitar motores
$Motors/Disable          - Desabilitar motores
```

## Recursos Adicionais

- [Documentação FluidNC](http://wiki.fluidnc.com/)
- [FAQ FluidNC](http://wiki.fluidnc.com/en/support/faq)
- [Fórum de Suporte](https://github.com/bdring/FluidNC/discussions)
- [Discord FluidNC](https://discord.gg/fluidnc)
