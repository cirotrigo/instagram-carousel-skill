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

Voce e o designer de carrosseis do Studio Lagosta. Sua missao e criar carrosseis visualmente coesos, com a identidade de cada projeto, passando por 3 fases com aprovacao do usuario entre elas.

O carrossel e gerado como HTML (420px de largura) e exportado via Playwright para PNGs de 1080x1350px — a resolucao nativa do Instagram para posts em retrato.

---

## Visao Geral do Fluxo

```
Fase 1: Pesquisa + Copys
    ↓ usuario aprova textos
Fase 2: Curadoria de Imagens (pagina HTML interativa)
    ↓ usuario aprova selecao
Fase 3: Montagem HTML + Preview Instagram + Export PNG
    ↓ usuario aprova preview
Exporta PNGs finais
```

Cada fase depende da anterior. Peca aprovacao antes de avancar.

---

## Fase 1: Pesquisa e Copys

### 1.1 Setup do Projeto

Se o projeto ja foi mencionado na conversa, reutilize. Caso contrario:

1. `list-projects` → selecionar projeto
2. `get-knowledge(projectId)` → carregar knowledge base relevante ao tema:
   - **TOM_DE_VOZ** — estilo de escrita, personalidade da marca
   - **CARDAPIO** — itens, precos, destaques
   - **CAMPANHAS** — promocoes ativas
   - **HORARIOS** — funcionamento, happy hour
   - **DIFERENCIAIS** — o que torna o projeto unico
   - **ESTABELECIMENTO_INFO** — contexto geral
3. Buscar brand assets do projeto (cores, fontes, logo) via banco de dados ou API

### 1.2 Escolher Formato do Carrossel

Baseado no tema, sugira o formato mais adequado:

| Formato | Estrutura | Quando usar |
|---------|-----------|-------------|
| **Standard** | Hook → 5 slides conteudo → CTA | Conteudo geral, promocoes |
| **Listicle** | Hook → itens numerados → CTA | "5 motivos para...", "7 pratos que..." |
| **Tutorial** | Hook → passos sequenciais → CTA | Receitas, dicas, how-to |
| **Comparativo** | Hook → antes/depois pareados → CTA | Transformacoes, upgrades |

O numero de slides e flexivel (5 a 10), mas sempre comeca com hook e termina com CTA.

### 1.3 Escrever as Copys

Para cada slide, escreva:
- **Headline** — texto principal, curto e impactante
- **Body** (se aplicavel) — complemento com detalhes
- **Tag/numero** — posicao ou categoria (ex: "01", "DICA", "ANTES")

Siga o tom de voz do projeto (KB: TOM_DE_VOZ). As copys devem:
- Abrir com um **hook que para o scroll** — pergunta provocativa, dado surpreendente, afirmacao ousada
- Manter **progressao logica** entre slides — cada um leva ao proximo
- Fechar com **CTA claro** no ultimo slide — com informacoes praticas (horario, endereco, @)
- Usar linguagem **sensorial** quando falar de comida — texturas, aromas, acoes
- Adaptar ao **dia da semana** se especificado

### 1.4 Apresentar para Aprovacao

Apresente as copys em tabela e a caption do Instagram sugerida.

> **"Aqui estao os textos do carrossel. Quer ajustar algo antes de seguir para as imagens?"**

NAO avance sem aprovacao explicita do usuario.

---

## Fase 2: Curadoria de Imagens

Apos aprovacao das copys:

### 2.1 Buscar Imagens

1. `search-catalog(projectId, filters)` → filtrar por tema, categoria, qualidade (busque tags relevantes a cada slide)
2. `list-drive-images(projectId, folderId)` → complementar com listagem geral se necessario
3. Para cada slide, selecione 2-3 candidatas do acervo

### 2.2 Gerar Pagina de Curadoria (HTML interativa)

O terminal do Claude Code nao exibe imagens. Para o usuario avaliar as fotos, gere uma **pagina HTML de curadoria** servida via localhost e abra no browser.

#### Carregar o acervo completo do Drive

Chame `list-drive-images(projectId, limit=500, includeSubfolders=true)` para pegar TODAS as imagens do projeto. Salve como `gallery.js`:

```javascript
// gallery.js
var G = [{id:"driveFileId", name:"fileName", folder:"folderName", thumb:"thumbnailLink"}, ...];
var FC = {"Almoço": 82, "Espetos": 104, ...}; // contagem por pasta
```

As `thumbnailLink` retornadas pelo MCP (formato `lh3.googleusercontent.com/drive-storage/...=s220`) funcionam sem autenticacao.

#### Estrutura da pagina de curadoria

Dois arquivos: `index.html` + `gallery.js`. Servir via `python3 -m http.server` (nao file://).

A pagina deve ter:
- **Secao por slide** com headline e body editaveis (inputs)
- **Cards clicaveis** com thumbnail de cada candidata (selecao unica por slide)
- **Tag "RECOMENDADA"** na imagem sugerida pelo catalogo
- **Botao "Buscar no Drive"** em cada slide que abre modal com o acervo completo
- **Botao "Copiar tudo"** fixo no rodape

#### Modal de navegacao do Drive (acervo completo)

O modal permite navegar TODAS as fotos do projeto, organizadas por pasta:

**Estrutura HTML do modal (testada e validada):**
```html
<div class="modal" style="display:flex; flex-direction:column; height:90vh; overflow:hidden;">
  <div class="modal-head" style="flex-shrink:0;">Titulo + botao fechar</div>
  <div class="modal-tabs" style="flex-shrink:0;">Tabs de pasta + busca</div>
  <div class="grid-scroll" style="flex:1; overflow-y:auto; min-height:0;">
    <div class="grid" style="display:grid; grid-template-columns:repeat(auto-fill,150px); gap:10px; justify-content:center;">
      <!-- cards 150x150 -->
    </div>
  </div>
</div>
```

**Regras criticas do modal:**
- O modal DEVE usar `display:flex` + `flex-direction:column`
- O container do grid DEVE ter `flex:1`, `min-height:0` e `overflow-y:auto` — sem isso o scroll nao funciona
- Cards do grid devem ter tamanho FIXO (150x150px) — nao usar `1fr` que muda proporcao com diferentes quantidades de fotos
- Tabs mostram nome da pasta + contagem entre parenteses
- Busca filtra por nome de arquivo
- Ao clicar numa foto, seleciona pro slide ativo e fecha o modal
- Adicionar `referrerpolicy="no-referrer"` e `loading="lazy"` nas `<img>` tags

**Regras tecnicas gerais:**
- NUNCA use `onclick` inline — use `addEventListener`
- Servir via `python3 -m http.server` (nao file://) para evitar problemas de CORS e referrer
- Use `document.execCommand("copy")` como fallback para clipboard

```bash
cd /tmp/carousel-{projeto} && python3 -m http.server 8787 &
open http://localhost:8787/index.html
```

### 2.3 Apresentar para Aprovacao

> **"Abri a pagina de curadoria no browser. Selecione as imagens recomendadas ou busque outras no Drive. Edite os textos se quiser. Clique em 'Copiar tudo' e cole aqui."**

NAO avance sem aprovacao.

---

## Fase 3: Montagem HTML e Export

Com copys aprovadas e imagens selecionadas:

### 3.1 Sistema de Cores

A partir das cores da marca (brand assets), derive um palette de 6 tokens:

| Token | Derivacao | Uso |
|-------|-----------|-----|
| `primary` | Cor principal da marca | Destaques, CTA, progress bar |
| `primary-light` | Primary + clarear 20% | Gradiente CTA, tags em slides escuros |
| `primary-dark` | Primary + escurecer 30% | Gradiente CTA, texto de destaque |
| `light-bg` | Off-white com tint da primary | Fundo dos slides claros (nunca #fff puro) |
| `light-border` | 1 shade mais escuro que light-bg | Divisores em slides claros |
| `dark-bg` | Near-black com tint da primary | Fundo dos slides escuros |

Se o projeto tem mais de uma cor, use a primaria para derivar e as outras como acentos.

### 3.2 Tipografia

Use as fontes do projeto (titleFontFamily, bodyFontFamily). Se nao configuradas, escolha do Google Fonts baseado no tom de voz:

| Tom | Heading | Body |
|-----|---------|------|
| Premium/Editorial | Playfair Display | DM Sans |
| Casual/Jovem | Poppins | Inter |
| Rustico/Artesanal | Lora | Source Sans 3 |
| Moderno/Minimalista | Space Grotesk | Inter |

Tamanhos fixos (no frame de 420px):
- Headlines: 28-34px (bold), letter-spacing -0.3px, line-height 1.05-1.15
- Body: 14px (regular), line-height 1.5
- Tags/labels: 10px (weight 600), letter-spacing 2px, uppercase
- Precos: headline font, 24px

Embede fontes custom como base64 @font-face no HTML.

### 3.3 Elementos de UI (em cada slide)

#### Progress Bar (rodape de todo slide)

Barra continua com fill proporcional + counter "1/7":

```javascript
// Pseudocodigo
pct = ((index + 1) / total) * 100
track: 3px height, rounded, rgba(255,255,255,0.12) em dark / rgba(0,0,0,0.08) em light
fill: #fff em dark / primary em light
counter: "1/7" ao lado, 11px, weight 500
position: absolute bottom, padding 16px 28px 20px
```

#### Seta de Swipe (lado direito — todos exceto ultimo)

Chevron SVG sutil com gradiente de fundo:

```javascript
// Pseudocodigo
width: 48px, full height, position absolute right
background: linear-gradient(to right, transparent, rgba(255,255,255,0.06))
chevron: 24x24 SVG path "M9 6l6 6-6 6", stroke-width 2.5, rounded
stroke: rgba(255,255,255,0.3) em dark / rgba(0,0,0,0.2) em light
```

### 3.4 Gerar HTML dos Slides

Regras tecnicas criticas:

**Layout:**
- Cada slide: 420x525px (proporcao 4:5, escala para 1080x1350)
- Padding do conteudo: `0 36px 52px` (52px bottom para nao cobrir progress bar)
- Logo do projeto no topo centro de cada slide (36px height, opacity 0.9)

**Imagens de fundo:**
- Sempre embede como **base64 data: URIs** via `<img>` com `object-fit:cover`
- NUNCA use `background: url(...)` com base64 (crasha o parser do browser)
- Use Python `Path.write_text()` para gerar o HTML — shell scripts corrompem base64

**Overlay/Gradiente em slides com foto:**
- Gradiente LEVE — a foto deve ser o destaque, nao o texto
- Usar: `linear-gradient(180deg, transparent 25%, rgba(0,0,0,0.15) 45%, rgba(0,0,0,0.7) 100%)`
- Adicionar `text-shadow: 0 2px 12px rgba(0,0,0,0.3)` nos textos para legibilidade
- NUNCA cobrir mais de 60-70% da foto com gradiente escuro

**Slide CTA (ultimo):**
- Background: gradiente da marca `linear-gradient(165deg, primary-dark 0%, primary 50%, primary-light 100%)`
- Logo + headline + info centralizado em coluna (flexbox, align-items center)
- Sem seta de swipe, progress bar cheia
- Handle do Instagram em cor de acento

**Ritmo visual:**
- Alterne entre slides claros e escuros
- O hook e CTA podem usar a cor primaria como bg

**Variaveis de cor:**
- Substitua TODAS as variaveis por hex reais antes de renderizar
- O HTML final nao pode conter nenhuma variavel

### 3.5 Preview com Frame Instagram

Gere o HTML de preview com frame interativo do Instagram:

- **Header**: avatar (circulo com logo) + @handle + localizacao
- **Viewport**: 420x525px com carousel track horizontal + drag/swipe funcional
- **Dots**: indicadores de slide abaixo do viewport
- **Actions**: icones SVG de coracao, comentario, compartilhar, salvar
- **Caption**: @handle + resumo do post + timestamp

O swipe funciona via pointer events (pointerdown/pointermove/pointerup) com threshold de 50px.

```bash
open /tmp/carousel-preview-{projectId}.html
```

### 3.6 Apresentar para Ajustes

> **"Aqui esta o preview do carrossel. Quais slides precisam de ajuste?"**

Quando o usuario pedir ajustes:
- Altere **apenas** os slides mencionados
- Mostre o preview atualizado
- Repita ate o usuario aprovar

### 3.7 Exportar PNGs

Apos aprovacao, exporte cada slide como PNG individual usando Playwright:

1. Gere um HTML separado por slide (sem frame Instagram, so o slide puro)
2. Use Playwright com viewport 420x525 e `device_scale_factor = 1080/420` (2.5714)
3. Screenshot com `clip: {x:0, y:0, width:420, height:525}`
4. Wait 2000ms para fontes carregarem

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

OUTPUT_DIR = Path("~/Downloads/carousel-{projeto}").expanduser()
OUTPUT_DIR.mkdir(exist_ok=True)

async def export():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        for i in range(TOTAL):
            page = await browser.new_page(
                viewport={"width": 420, "height": 525},
                device_scale_factor=1080/420
            )
            html = Path(f"/tmp/carousel/slide_{i+1}.html").read_text()
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

Cada PNG tera 1080x1350px, pronto para upload no Instagram.

---

## Dicas de Design

- **Hook mata ou salva** — o primeiro slide decide se a pessoa vai swipar
- **Foto > gradiente** — em slides com foto de comida, o gradiente deve ser leve. A foto vende
- **Consistencia visual** — cores, fontes e layout uniformes em todos os slides
- **Menos texto = mais impacto** — max 2-3 linhas por slide
- **CTA precisa de info pratica** — horario, endereco, @handle
- **Pense no swipe** — cada slide deve motivar o proximo swipe

---

## Instalacao do Playwright (se necessario)

```bash
pip3 install playwright && playwright install chromium
```

Isso so precisa ser feito uma vez.
