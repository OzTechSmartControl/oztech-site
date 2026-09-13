# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este repositório

Site institucional principal da OzTech SmartControl (`www.oztechsmartcontrol.com.br`). É um site estático puro — sem build, sem framework, sem dependências de pacote. Repositório GitHub: `OzTechSmartControl/oztech-site`.

Este é um repositório **separado** do produto "Portal de Sites" (`sites.oztechsmartcontrol.com.br`), que vive em `../Sites /Portal de Sites (Publicado)` — esse sim é uma aplicação Node/Vercel completa (checkout, geração de sites, admin, etc.). Este repositório aqui só faz link/referência pra lá; não edite o Portal de Sites a partir daqui.

## Comandos

Não há `package.json`, build, lint ou testes automatizados — é HTML/CSS/JS estático servido como está.

- **Rodar localmente**: `npx serve -p 3131 .` (já configurado como server "oz-site" em `.claude/launch.json`).
- **Deploy**: push na branch `main` → deploy automático na Vercel (integração GitHub, sem `vercel.json`, sem comando de build configurado — passthrough estático direto). Não existe ambiente de preview/staging separado neste repo; `main` é produção.

## Arquitetura

### Duas páginas HTML independentes, sem template compartilhado

`index.html` (site principal, single-page com âncoras) e `loja.html` (catálogo standalone) são arquivos completos e independentes — cada um com seu próprio `<head>`, CSS inline, config do `particles.js` e nav duplicados. **Não existe include/partial/build step** que sincronize os dois. Qualquer mudança que deva valer para as duas páginas (nav, footer, fontes, etc.) precisa ser editada manualmente em cada arquivo.

`loja.html` está **desatualizado e não é mais linkado pelo nav** de `index.html` desde 2026-09-12 (o menu "Loja de Sites" e a aba "🌐 Sites Prontos" agora apontam direto para `https://sites.oztechsmartcontrol.com.br`, o catálogo real e atualizado). `loja.html` continua acessível publicamente (e ainda está no `sitemap.xml`) mas só lista 4 nichos com link real — não reflita esse arquivo como fonte de verdade do catálogo, e não reintroduza links pra ele sem confirmar com o usuário.

### `index.html` — estrutura da página única

Seções navegadas por âncora: `#o-que-fazemos`, `#websites`, `#sobre`, `#catalogo`, `#planos`, `#como-funciona`, `#contato`. Todo o CSS e JS ficam inline no próprio arquivo (~3000 linhas).

A seção `#catalogo` tem 3 abas:
- **"Para Empresas"** (`panel-pj`, alternada via `switchTab()`) — vitrine dos outros produtos B2B SaaS da OzTech (Oz.Barber, Oz.Beauty, etc.), cada um implantado e hospedado independentemente, fora deste repositório.
- **"Sob Medida"** (`panel-custom`).
- **"🌐 Sites Prontos"** — não é uma aba JS, é um link externo direto para `https://sites.oztechsmartcontrol.com.br` (o Portal de Sites).

### i18n (PT/EN)

Tradução feita por um único objeto `I18N = { pt: {...}, en: {...} }` (por volta da linha 2690 de `index.html`), sem arquivos de locale externos. Elementos marcam a chave via `data-i18n`, `data-i18n-placeholder` ou `data-i18n-aria`; `applyLang(lang)` percorre o DOM e reescreve `innerHTML`/`placeholder`/`aria-label`, persiste a escolha em `localStorage["oz-lang"]`, e a IIFE `initLang()` no fim do arquivo aplica o idioma salvo no carregamento. `loja.html` não tem i18n — é só português.

Ao adicionar texto novo com `data-i18n`, é preciso cadastrar a mesma chave nos dois blocos (`pt` e `en`) do objeto `I18N` — se a chave não existir em `en`, `applyLang` deixa o texto original (em português) no lugar, silenciosamente.

### Widget de chat "Zé"

Funções `ze*` (`zeToggle`, `zeSend`, `zeCallAPI`, etc.) implementam um widget de chat que consome uma API externa em `ZE_API_URL = 'https://assistente.oztechsmartcontrol.com.br/api/web-chat/message'` — esse backend roda na VPS da OzTech, fora deste repositório e fora do Portal de Sites. Identidade da conversa é um UUID por navegador (`zeSessionId()`, `localStorage["oz_ze_session_id"]`) — nunca um telefone real; o servidor guarda o histórico, o cliente só manda a última mensagem.

Quando o backend acabou de gerar um protocolo de atendimento humano, a resposta JSON traz `actions: [{type, label, href}]` (`whatsapp`/`email`/`phone`) além de `reply` — `zeCallAPI()` renderiza isso como botões (`.ze-action-btn`) presos àquela bolha de mensagem específica, complementando o texto. `actions` vem vazio (`[]`) em qualquer resposta que não tenha acabado de criar um protocolo. Atenção ao CSS: `.ze-bubble a` (regra genérica pra links soltos no texto da IA) tem mais especificidade que `.ze-action-btn` sozinho — a regra real precisa ser `.ze-bubble a.ze-action-btn` pra não herdar o azul sublinhado.

### Formulário de contato

Os campos do formulário em `#contato` existem no HTML (`.contact-form`), mas o botão "Enviar Mensagem" não tem nenhum handler de submit ligado a ele hoje — não há `<form>`, `fetch`, nem `mailto` associados. Os únicos canais de contato realmente funcionais nessa seção são os links de WhatsApp (`wa.me`), e-mail e Instagram listados ao lado.

### Assets de imagem da loja

Cada nicho em `loja.html` tem um par `loja-{nicho}.jpg` + `loja-{nicho}.webp` (mesmo conteúdo, dois formatos) na raiz do repo.
