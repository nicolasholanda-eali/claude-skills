---
name: ghl-landing-page
description: Use when building landing pages as single HTML files for GoHighLevel (GHL), pages with Tailwind CDN, video backgrounds, pricing sections, animated sections, or any high-conversion landing page project. Also use when debugging GHL layout issues, PageSpeed performance problems, or video autoplay failures.
---

# Landing Page para GoHighLevel (GHL) - Guia Completo

## Overview

Guia para criar landing pages de alta conversão como **arquivo HTML único** para deploy no GoHighLevel. Cobre arquitetura, padroes CSS/JS, armadilhas de performance, e solucoes para problemas comuns do GHL.

## Arquitetura Base

### Stack
- **HTML unico** (sem build tools) - GHL nao suporta frameworks
- **Tailwind CDN** (`https://cdn.tailwindcss.com`) - rapido para prototipar, mas impacta TBT (~1.3s)
- **Google Fonts** via CDN (Inter recomendado)
- **Assets em CDN externo** (filesafe.space ou similar)

### Estrutura do Arquivo
```
1. <head> - Meta, Tailwind CDN, Fonts, Tailwind config, <style> customizado
2. <body> - <div class="ghl-wrapper"> envolve TUDO
3. Secoes em ordem logica de conversao
4. <script> no final do body com todo JS
```

## GHL - Problemas e Solucoes

### Full-Width Breakout (OBRIGATORIO)
GHL coloca a pagina dentro de containers com largura limitada. Para full-width:

```css
.ghl-wrapper {
  width: 100vw;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
  overflow-x: hidden;
}
```
**Envolver TODO o conteudo do body neste wrapper.**

### Transform Quebra Position Fixed
O `transform` no `.ghl-wrapper` faz com que `position: fixed` dentro dele se comporte como `absolute`. Solucao para elementos fixed (ex: cursor card da equipe):

```javascript
// Mover elemento para body via JS
document.body.appendChild(elementoFixed);
```

### Nao Temos Controle do Body
No GHL, o `<body>` nao e nosso. Nao tente estilizar o body diretamente para resolver layout. Use o wrapper.

## Estrutura de Pagina (Ordem de Conversao)

Ordem recomendada baseada em PAS (Problem-Agitation-Solution):

```
1. Navbar (fixed, glassmorphic)
2. Hero (video + headline + CTA)
3. Prova Social / Metricas
4. O que e (explicacao do produto)
5. Referencias / Cases
6. PAS - Problema (3 cards com dores)
7. PAS - Agitacao (comparacao sem vs com)
8. Como Funciona (timeline vertical)
9. Features / Agentes IA
10. Beneficios
11. Para quem e
12. Pricing (3 cards, destaque no anual)
13. Equipe
14. FAQ (accordion)
15. CTA Final
16. Footer
```

## Padroes CSS que Funcionam

### Animacoes Essenciais
```css
/* Fade-up on scroll (com IntersectionObserver) */
.fade-up {
  opacity: 0; transform: translateY(30px);
  transition: opacity 0.7s ease-out, transform 0.7s ease-out;
}
.fade-up.visible { opacity: 1; transform: translateY(0); }

/* Stagger delays */
.stagger-1 { transition-delay: 0ms !important; }
.stagger-2 { transition-delay: 100ms !important; }
.stagger-3 { transition-delay: 200ms !important; }
.stagger-4 { transition-delay: 300ms !important; }

/* CTA pulsante */
@keyframes pulse-ring {
  0% { box-shadow: 0 0 0 0 rgba(37, 99, 235, 0.5); }
  70% { box-shadow: 0 0 0 14px rgba(37, 99, 235, 0); }
  100% { box-shadow: 0 0 0 0 rgba(37, 99, 235, 0); }
}
.cta-pulse { animation: pulse-ring 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
.cta-pulse:hover { animation: none; transform: translateY(-2px); }
```

### Efeito Blur em Cards (hover nos irmaos)
```css
.cards-grid:hover .card { filter: blur(3px); opacity: 0.5; }
.cards-grid:hover .card:hover { filter: blur(0); opacity: 1; transform: scale(1.02); }
```

### Glass Morphism
```css
.glass-card {
  background: rgba(255,255,255,0.07);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255,255,255,0.1);
}
```

### Badge com Gradiente Animado
```css
@keyframes popular-gradient {
  0% { background-position-x: 0%; }
  100% { background-position-x: -200%; }
}
.popular-badge {
  background: linear-gradient(90deg, #3b82f6, #8b5cf6, #3b82f6);
  background-size: 200% 100%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: popular-gradient 2s linear infinite;
}
```

## Padroes JavaScript

### IntersectionObserver (fade-up + pricing stagger)
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) entry.target.classList.add('visible');
  });
}, { threshold: 0.1, rootMargin: '0px 0px -40px 0px' });
document.querySelectorAll('.fade-up, .pricing-card-animate').forEach(el => observer.observe(el));
```

### Smooth Scroll Customizado (600ms ideal)
```javascript
function smoothScrollTo(targetY, duration) {
  const startY = window.scrollY;
  const diff = targetY - startY;
  let start = null;
  function step(timestamp) {
    if (!start) start = timestamp;
    const progress = Math.min((timestamp - start) / duration, 1);
    const ease = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2;
    window.scrollTo(0, startY + diff * ease);
    if (progress < 1) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}
```
**600ms e o sweet spot.** 1800ms+ e muito lento. Instantaneo e brusco demais.

### Counter Animado
```javascript
const counterObs = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (!entry.isIntersecting) return;
    const el = entry.target;
    const target = +el.dataset.target;
    let current = 0;
    const step = target / (2000 / 16); // 2s duration
    function update() {
      current += step;
      if (current >= target) { el.textContent = target + '+'; counterObs.unobserve(el); return; }
      el.textContent = Math.floor(current) + '+';
      requestAnimationFrame(update);
    }
    update();
  });
}, { threshold: 0.5 });
```

## Pricing - Padrao que Converte

### 3 Cards: Mensal | Anual (destaque) | Semestral
- **Anual** no centro com:
  - `scale(1.05)` mobile, `scale(1.10)` desktop
  - Background escuro (dark-900) com texto branco
  - Badge "Popular" com gradiente animado
  - Ribbon de desconto (posicao absoluta, -top-4)
  - Preco riscado (text-decoration: line-through)
  - Economia em reais destacada
  - Botao CTA pulsante (cta-pulse)
  - Shadow maior (shadow-2xl)

- **Mensal e Semestral**: Background branco, shadow normal, CTA escuro

### Feature Stagger Animation
```css
.pricing-feature { opacity: 0; transform: translateX(-10px); transition: 0.4s ease; }
.pricing-card-animate.visible .pricing-feature { opacity: 1; transform: translateX(0); }
.pricing-card-animate.visible .pricing-feature:nth-child(1) { transition-delay: 0.1s; }
/* ... ate nth-child(5) a 0.5s */
```

### Preco Riscado - CUIDADO no Mobile
Preco riscado em `text-4xl` quebra layout no mobile. Solucao: colocar em linha separada com `text-lg`.

## Videos - O que DEU ERRADO

### NAO FACA:
| Tentativa | Resultado |
|-----------|-----------|
| `preload="metadata"` | Video para de dar autoplay |
| `preload="none"` | Video nao carrega |
| `loading="lazy"` em video | Autoplay falha |
| `data-src` com lazy JS | Autoplay falha |

### FACA:
```html
<video autoplay loop muted playsinline>
  <source src="url.mp4" type="video/mp4" />
</video>
```
**Simples assim. Sem preload, sem lazy. Autoplay so funciona com `muted`.**

### Video como Background
```html
<section class="relative overflow-hidden" style="min-height: 600px;">
  <video class="absolute inset-0 w-full h-full object-cover" autoplay muted loop playsinline>
    <source src="url.mp4" type="video/mp4">
  </video>
  <div class="absolute inset-0 bg-dark-900/80 backdrop-blur-[2px]"></div>
  <div class="relative z-10"><!-- conteudo --></div>
</section>
```

### COMPRIMA VIDEOS ANTES DE SUBIR
| Tamanho | Impacto |
|---------|---------|
| > 20MB | PageSpeed vai chorar, LCP > 5s |
| 5-20MB | Aceitavel com boa conexao |
| < 5MB | Ideal |

**Ferramentas**: HandBrake, CloudConvert, FFmpeg
**Target**: 720p, H.264, 1-2Mbps bitrate, max 30s loop

## Imagens - Licoes Aprendidas

### Redimensione ANTES de subir
| Uso | Tamanho Maximo |
|-----|----------------|
| Avatar/icone (36x36) | 100x100px, <50KB |
| Card/thumbnail | 400x400px, <200KB |
| Hero/banner | 1200px largura, <500KB |

### Formatos
- **WebP** ou **AVIF** > PNG/JPG (80%+ menor)
- **JPG** para fotos, **PNG** so se precisar transparencia
- Use [Squoosh.app](https://squoosh.app) ou TinyPNG

### Atributos Performance
```html
<!-- Acima do fold -->
<img src="url" width="200" height="200" alt="desc" />

<!-- Abaixo do fold -->
<img src="url" width="200" height="200" alt="desc" loading="lazy" decoding="async" />
```

### mix-blend para Logo
- `mix-blend-multiply` em fundo claro
- `mix-blend-screen` em fundo escuro
- Resultado inconsistente - prefira logo com fundo transparente (PNG/WebP)

## Performance / PageSpeed

### Metricas-Alvo
| Metrica | Meta | Armadilha |
|---------|------|-----------|
| LCP | < 2.5s | Videos grandes, imagens nao otimizadas |
| TBT | < 200ms | Tailwind CDN adiciona ~1.3s |
| CLS | < 0.1 | Imagens sem width/height |
| FCP | < 1.8s | Fonts bloqueando render |

### Checklist de Performance
- [ ] Videos comprimidos (< 5MB cada)
- [ ] Imagens redimensionadas para tamanho de exibicao
- [ ] Formatos modernos (WebP/AVIF)
- [ ] `loading="lazy"` em imagens abaixo do fold
- [ ] `decoding="async"` em imagens nao-criticas
- [ ] `width` e `height` em TODAS as imagens
- [ ] `<link rel="preconnect">` para CDNs de assets
- [ ] Passive listeners em scroll events
- [ ] NAO usar preload em videos com autoplay

### Tailwind CDN vs Producao
- CDN: rapido para prototipar, mas TBT alto (~1.3s no desktop)
- Producao: considere gerar CSS purgado via CLI do Tailwind (< 20KB)

## Equipe / Team Section

### Desktop: Cursor Card Flutuante
- Card segue o mouse com easing (0.12 factor)
- Rows escurecem (opacity 0.3) exceto hover
- **MOVER card para `document.body`** via JS (senao ghl-wrapper quebra fixed)

### Mobile: Accordion
- Click expande com max-height animation
- Icon swap (plus/minus)
- Um item aberto por vez

## Erros Comuns e Solucoes

| Problema | Causa | Solucao |
|----------|-------|---------|
| Bordas laterais no GHL | Container pai limitado | `.ghl-wrapper` com `100vw + translateX(-50%)` |
| Elemento fixed nao funciona | `transform` no pai (ghl-wrapper) | Mover para `document.body` via JS |
| Video nao da autoplay | `preload` atributo ou falta `muted` | Remover preload, manter `muted` |
| Preco riscado quebra mobile | Texto grande inline | Separar em linha propria, `text-lg` |
| Score PageSpeed baixo | Videos/imagens enormes | Comprimir videos, redimensionar imagens |
| Logo com fundo branco | PNG sem transparencia | `mix-blend-multiply` ou usar PNG transparente |
| Scroll muito lento/rapido | Duracao errada | 600ms com easeInOutCubic |
| Fotos enormes para avatar | 2048x2048 exibida em 36x36 | Redimensionar para 100x100 antes de subir |

## Workflow Recomendado

```
1. Estruturar HTML com todas as secoes (ordem PAS)
2. Adicionar .ghl-wrapper DESDE O INICIO
3. Estilizar com Tailwind + CSS customizado
4. Adicionar animacoes (fade-up, stagger, pulse)
5. Comprimir TODOS os assets ANTES de referenciar
6. Testar no navegador local
7. Testar no GHL (verificar full-width, fixed, videos)
8. Rodar PageSpeed e otimizar
9. Publicar
```

## Links de Pagamento (Kiwify)
Integrar via `<a href="https://pay.kiwify.com.br/SEU_ID" target="_blank">` nos botoes de pricing.

## Quick Reference - Tailwind Config
```javascript
tailwind.config = {
  theme: {
    extend: {
      fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] },
      colors: {
        brand: { 50: '#eff6ff', 500: '#3b82f6', 600: '#2563eb', 700: '#1d4ed8', 800: '#1e40af', 900: '#1e3a8a' },
        dark: { 900: '#0f172a', 800: '#1e293b', 700: '#334155' }
      }
    }
  }
}
```
