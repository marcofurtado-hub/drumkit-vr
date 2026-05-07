# Drumkit VR

Bateria de rock 5 peças em VR pra Meta Quest 2, rodando direto no browser via WebXR.
Single HTML, Three.js + Web Audio + samples reais.

## Layout do kit

- **Bumbo** (kick) – centro, no chão. Trigger de qualquer mão = pedal.
- **Caixa** (snare) – entre as pernas, leve à esquerda.
- **Tom 1** (alto) – montado sobre o bumbo, esquerda.
- **Tom 2** (médio) – montado sobre o bumbo, direita.
- **Tom 3** (floor) – à direita do baterista.
- **Chimbal** (hi-hat) – esquerda, em pedestal. Grip esquerdo segurado = abre.
- **Crash** – alto à esquerda, levemente inclinado.
- **Ride** – alto à direita, levemente inclinado.

## Controles (Quest 2)

| Ação | Como |
|---|---|
| Bater nos tambores/pratos | Mover o controle como uma baqueta — a ponta colide com o tambor |
| Volume | Velocidade do golpe (0.4 m/s = soft, 4+ m/s = slam) |
| Bumbo | Trigger de qualquer controle |
| Chimbal aberto | Segurar o Grip esquerdo |
| Sair do VR | Botão Oculus / menu |

Haptic feedback é proporcional à velocidade do golpe — um ghost note vibra leve, um rim shot vibra forte.

## Tech

- **Three.js 0.165** via importmap CDN (unpkg)
- **WebXR** API (`renderer.xr` + `VRButton`)
- **Web Audio API** com:
  - polyphony (cada hit cria um BufferSource novo)
  - velocity → gain (curva perceptual ^1.4)
  - velocity → pitch (±25¢ aleatório + ±40¢ por intensidade)
  - reverb pequeno via convolução com IR de ruído (sala curta)
  - pan estéreo por posição da peça no kit
  - anti-machinegun (mínimo 12ms entre dois hits da mesma peça)
- **Hit detection** via `Raycaster` segmento entre tip-position de frame anterior e atual
- **Cooldown** por (peça × stick) = 80ms pra evitar dupla-trigger no mesmo movimento

## Samples

Single-velocity, 8 peças, ~240KB total:

| Peça | Arquivo | Origem |
|---|---|---|
| kick, snare, hihat, tom1-3 | `sounds/*.mp3` | [Tone.js audio – Stark rock kit](https://github.com/Tonejs/audio/tree/master/drum-samples/Stark) |
| crash | `sounds/crash.wav` | [Dirt-Samples cc/CSHD0](https://github.com/tidalcycles/Dirt-Samples) |
| ride | `sounds/ride.wav` | [Dirt-Samples cr/RIDED0](https://github.com/tidalcycles/Dirt-Samples) |

Todos sob licenças permissivas. Pra trocar por samples melhores depois, basta substituir os arquivos em `sounds/` mantendo os mesmos nomes.

## Rodando localmente

```bash
cd ~/Desktop/drumkit
python3 -m http.server 8000
# abra http://localhost:8000
```

WebXR só funciona em **HTTPS** (exceto `localhost`). Pra testar no Quest 2 pela rede local, você precisa de HTTPS.

### Opção 1 — testar no Quest 2 via rede local com HTTPS auto-assinado

```bash
# instalar mkcert (uma vez)
brew install mkcert
mkcert -install

# gerar cert pro IP local (descubra com `ipconfig getifaddr en0`)
mkcert 192.168.X.X localhost

# rodar com http-server (npm)
npx http-server -S -C 192.168.X.X+1.pem -K 192.168.X.X+1-key.pem -p 8443
# abra https://192.168.X.X:8443 no browser do Quest
```

### Opção 2 — deploy direto (mais simples)

Sobe pro GitHub Pages e usa a URL HTTPS de produção (ver abaixo).

## Deploy no hub

O hub `marcofurtado-hub.github.io` já tem o card do Drumkit VR apontando pra `https://marcofurtado-hub.github.io/drumkit-vr/`. Pra ele funcionar você precisa criar o repo:

```bash
cd ~/Desktop/drumkit

# inicializar git
git init
git add .
git commit -m "Drumkit VR — initial release"

# criar repo no GitHub (gh CLI ou pela web)
gh repo create marcofurtado-hub/drumkit-vr --public --source=. --remote=origin --push

# ativar GitHub Pages: Settings → Pages → Source: deploy from branch → main → /(root)
gh repo edit marcofurtado-hub/drumkit-vr --enable-pages

# OU via web:
# 1. https://github.com/new → org=marcofurtado-hub, name=drumkit-vr, public
# 2. push o conteúdo
# 3. Settings → Pages → Source: main / root → Save
```

Depois de ~1min o site estará no ar em `https://marcofurtado-hub.github.io/drumkit-vr/`.

## Estrutura

```
~/Desktop/drumkit/
├── index.html          # tudo (Three.js scene + audio engine)
├── README.md           # este arquivo
└── sounds/
    ├── kick.mp3
    ├── snare.mp3
    ├── hihat.mp3
    ├── tom1.mp3
    ├── tom2.mp3
    ├── tom3.mp3
    ├── crash.wav
    └── ride.wav
```

## Roadmap (ideias futuras)

- [ ] Multi-velocity samples (round-robin entre 3-4 takes por peça pra realismo extra)
- [ ] Pedal de bumbo virtual rastreado pelos pés (precisa Body Tracking SDK do Quest)
- [ ] Modo "play along" com música de fundo
- [ ] Visualizer de partícula nos pratos quando batidos
- [ ] Selector de kits (rock / jazz / metal)
- [ ] Loopback recording — grava a performance e compartilha
