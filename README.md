# Instagram Carousel Skill — Studio Lagosta

Skill do Claude Code para gerar carrosséis de Instagram com a identidade visual de cada projeto do Studio Lagosta.

## O que faz

Gera carrosséis de Instagram (1080×1350px por slide) automaticamente:

1. **Pesquisa** — puxa dados do projeto via MCP (cardápio, promoções, tom de voz, horários)
2. **Copys** — escreve textos de cada slide respeitando o tom de voz, apresenta para aprovação
3. **Curadoria de imagens** — busca fotos no acervo do Drive, monta página HTML interativa para seleção
4. **Montagem** — gera HTML com identidade visual (cores, fontes, logo) e preview com frame do Instagram
5. **Export** — exporta cada slide como PNG via Playwright

## Instalação

### 1. Copie para o diretório de skills do Claude Code

```bash
cp -r instagram-carousel ~/.claude/skills/
```

### 2. Instale Playwright (para exportar PNGs)

```bash
pip3 install playwright && playwright install chromium
```

### 3. Pronto!

A skill será ativada automaticamente quando você pedir para criar um carrossel.

## Exemplos de uso

```
> Cria um carrossel de promoções para o Espeto Gaúcho para segunda-feira
> Faz um carrossel educativo sobre harmonização de vinhos para o Wine Vix
> Monta slides de happy hour para o By Rock
```

## Requisitos

- Claude Code com MCP do Studio Lagosta configurado
- Python 3 + Playwright (para export)
- Projeto cadastrado no Studio Lagosta com brand assets (cores, fontes, logo)

## Estrutura

```
instagram-carousel/
├── .claude/
│   └── settings.json   # Permissões (python, pip, playwright)
└── SKILL.md             # Instruções da skill
```
