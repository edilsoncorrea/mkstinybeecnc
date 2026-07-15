---
inclusion: manual
---

# CNCjs — Auto-Level Workflow para PCB

Documenta o fluxo de auto-nivelamento no CNCjs usando a extensão cncjs-kt-ext para fresamento de PCB.

Fonte: https://github.com/kreso-t/cncjs-kt-ext

---

## Pré-requisitos

- CNCjs instalado (desktop ou Raspberry Pi)
- Extensão cncjs-kt-ext rodando em paralelo (Node.js)
- Probe conectado (gpio.35 no projeto atual)
- Protocolo Grbl/FluidNC

### Inicialização da extensão

```bash
cd /caminho/para/cncjs-kt-ext
node . --port /dev/ttyACM0  # ou COM port no Windows
```

Com autenticação:
```bash
node . --id 'ID' --name 'user' --secret 'SECRET' --port 'COM8' --baudrate 115200 --socket-address 'localhost' --socket-port 8000
```

---

## Fluxo Completo de Auto-Level

### Passo 1: Carregar GCode no CNCjs

Carregue o arquivo GCode (isolamento, furos, contorno) normalmente.

### Passo 2: Posicionar e zerar

- Jog até a origem da PCB (X0, Y0)
- Z-probe manual para zerar Z na superfície do cobre

### Passo 3: Executar macro de autolevel

Criar uma macro no CNCjs com:
```
(#autolevel)
```

**IMPORTANTE:** Este comando SÓ funciona via macro — não funciona no console nem dentro do GCode.

### O que acontece automaticamente:

1. Lê os limites (xmin, ymin, xmax, ymax) do GCode carregado
2. Faz grid de probing (G38.2) em toda a área da PCB
3. Interpola valores Z via interpolação bilinear entre os 4 pontos mais próximos
4. **Modifica o GCode carregado** — ajusta Z de cada movimento com compensação
5. Recarrega o GCode compensado no CNCjs

### Passo 4: Executar o GCode

Basta dar Play — o GCode já está com Z compensado.

---

## Parâmetros Customizáveis

```
(#autolevel D[distância] H[altura] F[feedrate] M[margem] P[probeOnly] X[xSize] Y[ySize])
```

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| D | 10 mm | Distância entre pontos do grid (XY) |
| H | 2 mm | Altura de deslocamento entre probes |
| F | 50 mm/min | Feedrate do probe (G38.2) |
| M | 0 mm | Margem ao redor da área da PCB |
| P | 0 | 0=aplica ao GCode, 1=só proba (não modifica GCode) |
| X | auto | Tamanho X da área (quando P=1 sem GCode carregado) |
| Y | auto | Tamanho Y da área (quando P=1 sem GCode carregado) |

### Recomendação para PCB

```
(#autolevel D5 H1.0 F30 M0.2)
```

- D5 ou D7.5 para melhor resolução em trilhas finas
- F30 para probe mais lento e preciso
- M0.2 para margem de segurança

---

## Reaplicar em Novo GCode (mesma PCB)

Quando precisa rodar outro GCode na mesma PCB (ex: furos depois do isolamento):

1. Carregue o novo GCode
2. Se trocou a broca: Z-probe no ponto de origem (xmin, ymin) e zere Z
3. Execute macro:
   ```
   (#autolevel_reapply)
   ```

Isso reaplica os dados do probe anterior — sem probar novamente.

---

## Comandos Auxiliares

| Comando | Função |
|---------|--------|
| `(#autolevel)` | Proba + aplica ao GCode carregado |
| `(#autolevel_reapply)` | Reaplica probe anterior ao GCode atual |
| `(PROBEOPEN filename)` | Salva valores probados em arquivo |
| `(PROBECLOSE)` | Fecha arquivo de probe (automático no fim do autolevel) |

---

## Notas Importantes

- Movimentos longos são divididos em segmentos menores que o grid de probe para interpolação correta
- O GCode original é substituído na memória do CNCjs — o arquivo no disco não é alterado
- Se o CNCjs for reiniciado, precisa reprobar ou usar `#autolevel_reapply` com dados salvos
- Compatível apenas com G0/G1 (movimentos lineares) — G2/G3 (arcos) não são suportados pela extensão

---

## Alternativa: bCNC

O bCNC tem auto-level nativo (sem extensão). Fluxo similar:
1. Scan > define área e grid
2. Probe (executa G38.2 grid)
3. Apply heightmap ao GCode (botão "Level")
4. Executa o GCode compensado

Ambos fazem a mesma coisa — a diferença é que no bCNC é built-in e no CNCjs requer a extensão.
