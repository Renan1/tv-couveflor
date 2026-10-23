# Projeto TV Couve Flor (sinalização digital do restaurante)

Resumo objetivo para retomar o trabalho em outra sessão. Idioma: português (BR). Usuário: Renan (Couve Flor, restaurante corporativo, Juiz de Fora - MG). Prefere respostas diretas e passo a passo.

## Objetivo
Usar uma TV ociosa do restaurante como TV corporativa: slideshow de imagens/vídeos promocionais, aberto sozinho ao ligar, sem mensalidade.

## Decisões e histórico
- Sistemas prontos analisados (Beehome, OptiSigns, Pixmidia): B2B, pagos, exagero pra 1 TV.
- PiSignage (grátis até 2 telas) foi tentado e **abandonado**: a imagem ficava reiniciando em loop (suspeita de fonte fraca / "Undervoltage").
- Solução adotada: **Raspberry Pi 3 com Raspberry Pi OS padrão + Chromium em modo kiosk** abrindo uma página web própria.
- Hospedagem: saiu do Netlify (plano grátis virou sistema de créditos: 300/mês, 15 por deploy, site pausa se estourar) e foi pro **GitHub Pages + subdomínio no Cloudflare** (CNAME para `SEU-USUARIO.github.io`, proxy em "DNS only" no início, depois "Enforce HTTPS"). Funcionou.
- Limites verificados: GitHub Pages ~100 GB/mês de banda (informal), 1 GB por site, 10 builds/hora, um site por repositório. Cloudflare grátis: 200 registros DNS por zona (1000 se o domínio é antigo). Só serve arquivos estáticos (sem backend).

## A página (index.html)
Arquivo único, sem lista de conteúdo. Varre a pasta `conteudo/` por arquivos numerados:
- Nome: dois dígitos + extensão (`01.png`, `02.jpg`, `03.mp4`...). Número define a ordem; "buracos" são permitidos; procura de 01 a 30 (`MAX_ITEMS`).
- Imagens: jpg, jpeg, png, webp, gif. Vídeos: mp4, webm, mov, m4v (precisa ser **H.264**; HEVC não toca).
- Imagem fica 18 s (`DEFAULT_DURATION`). Vídeo toca uma vez e congela no último quadro; só avança quando terminou **e** o tempo mínimo passou (serve p/ gif convertido em mp4). Trava de segurança de 3 min.
- Imagens e vídeos usam `object-fit: cover` (preenche a tela, sem faixa preta).
- Vídeos só carregam quando entram na tela e são liberados ao sair (Pi 3 tem um único decodificador; carregar todos travava/ficava tela branca).
- Relógio grande no canto (56px). Erros de vídeo aparecem num aviso vermelho temporário. Recarrega a página a cada 30 min.
- **Música de fundo foi removida** (pesava no Pi).
- GIF animado trava o Pi: converter para mp4 (ex.: ezgif.com/gif-to-mp4).

## Arquivos
Pasta canônica no PC: `C:\Users\diret\OneDrive\COUVE FLOR\2026\RESPOSTAS CLAUDE IA\TV Couve Flor\`
- `index.html` (versão final), `conteudo/01.png … 04.png` (+ novas).
- `img/`, `video/`, `content.json` e o `LEIA-ME.txt` são da versão antiga e **não são mais usados**.
- Conferir se o `index.html` no GitHub é igual ao da pasta (última versão: sem música, vídeo com carregamento sob demanda).
- Atualizar conteúdo: no repositório do GitHub, abrir `conteudo/` → Add file → Upload files → Commit. Leva 1-2 min. (Upload pelo navegador tem limite por arquivo, ~25 MB; manter vídeos leves, 1080p, 5-10 Mbps.)

## Raspberry Pi
- Hostname `Pi3b-teste`, usuário `host`, IP fixo `192.168.0.113` (via nmcli, conexão `netplan-wlan0-Restaurante Couve Flor`). Senha: a definida na gravação do cartão.
- OS recente (Debian trixie, Wayland/labwc). Acesso: SSH (`ssh host@192.168.0.113`) e **Raspberry Pi Connect** (https://connect.raspberrypi.com, já configurado).
- Ajustes feitos: login automático no desktop; Screen Blanking desativado.
- Autostart: `~/.config/labwc/autostart` (versão mais recente sugerida):
```
pkill -9 chromium 2>/dev/null
rm -f ~/.config/chromium/Singleton* 2>/dev/null
sleep 1
until ping -c1 -W1 8.8.8.8 >/dev/null 2>&1; do sleep 1; done
for i in $(seq 1 20); do
  wlr-randr --output HDMI-A-1 --mode 1920x1080@60Hz && break
  sleep 1
done
sleep 2
while true; do
  chromium https://SEU_SUBDOMINIO --kiosk --noerrdialogs --disable-infobars --no-first-run --enable-features=OverlayScrollbar --start-maximized --password-store=basic
  sleep 2
done &
```
- `--password-store=basic` evita o pedido de senha do chaveiro. `--autoplay-policy=no-user-gesture-required` era só pra música (opcional).
- TV: LG com painel nativo **1360x768** (modo preferido no EDID). Testar `1360x768@60Hz` no lugar de 1920x1080 se houver corte/zoom. Se o overscan persistir, renomear a entrada HDMI para "PC" / "Just Scan" nas configs da TV.

## Problemas conhecidos / lições
- **Resolução voltava a cada reboot**: `wlr-randr` manual não persiste; por isso o loop de tentativas no autostart. Alternativa extra: `video=HDMI-A-1:1920x1080@60` no fim da linha única de `/boot/firmware/cmdline.txt`.
- **Chromium abrindo em janela pequena (sem kiosk)**: havia instância antiga; resolvido com `pkill` + remoção de `Singleton*` no início do autostart. Se voltou a dar "pau de resolução" depois disso, pedir foto da TV + saída de `wlr-randr` logo após o boot.
- Aviso `Undervoltage detected`: usar fonte oficial 5V/2.5A+, cabo curto, direto na tomada (não na USB da TV).
- Modo somente leitura (`raspi-config` → Performance → Overlay File System) protege o SD de queda de energia; ativar só no final, após tudo testado.
- Música continuar tocando após fechar a aba no PC é comportamento do navegador (sessão de mídia), não afeta a TV.

## Pendências
1. Confirmar que o autostart novo mantém a resolução após reboot e queda de energia.
2. Criar a **página explicativa para o site** (o que é, o que faz, como usa, com prints do GitHub). Precisa: link do repositório, subdomínio e o Renan logado no GitHub no painel do navegador do app (Claude não digita senha). Formato: página HTML pronta pra publicar (+ PDF opcional).
3. Outros sistemas (cardápio diário, QR code, viagens Uber) migrando do Netlify para GitHub Pages + subdomínios. Atenção: só funciona se forem 100% estáticos (sem backend/banco/chave secreta).

## Alternativa futura: Fire TV Stick / Chromecast no lugar do Raspberry
A TV atual (LG antiga, provavelmente webOS) não faz kiosk nem abre sozinha ao ligar, então o Pi segue. Renan tem em casa um Chromecast e um Fire TV Stick.
- **Fire TV Stick (Amazon)**: mais promissor. Roda Fire OS (base Android); o **Fully Kiosk Browser** tem "Launch on Boot" e tela cheia; PiSignage também roda nele. Fraqueza: a tela inicial da Amazon tem anúncios se o app fechar.
- **Chromecast com Google TV** (com controle remoto): serve, mesmo tipo de app. **Chromecast antigo** (sem controle, só "transmite"): **não serve**, não instala apps nem navegador.
- Vale pra ambos: ligar em **tomada na parede** (não na USB da TV, mesmo problema de Undervoltage); deixar sempre ligado, assim a página já está na tela ao ligar a TV; o autostart só importa após queda de luz. Saída do stick em **720p** (painel é 1360x768). Liberar manualmente as permissões "Exibir sobre outros apps" e "Modificar configurações do sistema" para o app abrir sozinho; no Android 10+ o sistema pode bloquear o autoinício (saída: definir o app como tela inicial via adb). **Testar puxar da tomada e religar** antes de confiar.
- Fully Kiosk: tem versão para testar; alguns recursos de kiosk são pagos (licença única, conferir preço atual).
- Plano: testar primeiro com o Fire TV Stick + Fully Kiosk apontando para o mesmo subdomínio; se funcionar, aposentar o Raspberry sem mexer no site. Perguntas em aberto: se o Chromecast tem controle remoto; modelo/versão do Fire TV Stick.
