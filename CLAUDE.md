# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão geral

"Caleidoscópio Vivo": arte generativa interativa em um único arquivo, `index.html` (HTML + CSS + JS puros, sem dependências). Partículas seguem um campo de fluxo e são desenhadas com simetria rotacional/espelhada num `<canvas>` 2D. Toda a interface e os comentários estão em português do Brasil. Mantenha esse padrão.

## Como rodar

Não há build, gerenciador de pacotes nem testes. Abra `index.html` direto no navegador e recarregue a página após cada alteração. A verificação é visual: confira a animação, os controles do painel e os atalhos de teclado.

## Arquitetura (tudo dentro de uma IIFE no `<script>`)

- **`state`**: fonte única de verdade para todos os parâmetros (paleta, sliders, toggles, pausa). Loop e desenho leem sempre daqui.
- **Loop**: `loop()` → `step()` (move partículas) → `draw()` (desenha), via `requestAnimationFrame`. O tempo `t` e o `hueShift` só avançam quando não está pausado.
- **Campo de fluxo**: `flowAngle(x, y)` é uma soma de senos/cossenos com `seedA`/`seedB` e `t`. Não é ruído Perlin. `state.chaos` controla a escala e a amplitude. Partículas renascem via `spawn()` quando `life` zera ou quando saem do raio `maxR`.
- **Rastros**: não se limpa o canvas a cada quadro. `draw()` pinta um retângulo semitransparente com `palette.bg` (a opacidade vem de `state.trail`), e `trail = 100` desliga o esmaecimento. Por isso o `bg` da paleta afeta tanto o fundo quanto a cor do rastro.
- **Simetria**: cada segmento da partícula (`px,py` → `x,y`) é rotacionado `state.sym` vezes em torno do centro (`cx, cy`), com o reflexo adicional quando `mirror` está ativo, tudo num único `beginPath/stroke` por partícula. `glow` troca o composite para `'lighter'`.
- **Coordenadas**: tudo em pixels CSS. `resize()` aplica `setTransform(dpr)` com `dpr` limitado a 2, e também limpa a tela e recria todas as partículas.
- **Pintura com mouse/toque**: `splat()` não cria partículas novas. Ele substitui entradas existentes do array (índice `splatIdx` avançando de 7 em 7) para manter a contagem igual a `state.count`.

## Convenções para mexer nos controles

- **Novo slider**: exige três partes em sincronia: `<input type="range" id="X">` + `<span id="v-X">` no HTML, uma entrada `X: formatador` no objeto `sliders` e a chave `X` em `state`. O laço sobre `sliders` liga o evento e já inicializa o valor.
- **Novo toggle**: checkbox com `id` + chave em `state` + id na lista `['mirror', 'glow', 'drift']`.
- **Alterar um slider por código** (como em `surprise()`): use `setSlider(id, v)`, que dispara o evento `input` e mantém `state` e o rótulo sincronizados. Não altere `state` diretamente.
- **Nova paleta**: adicione a `PALETTES` com `{ name, hue, spread, sat, light, bg }`. `spread >= 360` é tratado como arco-íris no botão de pré-visualização.
- **Atalhos de teclado**: ficam no listener `keydown`. Se mudar algum, atualize também o texto em `.keys` no painel.
- **Layout mobile**: abaixo de 600px o painel vira uma gaveta inferior (media query no CSS). Teste as mudanças de UI nas duas larguras.
