---
name: instagram-carousel
description: >
  Gera carrosseis de Instagram (slides 1080x1350px) com a identidade visual de cada projeto
  do Studio Lagosta — cores, fontes, logo e tom de voz puxados automaticamente da base.
  Use esta skill sempre que o usuario pedir para criar carrossel, carousel, post de varias paginas,
  slides para Instagram, conteudo swipeable, post educativo, listicle, tutorial visual,
  comparativo antes/depois, ou qualquer conteudo de multiplos slides para feed.
  Tambem se aplica quando o usuario diz coisas como "faz um carrossel sobre o cardapio",
  "cria slides do happy hour", "monta um post educativo sobre vinhos",
  "faz um antes e depois do restaurante", "cria um conteudo de dicas".
  Esta skill NAO se aplica a Stories (use content-planner para isso).
---

# Carrossel de Instagram

Voce e o designer de carrosseis do Studio Lagosta. Sua missao e criar carrosseis visualmente coesos, com a identidade de cada projeto, passando por 4 fases com aprovacao do usuario entre elas.

O carrossel e gerado como HTML (420px de largura) e exportado via Playwright para PNGs de 1080x1350px — a resolucao nativa do Instagram para posts em retrato.

---

## Visao Geral do Fluxo

```
Fase 0: Carregar Projeto (cache local — instantaneo)
    ↓ projeto identificado
Fase 1: Copys + Escolha de Preset
    ↓ usuario aprova textos + preset
Fase 2: Curadoria de Imagens (pagina HTML interativa)
    ↓ usuario aprova selecao
Fase 3: Montagem HTML + Preview Instagram + Export PNG
    ↓ usuario aprova preview
Exporta PNGs finais
```

Cada fase depende da anterior. Peca aprovacao antes de avancar.

---

## Fase 0: Carregar Projeto

O sistema usa **cache local** para brand assets que raramente mudam. Isso evita buscar cores, fontes e logo do banco de dados toda vez.

### 0.1 Estrutura de Arquivos

```
~/.claude/skills/instagram-carousel/
├── SKILL.md                           (este arquivo)
├── curadoria-template.html            (template de curadoria — nao modificar)
├── design-system-template.html        (template base para design systems)
└── projects/                          (cache por projeto)
    └── {id}-{slug}/
        ├── brand.json                 (cores, logo base64, fontes, KB resumo)
        ├── design-system.html         (pagina visual para revisar no browser)
        ├── design-system.json         (tokens programaticos para a skill)
        └── fonts/
            ├── heading.otf            (fonte de titulos)
            └── body.otf              (fonte de corpo)
```

### 0.2 Fluxo de Carregamento

```
1. Matar servidor HTTP anterior se estiver rodando
   $ lsof -ti:8787 | xargs kill -9 2>/dev/null

2. Identificar projeto:
   - Se o usuario mencionou o projeto → usar
   - Senao → list-projects e perguntar

3. Verificar cache local:
   projects/{id}-{slug}/brand.json existe?
   ├─ SIM → carregar brand.json + design-system.json
   │        Se cachedAt > 30 dias → avisar usuario
   └─ NAO → executar Setup Automatico (0.3)

4. Criar pasta de trabalho:
   /tmp/carousel-{slug}/
```

### 0.3 Setup Automatico (primeira vez)

Quando o projeto nao tem cache local:

```
1. get-brand-assets(projectId) → receber cores, fontes, logos, KB
2. Criar diretorio: projects/{id}-{slug}/
3. Baixar logo como base64:
   - Usar a URL do logo principal (isProjectLogo=true)
   - Converter para data:image/png;base64,...
4. Baixar fontes:
   - Para cada CustomFont, baixar fileUrl → projects/{id}-{slug}/fonts/
   - Converter para base64 para embeder no HTML
5. Gerar brand.json com dados cacheados
6. Gerar design-system.json com palette derivada + typography + presets
7. Gerar design-system.html a partir do template (substituir placeholders)
```

### 0.4 Formato do brand.json

```json
{
  "projectId": 7,
  "projectName": "By Rock",
  "projectSlug": "by-rock",
  "instagramUsername": "by.rock",
  "cachedAt": "2026-04-06T01:00:00Z",
  "logo": {
    "fileUrl": "https://...",
    "base64": "data:image/png;base64,..."
  },
  "colors": [
    {"name": "Primary", "hexCode": "#C0392B"}
  ],
  "fonts": {
    "heading": {"name": "Metrisch ExtraBold", "fontFamily": "Metrisch ExtraBold", "localFile": "fonts/heading.otf", "base64": "data:font/otf;base64,..."},
    "body": {"name": "Metrisch Book", "fontFamily": "Metrisch Book", "localFile": "fonts/body.otf", "base64": "data:font/otf;base64,..."}
  },
  "knowledge": {
    "tomDeVoz": "Rock and roll, irreverente, energetico...",
    "estabelecimento": "Steak house com tematica rock...",
    "horarios": "Todos os dias 11h a meia-noite, HH 17h-20h",
    "diferenciais": "Cortes nobres, ambiente rock, drinks autorais"
  }
}
```

### 0.5 Formato do design-system.json

```json
{
  "projectId": 7,
  "projectSlug": "by-rock",
  "palette": {
    "primary": "#C0392B",
    "primaryLight": "#E74C3C",
    "primaryDark": "#8B1A10",
    "lightBg": "#FFF8F7",
    "darkBg": "#1A0A08",
    "textLight": "#FFFFFF",
    "textDark": "#1A1A1A"
  },
  "typography": {
    "headingFont": "Metrisch ExtraBold",
    "bodyFont": "Metrisch Book",
    "headingSizePx": 32,
    "bodySizePx": 14,
    "tagSizePx": 10
  },
  "overlays": {
    "bottom": {
      "position": "bottom", "height": "50%",
      "gradient": "linear-gradient(to top, rgba(0,0,0,0.92) 0%, rgba(0,0,0,0.75) 50%, rgba(0,0,0,0) 100%)"
    },
    "top": {
      "position": "top", "height": "50%",
      "gradient": "linear-gradient(to bottom, rgba(0,0,0,0.92) 0%, rgba(0,0,0,0.75) 50%, rgba(0,0,0,0) 100%)"
    },
    "left": {
      "position": "left", "width": "70%",
      "gradient": "linear-gradient(to right, rgba(0,0,0,0.92) 0%, rgba(0,0,0,0.75) 50%, rgba(0,0,0,0) 100%)"
    },
    "right": {
      "position": "right", "width": "70%",
      "gradient": "linear-gradient(to left, rgba(0,0,0,0.92) 0%, rgba(0,0,0,0.75) 50%, rgba(0,0,0,0) 100%)"
    },
    "gradientBrand": {
      "position": "full",
      "gradient": "linear-gradient(165deg, {{primaryDark}} 0%, {{primary}} 50%, {{primaryLight}} 100%)"
    }
  },
  "layoutPresets": {
    "standard": [
      {"overlay": "bottom", "textAlign": "bottom-left", "hasPhoto": true},
      {"overlay": "top", "textAlign": "top-left", "hasPhoto": true},
      {"overlay": "left", "textAlign": "center-left", "hasPhoto": true},
      {"overlay": "right", "textAlign": "center-right", "hasPhoto": true},
      {"overlay": "gradientBrand", "textAlign": "center", "hasPhoto": false}
    ],
    "editorial": [
      {"overlay": "bottom", "textAlign": "bottom-left", "hasPhoto": true},
      {"overlay": "left", "textAlign": "center-left", "hasPhoto": true},
      {"overlay": "bottom", "textAlign": "bottom-left", "hasPhoto": true},
      {"overlay": "right", "textAlign": "center-right", "hasPhoto": true},
      {"overlay": "gradientBrand", "textAlign": "center", "hasPhoto": false}
    ],
    "bold": [
      {"overlay": "gradientBrand", "textAlign": "center", "hasPhoto": false},
      {"overlay": "top", "textAlign": "top-left", "hasPhoto": true},
      {"overlay": "bottom", "textAlign": "bottom-left", "hasPhoto": true},
      {"overlay": "top", "textAlign": "top-left", "hasPhoto": true},
      {"overlay": "gradientBrand", "textAlign": "center", "hasPhoto": false}
    ]
  }
}
```

### 0.6 Derivar Palette de Cores

A partir da cor primaria do projeto, derive 5 tokens adicionais:

| Token | Derivacao | Uso |
|-------|-----------|-----|
| `primary` | Cor principal da marca | Destaques, CTA, progress bar |
| `primaryLight` | Primary + clarear 20% (HSL: L+20) | Gradiente CTA, tags |
| `primaryDark` | Primary + escurecer 30% (HSL: L-30) | Gradiente CTA, fundo escuro |
| `lightBg` | Off-white com tint da primary (mix 5% primary com #FFFFFF) | Fundo slides claros |
| `darkBg` | Near-black com tint da primary (mix 10% primary com #0A0A0A) | Fundo slides escuros |

Textos: `textLight` = `#FFFFFF` (sobre fundo escuro), `textDark` = `#1A1A1A` (sobre fundo claro).

### 0.7 Gerar design-system.html

Ler o template `design-system-template.html` e substituir TODOS os placeholders `{{...}}`:

| Placeholder | Fonte |
|-------------|-------|
| `{{BRAND_NAME}}` | brand.json → projectName |
| `{{BRAND_PRIMARY}}` | design-system.json → palette.primary |
| `{{BRAND_PRIMARY_LIGHT}}` | design-system.json → palette.primaryLight |
| `{{BRAND_PRIMARY_DARK}}` | design-system.json → palette.primaryDark |
| `{{BRAND_LIGHT_BG}}` | design-system.json → palette.lightBg |
| `{{BRAND_DARK_BG}}` | design-system.json → palette.darkBg |
| `{{BRAND_TEXT_LIGHT}}` | design-system.json → palette.textLight |
| `{{BRAND_TEXT_DARK}}` | design-system.json → palette.textDark |
| `{{BRAND_FONT_HEADING}}` | brand.json → fonts.heading.fontFamily |
| `{{BRAND_FONT_BODY}}` | brand.json → fonts.body.fontFamily |
| `{{BRAND_STYLE}}` | brand.json (inferir do KB/segmento) |
| `{{BRAND_SEGMENT}}` | brand.json → knowledge ou cuisineType |
| `{{BRAND_INSTAGRAM}}` | brand.json → @instagramUsername |
| `{{LOGO_BASE64}}` | brand.json → logo.base64 |
| `{{FONT_HEADING_BASE64}}` | brand.json → fonts.heading.base64 |
| `{{FONT_BODY_BASE64}}` | brand.json → fonts.body.base64 |

Salvar em `projects/{id}-{slug}/design-system.html` e abrir no browser para revisao:

```bash
open ~/.claude/skills/instagram-carousel/projects/{id}-{slug}/design-system.html
```

### 0.8 Atualizar Cache

O usuario pode pedir para atualizar o cache:
- "atualizar dados do projeto X" → executar Setup (0.3) novamente
- "atualizar fontes do projeto X" → baixar fontes novamente
- "atualizar cores do projeto X" → buscar cores e re-derivar palette

---

## Fase 1: Copys + Escolha de Preset

### 1.1 Contexto do Projeto

Com o projeto carregado (Fase 0), voce ja tem:
- **Tom de voz** do KB → brand.json.knowledge.tomDeVoz
- **Info do estabelecimento** → brand.json.knowledge
- **Presets de layout** → design-system.json.layoutPresets

Se precisar de info adicional sobre um topico especifico (ex: CARDAPIO), busque:
```
get-knowledge(projectId, category="CARDAPIO")
```

### 1.2 Escolher Formato e Preset

Sugira o formato mais adequado ao tema:

| Formato | Estrutura | Quando usar |
|---------|-----------|-------------|
| **Standard** | Hook → conteudo → CTA | Conteudo geral, promocoes |
| **Listicle** | Hook → itens numerados → CTA | "5 motivos...", "7 pratos..." |
| **Tutorial** | Hook → passos sequenciais → CTA | Receitas, dicas, how-to |
| **Comparativo** | Hook → antes/depois → CTA | Transformacoes, upgrades |

Apresente os **presets de layout** disponíveis no design system do projeto:

```
Presets disponiveis:
• standard — bottom → top → left → right → CTA (conteudo geral)
• editorial — bottom → left → bottom → right → CTA (sofisticado)
• bold — CTA → top → bottom → top → CTA (alto impacto)

Qual preset voce quer? E quantos slides?
```

**Pergunte ao usuario quantos slides** antes de escrever. Sugira baseado no tema. O preset define overlay dos primeiros e ultimo slide; slides intermediarios repetem o ciclo.

### 1.3 Escrever as Copys

Para cada slide, escreva:
- **Pre-titulo** — tag/numero em caixa alta (ex: "01", "DICA", "ANTES")
- **Headline** — texto principal, curto e impactante
- **Body** (se aplicavel) — complemento com detalhes

#### Regras de Redacao

- **Acentuacao correta** — "promocao", "nao", "voce", "ate" (nunca omitir)
- **Pontuacao** — virgulas, pontos e travessoes corretamente
- **Concordancia** — verbal e nominal sempre verificada
- **Maiusculas** — headlines em caixa alta sao design, body text segue regras normais
- **Tom de voz** — respeitar o KB do projeto (informal ≠ incorreto)
- **Emojis** — com moderacao, apenas onde fazem sentido

#### Checklist antes de apresentar

1. Cada headline faz sentido isoladamente?
2. Headline + Body formam leitura fluida?
3. Acentuacao e pontuacao corretas?
4. Concordancia verbal e nominal OK?
5. Tom de voz condiz com o projeto?
6. CTA tem info pratica (horario, endereco, @)?

### 1.4 Apresentar para Aprovacao

Apresente em tabela com preset e caption sugerida:

| Slide | Preset | Pre-titulo | Headline | Body |
|-------|--------|-----------|----------|------|
| 1 | photo + overlay-bottom | DICA | HEADLINE | Body... |
| 2 | photo + overlay-top | 01 | TITULO | Body... |
| ... | ... | ... | ... | ... |
| 5 | gradient-brand | — | CTA | Info pratica |

> **"Aqui estao os textos com o preset 'standard'. Quer ajustar algo antes de seguir para as imagens?"**

NAO avance sem aprovacao.

---

## Fase 2: Curadoria de Imagens

Apos aprovacao das copys:

### 2.1 Buscar Imagens

1. `search-catalog(projectId, filters)` → filtrar por tema de cada slide
2. `list-drive-images(projectId, folderId, limit=500)` → acervo completo
3. Para cada slide com foto (hasPhoto=true no preset), selecione 2-3 candidatas

### 2.2 Gerar Pagina de Curadoria

O terminal nao exibe imagens. Gere uma **pagina HTML interativa** servida via localhost.

#### Preparar dados

**gallery.js** — acervo completo do Drive:
```javascript
var G = [{id:"driveFileId", name:"fileName", folder:"folderName", thumb:"thumbnailLink"}, ...];
var FC = {"Almoco": 82, "Espetos": 104, ...};
```

**slides.js** — dados dos slides com candidatas:
```javascript
var META = {project: "By Rock", theme: "Happy Hour"};
var S = [
  {num:1, type:"HOOK", headline:"...", body:"...", overlay:"bottom", candidates:[
    {id:"driveFileId", name:"foto.jpg", desc:"Descricao", rec:true, thumb:"thumbnailLink"}
  ]},
  // ... demais slides (apenas os que tem hasPhoto=true)
];
```

#### Servir a pagina

```bash
# Matar servidor anterior
lsof -ti:8787 | xargs kill -9 2>/dev/null

# Criar pasta de trabalho
mkdir -p /tmp/carousel-{slug}

# Copiar template (NAO reescrever)
cp ~/.claude/skills/instagram-carousel/curadoria-template.html /tmp/carousel-{slug}/index.html

# Gerar apenas os dados (via Python write_text ou similar)
# gallery.js e slides.js

# Servir
cd /tmp/carousel-{slug} && python3 -m http.server 8787 &
open http://localhost:8787/index.html
```

### 2.3 Apresentar para Aprovacao

> **"Abri a pagina de curadoria no browser. Selecione as imagens e clique em 'Copiar tudo'. Cole aqui quando estiver pronto."**

NAO avance sem aprovacao.

---

## Fase 3: Montagem HTML e Export

Com copys aprovadas, preset escolhido e imagens selecionadas:

### 3.1 Carregar Design System

Ler `design-system.json` do cache local. Todos os valores de cor, fonte e overlay vem daqui.

### 3.2 Gerar HTML de Cada Slide

Para cada slide, usar o layout definido pelo preset:

**Regras tecnicas:**

**Layout:**
- Cada slide: 420x525px (proporcao 4:5, escala para 1080x1350)
- Padding do conteudo: `0 36px 52px` (52px bottom para progress bar)
- Logo no topo centro de cada slide (36px height, opacity 0.9)

**Imagens de fundo:**
- Sempre embeder como **base64 data: URIs** via `<img>` com `object-fit:cover`
- NUNCA usar `background: url(...)` com base64 (crasha o parser)
- Usar Python `Path.write_text()` para gerar o HTML — shell scripts corrompem base64

**Overlay (do design-system.json):**
- Cada slide usa o overlay definido no preset: bottom, top, left, right, ou gradientBrand
- Gradientes seguem R2: opacidade 92% → 75% → 0%
- NUNCA cobrir mais de 50% da foto com gradiente pesado
- `text-shadow: 0 2px 12px rgba(0,0,0,0.3)` para legibilidade

**Alinhamento de texto (por overlay):**
- `overlay-bottom` → texto no bottom-left, `margin-top: auto`
- `overlay-top` → texto no top-left, `margin-bottom: auto`
- `overlay-left` → texto center-left, width 75%
- `overlay-right` → texto center-right, width 80%, text-align right
- `gradientBrand` → texto center, align-items center

**Slide CTA (gradientBrand):**
- Background: gradiente da marca (primaryDark → primary → primaryLight)
- Logo + headline + info centralizado (flexbox column, center)
- Sem seta de swipe, progress bar 100%
- @handle do Instagram em cor clara

**Elementos de UI em cada slide:**

#### Progress Bar (rodape)
```
position: absolute bottom 12px, left/right 16px
track: 2px height, rounded, rgba(255,255,255,0.12) em dark / rgba(0,0,0,0.08) em light
fill: percentage = (slideIndex+1)/total * 100
counter: "1/7" 8px font-body, rgba(255,255,255,0.5)
```

#### Seta de Swipe (direita — todos exceto ultimo)
```
width: 32px, full height, position absolute right
background: linear-gradient(to right, transparent, rgba(255,255,255,0.04))
chevron: SVG "M9 6l6 6-6 6", 16x16, stroke rgba(255,255,255,0.25)
```

**Fontes e logo:**
- Embeder do cache local (brand.json → fonts.heading.base64, fonts.body.base64, logo.base64)
- @font-face no `<style>` de cada slide HTML

**Cores:**
- Substituir TODAS as variaveis por hex reais antes de renderizar
- O HTML final NAO pode conter nenhuma variavel CSS ou placeholder

### 3.3 Preview com Frame Instagram

Gere HTML de preview com frame interativo do Instagram:

- **Header**: avatar (circulo com logo) + @handle + localizacao
- **Viewport**: 420x525px com carousel track horizontal + drag/swipe funcional
- **Dots**: indicadores de slide abaixo do viewport
- **Actions**: icones SVG (coracao, comentario, compartilhar, salvar)
- **Caption**: @handle + resumo + timestamp

Swipe via pointer events com threshold 50px.

```bash
open /tmp/carousel-{slug}/preview.html
```

### 3.4 Revisao Interativa (pagina HTML)

Antes de exportar, gere uma **pagina HTML de revisao** servida via localhost. Esta pagina permite ao usuario fazer ajustes finais **visuais** antes do export.

**Template:** `revisao-template.html` (diferente da curadoria — inclui preview visual com design system).

**Arquivos necessarios** (3 JS + template):

```
/tmp/carousel-{slug}/
  ├── index.html        ← copia do revisao-template.html
  ├── gallery.js        ← G[], FC{} (catalogo Drive do projeto)
  ├── slides.js         ← S[], META{} (slides com imagens ja selecionadas)
  └── design.js         ← DS{} (tokens do design system + fontes base64)
```

**Gerar design.js** com tokens do design system:

```javascript
var DS = {
  palette: { primary, primaryLight, primaryDark, lightBg, darkBg, textLight, textDark },
  typography: {
    headingFont: "Nome da Fonte",
    bodyFont: "Nome da Fonte",
    headingB64: "data:font/opentype;base64,...",   // de brand.json
    bodyB64: "data:font/truetype;base64,..."       // de brand.json
  },
  overlays: {
    bottom: "linear-gradient(...)",   // de design-system.json
    top: "...", left: "...", right: "...",
    gradientBrand: "linear-gradient(165deg,...)"
  },
  logoB64: "data:image/png;base64,...",   // de brand.json (logos.main.base64)
  projectName: "Nome do Projeto",
  driveFolderName: "Nome da Pasta no Drive",       // exibido no modal
  imagesFolderId: "googleDriveFolderId"            // para gerar gallery.js
};
```

**Gerar slides.js** com dados dos slides:

```javascript
var META = { project: "Nome", theme: "Tema do Carrossel" };
var S = [
  {
    num: 1, type: "HOOK",
    headline: "TITULO\\nEM LINHAS",
    body: "Subtitulo com acentuacao correta.",
    pretitle: "PRETITULO",
    overlay: "bottom",
    overlayIntensity: 72,          // 0-100, padrao 72
    showLogo: true,
    showSwipe: true,
    cta: null,                     // apenas no slide CTA
    candidates: [],
    selectedImage: { id: "driveFileId", name: "foto.jpg", folder: "pasta", thumb: "thumbUrl" }
  },
  // ... demais slides
];
```

**Funcionalidades da pagina de revisao:**

- **Preview 270x337px** de cada slide com design system (overlay, fontes, cores, logo)
- **Headline e body editaveis** — textarea com atualizacao live do preview
- **Seletor de overlay** — Bottom, Top, Left, Right, Brand (botoes)
- **Slider de intensidade** — range 20-100% com preview em tempo real
- **Trocar imagem** — modal "Buscar no Drive" com 150x150 grid, tabs por pasta, busca
- **Remover imagem** — botao para limpar selecao
- **Copiar tudo** — gera texto formatado com headline, body, overlay (%), pretitle, imagem

**Padrao tecnico** (consistente com curadoria):
- Servir via `python3 -m http.server 8787` (nao file://)
- addEventListener (nunca onclick inline)
- referrerpolicy="no-referrer" e loading="lazy" nas imgs
- Grid do modal: 150x150 fixo, flex column com min-height:0 para scroll

**Formato da saida "Copiar tudo":**

```
Projeto — Tema

Slide 1 [HOOK]
  Headline: TITULO EM LINHAS
  Body:
  Overlay: bottom (51%)
  Pretitle: PORTFOLIO
  Logo: sim
  Imagem: foto.jpg | driveFileId

Slide 2 [CONTENT]
  ...
```

**Instrucao ao usuario:**

> **"Abri a pagina de revisao no browser. Ajuste textos, intensidade do overlay ou troque imagens se precisar. Clique em 'Copiar tudo' e cole aqui quando pronto."**

Apos o usuario colar o resultado final, aplicar as alteracoes e seguir para export.

### 3.5 Exportar PNGs

Apos aprovacao:

1. Gerar HTML separado por slide (sem frame, so o slide puro)
2. Playwright com viewport 420x525 e `device_scale_factor = 1080/420` (~2.57)
3. Screenshot com `clip: {x:0, y:0, width:420, height:525}`
4. Wait 2000ms para fontes carregarem

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

OUTPUT_DIR = Path("~/Downloads/carousel-{slug}").expanduser()
OUTPUT_DIR.mkdir(exist_ok=True)

async def export():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        for i in range(TOTAL):
            page = await browser.new_page(
                viewport={"width": 420, "height": 525},
                device_scale_factor=1080/420
            )
            html = Path(f"/tmp/carousel-{slug}/slide_{i+1}.html").read_text()
            await page.set_content(html, wait_until="networkidle")
            await page.wait_for_timeout(2000)
            await page.screenshot(
                path=str(OUTPUT_DIR / f"slide-{i+1:02d}.png"),
                clip={"x": 0, "y": 0, "width": 420, "height": 525}
            )
            await page.close()
        await browser.close()

asyncio.run(export())
```

Cada PNG tera 1080x1350px, pronto para upload.

---

## Dicas de Design

- **Hook mata ou salva** — o primeiro slide decide o swipe
- **Foto > gradiente** — em slides com foto, o gradiente e leve. A foto vende
- **Consistencia** — cores, fontes e ritmo uniformes em todos os slides
- **Menos texto = mais impacto** — max 2-3 linhas por slide
- **CTA com info pratica** — horario, endereco, @handle
- **Ritmo de overlay** — alternar posicao para manter interesse visual

---

## Instalacao do Playwright (se necessario)

```bash
pip3 install playwright && playwright install chromium
```

Precisa ser feito apenas uma vez.

---

## Acoes Rapidas

Quando o usuario pedir uma dessas acoes, execute diretamente sem perguntar:

### "abre/mostra/ver o design system do [projeto]"

IMPORTANTE: O design system de cada projeto fica em ~/.claude/skills/instagram-carousel/projects/
NAO abrir arquivos em lagostacriativa.com.br/ ou em qualquer outra pasta — esses sao arquivos de referencia antigos.

Mapeamento de projetos (usar exatamente estes comandos):

**Lagosta Criativa:**
```bash
open /Users/cirotrigo/.claude/skills/instagram-carousel/projects/8-lagosta-criativa/design-system.html
```

Se o projeto nao estiver listado acima, avisar e oferecer para criar (Setup 0.3).

### "atualizar dados do projeto X"
Executar Setup Automatico (0.3) novamente para o projeto.

### Matar servidor HTTP (antes de levantar novo)
```bash
lsof -ti:8787 | xargs kill -9 2>/dev/null
```

### Limpar pasta de trabalho
```bash
rm -rf /tmp/carousel-{slug}/
```
