# Guia de Reinstalação — Arch Linux + i3 + NVIDIA PRIME + Steam/CS2

> Setup pensado para um Acer Nitro 5 com:
>
> - CPU: AMD Ryzen 5 4600H
> - GPU integrada: AMD Radeon Vega
> - GPU dedicada: NVIDIA GeForce GTX 1650 Mobile
> - Memória: 16 GB
> - Monitor externo: AOC 1920×1080 a 144 Hz
> - Interface: i3wm sobre Xorg
> - Jogos: Steam e Counter-Strike 2
>
> Objetivo: manter uma instalação minimalista, reproduzível e fácil de diagnosticar, sem reutilizar backups inteiros do sistema anterior.

---

## 1. Arquitetura do setup

A configuração recomendada é:

```text
Arch Linux
├── Kernel linux
├── Xorg
├── i3wm
├── Radeon integrada para o desktop
├── GTX 1650 para jogos via PRIME Render Offload
├── Reverse PRIME para o monitor HDMI
├── PipeWire para áudio
├── NetworkManager para rede
├── Steam
└── GameMode e MangoHud opcionais
```

O ponto mais específico deste notebook é o uso de duas GPUs:

```text
Tela interna
└── controlada pela AMD Radeon integrada

Saída HDMI
└── conectada fisicamente à GTX 1650

Desktop
└── renderizado pela AMD

CS2
└── executado na GTX 1650 com prime-run
```

Por isso, o monitor externo pode não aparecer automaticamente após uma instalação limpa. É necessário conectar o provedor de saída da NVIDIA à sessão Xorg da AMD usando Reverse PRIME.

---

# 2. Recomendações para o archinstall

## 2.1 Perfil

Selecione:

```text
Profile
└── Desktop
    └── i3
```

Isso normalmente instala:

```text
xorg-server
i3-wm
i3status
i3lock
dmenu
lightdm
lightdm-gtk-greeter
```

A lista exata pode variar conforme a versão do `archinstall`.

## 2.2 Kernel

Recomendação inicial:

```text
linux
```

Alternativas:

| Kernel | Quando usar | Pacote NVIDIA correspondente |
|---|---|---|
| `linux` | Melhor escolha inicial | `nvidia-open` |
| `linux-lts` | Recuperação ou maior estabilidade | `nvidia-open-lts` |
| `linux-zen` | Ajustes para desktop/jogos | `nvidia-open-dkms` + `linux-zen-headers` |

Para a primeira instalação, use apenas `linux`. Isso reduz variáveis durante o diagnóstico.

## 2.3 Sistema de arquivos

Recomendação:

```text
ext4
```

Vantagens:

- simples;
- estável;
- fácil de recuperar;
- adequado para Steam, desenvolvimento e containers;
- não exige planejamento de subvolumes.

Alternativa:

```text
btrfs
```

Use Btrfs se quiser:

- snapshots;
- compressão;
- rollback;
- integração com Snapper ou Timeshift.

Btrfs adiciona complexidade e não é necessário para este setup.

## 2.4 Bootloader

Recomendação para UEFI:

```text
systemd-boot
```

Alternativa:

```text
GRUB
```

Use GRUB se precisar de uma configuração mais complexa de dual boot ou customizações avançadas.

## 2.5 Swap

Com 16 GB de RAM:

```text
zram
```

É suficiente para:

- navegação;
- desenvolvimento;
- jogos;
- ferramentas de segurança;
- uso diário.

Para hibernação, será necessário planejar uma área de swap em disco suficientemente grande.

## 2.6 Rede

Selecione:

```text
NetworkManager
```

Ele facilita:

- Wi-Fi;
- Ethernet;
- OpenVPN;
- WireGuard;
- VPNs para CTF;
- integração com `nm-applet`.

## 2.7 Áudio

Selecione:

```text
PipeWire
```

Pacotes esperados:

```text
pipewire
pipewire-pulse
wireplumber
```

Não é necessário instalar PulseAudio separadamente.

## 2.8 Microcode

Para o Ryzen:

```text
amd-ucode
```

## 2.9 Repositório multilib

Habilite o repositório:

```ini
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Arquivo:

```text
/etc/pacman.conf
```

Depois:

```bash
sudo pacman -Syu
```

O `multilib` é necessário para Steam e bibliotecas de 32 bits.

---

# 3. Pacotes essenciais

## 3.1 Base do sistema

Muitos destes pacotes já serão instalados pelo `archinstall`.

```bash
sudo pacman -S --needed \
  linux \
  linux-firmware \
  amd-ucode \
  networkmanager \
  pipewire \
  pipewire-pulse \
  wireplumber
```

Ative a rede:

```bash
sudo systemctl enable --now NetworkManager
```

## 3.2 Xorg e i3

```bash
sudo pacman -S --needed \
  xorg-server \
  xorg-xrandr \
  i3-wm \
  i3status \
  i3lock \
  dmenu
```

### Alternativas

| Componente | Padrão | Alternativa |
|---|---|---|
| Launcher | `dmenu` | `rofi` |
| Barra | `i3bar` + `i3status` | `i3status-rust`, Polybar |
| Terminal | terminal disponível | Kitty, Alacritty, Foot |
| Login | LightDM | TTY + `startx` |
| Gerenciador de arquivos | nenhum | Thunar, Dolphin, PCManFM |

O perfil do `archinstall` pode instalar LightDM automaticamente.

Para um sistema ainda mais minimalista, é possível remover LightDM e iniciar o i3 com `startx`, mas LightDM é mais simples para a primeira instalação.

---

# 4. Drivers gráficos

## 4.1 AMD integrada

```bash
sudo pacman -S --needed mesa
```

Pacotes opcionais para Vulkan na AMD:

```bash
sudo pacman -S --needed \
  vulkan-radeon \
  lib32-mesa \
  lib32-vulkan-radeon
```

Esses pacotes são úteis se aplicações ou jogos forem executados na Radeon integrada.

Para o CS2 rodando exclusivamente na NVIDIA via `prime-run`, os pacotes Vulkan da AMD não são obrigatórios.

## 4.2 NVIDIA GTX 1650

Para o kernel padrão:

```bash
sudo pacman -S --needed \
  nvidia-open \
  nvidia-utils \
  nvidia-prime
```

Para Steam:

```bash
sudo pacman -S --needed lib32-nvidia-utils
```

### Alternativas por kernel

#### Kernel LTS

```bash
sudo pacman -S --needed \
  linux-lts \
  nvidia-open-lts \
  nvidia-utils \
  nvidia-prime
```

#### Kernel alternativo com DKMS

```bash
sudo pacman -S --needed \
  nvidia-open-dkms \
  linux-headers \
  nvidia-utils \
  nvidia-prime
```

Troque `linux-headers` pelos headers correspondentes ao kernel utilizado.

## 4.3 Pacotes de diagnóstico

```bash
sudo pacman -S --needed \
  mesa-utils \
  vulkan-tools
```

Eles fornecem:

```text
glxinfo
vulkaninfo
vkcube
```

Não são obrigatórios para jogar, mas são muito úteis para diagnóstico.

---

# 5. Validação dos drivers

## 5.1 Verificar as GPUs

```bash
lspci -nnk | grep -A3 -E 'VGA|3D|Display'
```

Resultado esperado:

```text
AMD Renoir / Radeon Vega
NVIDIA TU117M / GeForce GTX 1650 Mobile
```

## 5.2 Verificar a GPU do desktop

```bash
glxinfo -B | grep "OpenGL renderer"
```

O esperado é a AMD Radeon integrada.

## 5.3 Testar a GTX 1650

```bash
prime-run glxinfo -B | grep "OpenGL renderer"
```

O esperado:

```text
NVIDIA GeForce GTX 1650
```

## 5.4 Testar NVIDIA

```bash
nvidia-smi
```

## 5.5 Testar Vulkan

```bash
prime-run vulkaninfo --summary
```

Teste visual:

```bash
prime-run vkcube
```

---

# 6. Monitor externo e Reverse PRIME

## 6.1 Problema esperado

O Nitro 5 pode iniciar o Xorg apenas na GPU AMD. Como o HDMI está fisicamente ligado à NVIDIA, o monitor externo pode não aparecer no `xrandr`.

## 6.2 Verificar os provedores

```bash
xrandr --listproviders
```

Resultado típico:

```text
Provider 0: modesetting
Provider 1: NVIDIA-G0
```

Os nomes podem variar.

## 6.3 Conectar o provedor NVIDIA

```bash
xrandr --setprovideroutputsource NVIDIA-G0 modesetting
xrandr --auto
```

Depois:

```bash
xrandr --query
```

A saída HDMI pode aparecer como:

```text
HDMI-1-0
HDMI-0
HDMI-1
HDMI-A-0
```

Use sempre os nomes reais mostrados no seu sistema.

## 6.4 Configurar o AOC em 144 Hz

Exemplo:

```bash
xrandr \
  --output HDMI-1-0 \
  --mode 1920x1080 \
  --rate 144 \
  --primary \
  --right-of eDP-1
```

Confirme:

```bash
xrandr --current
```

O modo ativo aparece com `*`:

```text
1920x1080  144.00*
```

## 6.5 Jogar somente no AOC

```bash
xrandr \
  --output HDMI-1-0 \
  --mode 1920x1080 \
  --rate 144 \
  --primary \
  --output eDP-1 \
  --off
```

Reativar a tela interna:

```bash
xrandr \
  --output eDP-1 \
  --auto \
  --left-of HDMI-1-0
```

## 6.6 Tornar permanente no i3

Arquivo:

```text
~/.config/i3/config
```

Exemplo:

```ini
# Disponibiliza as saídas conectadas à GTX 1650
exec_always --no-startup-id xrandr --setprovideroutputsource NVIDIA-G0 modesetting

# Aguarda o provedor NVIDIA e configura o monitor externo
exec_always --no-startup-id sh -c 'sleep 2; xrandr --output HDMI-1-0 --mode 1920x1080 --rate 144 --primary --right-of eDP-1'
```

Substitua:

```text
NVIDIA-G0
modesetting
HDMI-1-0
eDP-1
```

pelos nomes reais do seu sistema.

## 6.7 DRM modesetting

Verifique:

```bash
cat /sys/module/nvidia_drm/parameters/modeset
```

Esperado:

```text
Y
```

Se já retornar `Y`, não faça nenhuma alteração.

Se retornar `N`, adicione ao kernel:

```text
nvidia_drm.modeset=1
```

Não adicione esse parâmetro automaticamente sem antes verificar.

---

# 7. Steam e CS2

## 7.1 Pacotes mínimos

```bash
sudo pacman -S --needed \
  steam \
  lib32-nvidia-utils \
  nvidia-prime
```

## 7.2 GameMode

Instalação:

```bash
sudo pacman -S --needed \
  gamemode \
  lib32-gamemode
```

Adicionar o usuário ao grupo:

```bash
sudo usermod -aG gamemode "$USER"
```

Depois faça logout completo ou reinicie.

Verifique:

```bash
groups
```

Teste:

```bash
gamemoded -t
```

Verificar durante um jogo:

```bash
gamemoded -s
```

## 7.3 MangoHud

Instalação:

```bash
sudo pacman -S --needed \
  mangohud \
  lib32-mangohud
```

Teste:

```bash
prime-run mangohud vkcube
```

## 7.4 Opções de inicialização do CS2

### Configuração mínima

```text
prime-run %command%
```

### Configuração recomendada

```text
prime-run gamemoderun %command%
```

### Configuração para diagnóstico

```text
prime-run gamemoderun mangohud %command%
```

Não adicione vários parâmetros de otimização antes de obter um resultado-base.

## 7.5 Validar o uso da NVIDIA

Com o jogo aberto:

```bash
watch -n 1 nvidia-smi
```

O processo do CS2 deve aparecer utilizando a GTX 1650.

## 7.6 Testes recomendados

Use sempre o mesmo mapa, resolução e configurações.

### Teste A

```text
Tela interna ligada
AOC ligado em 144 Hz
CS2 em tela cheia no AOC
```

### Teste B

```text
Somente o AOC ligado
CS2 em tela cheia
```

### Teste C

```text
Sem MangoHud
prime-run gamemoderun %command%
```

Compare:

- FPS médio;
- quedas de FPS;
- frametime;
- utilização da GPU;
- temperatura;
- tearing;
- microtravamentos;
- áudio;
- microfone;
- comportamento do mouse.

---

# 8. Componentes opcionais do ambiente i3

Depois que o CS2 estiver funcionando:

```bash
sudo pacman -S --needed \
  rofi \
  dunst \
  kitty \
  thunar \
  thunar-archive-plugin \
  network-manager-applet \
  pavucontrol \
  brightnessctl \
  playerctl \
  feh \
  flameshot \
  unzip \
  p7zip \
  git \
  base-devel
```

## 8.1 Exemplo de configuração do i3

```ini
set $mod Mod4

bindsym $mod+Return exec kitty
bindsym $mod+d exec rofi -show drun
bindsym Print exec flameshot gui

exec --no-startup-id dunst
exec --no-startup-id nm-applet
exec_always --no-startup-id feh --bg-fill ~/Pictures/wallpaper.jpg
```

## 8.2 Picom

Picom é opcional.

Instalação:

```bash
sudo pacman -S picom
```

Vantagens:

- transparência;
- sombras;
- fading;
- compositor para X11.

Possíveis problemas:

- tearing;
- microtravamentos;
- latência;
- conflito com fullscreen;
- consumo adicional de GPU.

Recomendação:

```text
não instalar Picom antes de validar o CS2
```

Se usar Picom, considere encerrá-lo antes de jogos competitivos.

---

# 9. O que não é necessário neste setup

Evite adicionar sem uma necessidade concreta:

```text
Hyprland
Gamescope
BlackArch completo
Polybar
Picom
nvidia-xconfig
xorg.conf estático
overclock da NVIDIA
parâmetros aleatórios de kernel
parâmetros antigos de inicialização do CS2
scripts agressivos de otimização
```

## 9.1 Não execute automaticamente

```bash
sudo nvidia-xconfig
```

Esse comando pode criar um `/etc/X11/xorg.conf` estático e atrapalhar o funcionamento híbrido AMD + NVIDIA.

Deixe o Xorg detectar as GPUs automaticamente e use Reverse PRIME via `xrandr`.

---

# 10. Auditoria da instalação atual

## 10.1 Pacotes instalados explicitamente

```bash
pacman -Qqe
```

Salvar:

```bash
pacman -Qqe > ~/pacotes-explicitos.txt
```

## 10.2 Pacotes oficiais

```bash
pacman -Qqen
```

## 10.3 Pacotes externos ou AUR

```bash
pacman -Qqem
```

## 10.4 Histórico de instalações

```bash
grep '\[ALPM\] installed' /var/log/pacman.log
```

Salvar:

```bash
grep '\[ALPM\] installed' /var/log/pacman.log \
  > ~/historico-instalacoes.txt
```

## 10.5 Pacotes órfãos

```bash
pacman -Qdtq
```

Antes de remover:

```bash
pacman -Qi nome-do-pacote
```

Não remova pacotes órfãos automaticamente sem revisar.

## 10.6 Inventário específico deste setup

```bash
pacman -Q | grep -E \
'^(linux|amd-ucode|mesa|nvidia|lib32-nvidia|xorg|i3|lightdm|steam|gamemode|lib32-gamemode|mangohud|lib32-mangohud|vulkan)'
```

---

# 11. Lista mínima para futura reinstalação

Depois do `archinstall` com perfil i3:

## 11.1 Drivers

```bash
sudo pacman -S --needed \
  mesa \
  nvidia-open \
  nvidia-utils \
  nvidia-prime \
  xorg-xrandr
```

## 11.2 Multilib

Em `/etc/pacman.conf`:

```ini
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Atualizar:

```bash
sudo pacman -Syu
```

## 11.3 Steam e CS2

```bash
sudo pacman -S --needed \
  steam \
  lib32-nvidia-utils
```

## 11.4 Otimizações opcionais

```bash
sudo pacman -S --needed \
  gamemode \
  lib32-gamemode \
  mangohud \
  lib32-mangohud
```

Grupo GameMode:

```bash
sudo usermod -aG gamemode "$USER"
```

## 11.5 Diagnóstico opcional

```bash
sudo pacman -S --needed \
  mesa-utils \
  vulkan-tools
```

## 11.6 Monitor externo

```bash
xrandr --setprovideroutputsource NVIDIA-G0 modesetting
xrandr --auto
```

Depois configure o monitor com os nomes reais:

```bash
xrandr \
  --output HDMI-1-0 \
  --mode 1920x1080 \
  --rate 144 \
  --primary \
  --right-of eDP-1
```

## 11.7 Configuração permanente

Adicionar ao `~/.config/i3/config`:

```ini
exec_always --no-startup-id xrandr --setprovideroutputsource NVIDIA-G0 modesetting
exec_always --no-startup-id sh -c 'sleep 2; xrandr --output HDMI-1-0 --mode 1920x1080 --rate 144 --primary --right-of eDP-1'
```

## 11.8 CS2

Opção mínima:

```text
prime-run %command%
```

Opção recomendada:

```text
prime-run gamemoderun %command%
```

Opção de diagnóstico:

```text
prime-run gamemoderun mangohud %command%
```

---

# 12. Checklist de reinstalação

```text
[ ] Instalar Arch em modo UEFI
[ ] Selecionar kernel linux
[ ] Selecionar perfil Desktop → i3
[ ] Selecionar NetworkManager
[ ] Selecionar PipeWire
[ ] Instalar amd-ucode
[ ] Usar ext4 para uma instalação simples
[ ] Usar systemd-boot
[ ] Habilitar multilib
[ ] Instalar mesa
[ ] Instalar nvidia-open
[ ] Instalar nvidia-utils
[ ] Instalar nvidia-prime
[ ] Instalar lib32-nvidia-utils
[ ] Validar nvidia-smi
[ ] Validar prime-run
[ ] Configurar Reverse PRIME
[ ] Configurar AOC em 144 Hz
[ ] Tornar xrandr permanente no i3
[ ] Instalar Steam
[ ] Testar CS2 com prime-run
[ ] Instalar GameMode se desejado
[ ] Adicionar usuário ao grupo gamemode
[ ] Instalar MangoHud apenas para métricas
[ ] Validar CS2 antes de instalar ferramentas de trabalho
```

---

# 13. Ordem recomendada de instalação

```text
Arch básico
    ↓
Xorg + i3
    ↓
NetworkManager + PipeWire
    ↓
Mesa + driver NVIDIA
    ↓
Validação das duas GPUs
    ↓
Reverse PRIME
    ↓
Monitor AOC em 144 Hz
    ↓
Steam
    ↓
CS2 com prime-run
    ↓
GameMode
    ↓
MangoHud
    ↓
Ambiente visual do i3
    ↓
Ferramentas de programação
    ↓
Ferramentas de segurança
    ↓
Containers e máquinas virtuais
```

Essa ordem evita instalar muitas ferramentas antes de confirmar que o sistema gráfico e os jogos estão estáveis.

---

# 14. Princípios para manter o sistema limpo

1. Instale somente ferramentas realmente utilizadas.
2. Prefira pacotes oficiais do Arch.
3. Use AUR apenas quando necessário.
4. Não adicione o repositório completo do BlackArch ao host.
5. Use containers ou VMs para ferramentas específicas.
6. Evite arquivos globais de configuração do Xorg sem necessidade.
7. Teste uma alteração por vez.
8. Registre comandos e configurações que realmente resolveram problemas.
9. Não reutilize um backup completo em uma reinstalação.
10. Reaplique apenas configurações comprovadamente necessárias.

---

# 15. Resumo essencial

O setup mínimo realmente necessário é:

```text
Arch Linux
├── linux
├── linux-firmware
├── amd-ucode
├── NetworkManager
├── PipeWire
├── Xorg
├── i3
├── Mesa
├── nvidia-open
├── nvidia-utils
├── nvidia-prime
├── Reverse PRIME
├── multilib
├── Steam
└── lib32-nvidia-utils
```

Itens opcionais:

```text
GameMode
MangoHud
mesa-utils
vulkan-tools
rofi
dunst
kitty
thunar
nm-applet
pavucontrol
feh
flameshot
Picom
```

O único ajuste específico que provavelmente será necessário em toda reinstalação deste notebook é o Reverse PRIME para expor a saída HDMI da GTX 1650 dentro da sessão Xorg executada na GPU AMD.
