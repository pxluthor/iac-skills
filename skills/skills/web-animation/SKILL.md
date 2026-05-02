---
name: web-animation
description: >
  Animation system for production HTML/CSS/JS landing pages and single-page sites.
  Use this skill whenever the user wants to add animations, motion, scroll effects,
  entrance effects, counters, parallax, microinteractions, or any kind of movement
  to a plain HTML file, sales page, landing page, or static site — even if they
  don't explicitly say "animation". Triggers include: "make it more dynamic",
  "add motion", "improve the hero", "make the page feel alive", "add scroll effects",
  "animate the cards", "counter animation", "parallax", or any request to improve
  a static HTML page visually. Covers GSAP (primary) and Anime.js (lightweight
  alternative), with a full design-token system the client can override.
---

# Web Animation Skill
## For Plain HTML / JS Landing Pages & Sales Pages

> **Scope:** This skill is for standard HTML files — NOT HyperFrames.
> There is no `window.__timelines` registry, no `data-composition-id`, no `npx hyperframes` tooling.
> Animations run freely on the browser clock.

---

## Quick Decision Tree

```
Precisa de animação?
│
├── Página pequena / animações simples?  →  Anime.js  (18 KB, zero deps)
│
└── Scroll effects / timelines complexas / performance crítica?  →  GSAP + ScrollTrigger
```

> **Regra geral:** Use GSAP por padrão. Anime.js apenas quando o cliente exige bundle mínimo.

---

## 1. Design Token System

Toda página animada deve expor um bloco `:root` com tokens padronizados.
O cliente pode sobrescrever qualquer token sem tocar no código de animação.

```css
:root {
  /* ── Durations ─────────────────────────────── */
  --dur-xs:   0.18s;   /* micro: hover, ripple      */
  --dur-sm:   0.35s;   /* fast:  appear, fade       */
  --dur-md:   0.60s;   /* base:  most entrances     */
  --dur-lg:   0.90s;   /* slow:  hero, big reveals  */
  --dur-xl:   1.40s;   /* dramatic: full-page       */

  /* ── Easing (CSS) ──────────────────────────── */
  --ease-out:      cubic-bezier(0.16, 1, 0.3, 1);   /* snappy deceleration  */
  --ease-in-out:   cubic-bezier(0.87, 0, 0.13, 1);  /* smooth both ends     */
  --ease-spring:   cubic-bezier(0.34, 1.56, 0.64, 1); /* slight overshoot  */
  --ease-linear:   linear;

  /* ── Distances ─────────────────────────────── */
  --slide-sm:  24px;
  --slide-md:  48px;
  --slide-lg:  80px;

  /* ── Stagger (JS — read via getComputedStyle) ─ */
  --stagger-sm:  0.06;   /* seconds between each item */
  --stagger-md:  0.10;
  --stagger-lg:  0.16;

  /* ── Reduced motion (always respect this) ───── */
  --motion-ok: 1;        /* set to 0 via JS if prefers-reduced-motion */
}

@media (prefers-reduced-motion: reduce) {
  :root { --motion-ok: 0; }
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Como o cliente customiza:**
```css
/* Na folha de estilos do projeto, basta sobrescrever: */
:root {
  --dur-md: 0.45s;           /* mais rápido */
  --ease-out: ease-out;      /* curva simples */
  --slide-md: 32px;          /* menor deslocamento */
}
```

---

## 2. GSAP — Padrão Principal

### 2.1 Setup (CDN)

```html
<!-- Sempre no <head> ou antes do </body> -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js"></script>
<!-- ScrollTrigger: só quando houver scroll animations -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js"></script>
```

### 2.2 Inicialização padrão

```js
// Sempre no início do bloco de animação
gsap.registerPlugin(ScrollTrigger);

// Ler tokens de duração do CSS
const root    = document.documentElement;
const durMd   = parseFloat(getComputedStyle(root).getPropertyValue('--dur-md'))   || 0.6;
const durLg   = parseFloat(getComputedStyle(root).getPropertyValue('--dur-lg'))   || 0.9;
const stagMd  = parseFloat(getComputedStyle(root).getPropertyValue('--stagger-md')) || 0.1;

// Respeitar preferência de movimento reduzido
const motionOk = getComputedStyle(root).getPropertyValue('--motion-ok').trim() !== '0';
if (!motionOk) { /* skip ou reduzir animações */ }
```

### 2.3 Padrões de entrada (Hero)

```js
// Sequência de entrada da hero — roda uma vez ao carregar
const heroTl = gsap.timeline({ defaults: { ease: 'power3.out' } });

heroTl
  .from('.eyebrow',       { y: 20, autoAlpha: 0, duration: durMd })
  .from('h1',             { y: var_slide_lg, autoAlpha: 0, duration: durLg }, '-=0.3')
  .from('.hero-copy',     { y: 24, autoAlpha: 0, duration: durMd }, '-=0.4')
  .from('.hero-actions',  { y: 16, autoAlpha: 0, duration: durMd }, '-=0.35')
  .from('.hero-stats .stat', {
      y: 20, autoAlpha: 0,
      duration: durMd,
      stagger: stagMd
  }, '-=0.2');
```

> **Nota:** Nunca anime `left`, `top`, `width`, `height` — use sempre `x`, `y`, `scale`.
> Use `autoAlpha` em vez de `opacity` (também controla `visibility`).

### 2.4 Scroll Reveals (padrão reutilizável)

```js
// Função utilitária — aplicar a qualquer seção
function revealOnScroll(selector, options = {}) {
  const defaults = {
    y: 40, autoAlpha: 0, duration: durMd, ease: 'power2.out',
    stagger: stagMd, start: 'top 85%'
  };
  const cfg = { ...defaults, ...options };

  gsap.from(selector, {
    y:         cfg.y,
    autoAlpha: cfg.autoAlpha,
    duration:  cfg.duration,
    ease:      cfg.ease,
    stagger:   cfg.stagger,
    scrollTrigger: {
      trigger:  typeof selector === 'string' ? selector : selector[0],
      start:    cfg.start,
      once:     true,            // dispara só uma vez — melhor para páginas de vendas
    }
  });
}

// Uso:
revealOnScroll('.product-card');
revealOnScroll('.trust-item',  { stagger: 0.08 });
revealOnScroll('h2',           { y: 30, stagger: 0 });
```

### 2.5 Contador numérico animado

```js
// Para stats como "12x", "2 anos", "24h"
function animateCounter(el) {
  const raw    = el.textContent.trim();
  const num    = parseFloat(raw);
  const suffix = raw.replace(/[\d.]/g, '');   // captura "x", "h", " anos"
  if (isNaN(num)) return;

  gsap.from({ val: 0 }, {
    val: num,
    duration: 1.6,
    ease: 'power2.out',
    snap: { val: Number.isInteger(num) ? 1 : 0.1 },
    scrollTrigger: { trigger: el, start: 'top 88%', once: true },
    onUpdate() { el.textContent = this.targets()[0].val + suffix; }
  });
}

document.querySelectorAll('.stat strong').forEach(animateCounter);
```

### 2.6 Navbar — hide/show ao scroll

```js
let lastY = 0;
ScrollTrigger.create({
  start: 'top -80px',
  onUpdate: self => {
    const dir = self.direction;   // 1 = down, -1 = up
    gsap.to('.topbar', {
      yPercent: dir === 1 ? -100 : 0,
      duration: 0.35,
      ease: 'power2.inOut'
    });
    lastY = self.scroll();
  }
});
```

### 2.7 Parallax simples no Hero

```js
gsap.to('.hero img', {
  yPercent: 30,
  ease: 'none',
  scrollTrigger: {
    trigger: '.hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true
  }
});
```

### 2.8 Microinterações em botões/cards

```js
// Pulse no CTA principal — chama atenção sem ser intrusivo
gsap.to('.button:not(.secondary)', {
  scale: 1.025,
  duration: 0.9,
  ease: 'sine.inOut',
  yoyo: true,
  repeat: -1,
  delay: 3      // começa depois das animações de entrada
});

// Hover em cards — via CSS é suficiente, mas GSAP permite mais controle:
document.querySelectorAll('.product-card').forEach(card => {
  card.addEventListener('mouseenter', () =>
    gsap.to(card, { y: -6, boxShadow: '0 24px 48px rgba(0,0,0,.18)', duration: 0.25, ease: 'power2.out' })
  );
  card.addEventListener('mouseleave', () =>
    gsap.to(card, { y: 0,  boxShadow: '0 0px 0px rgba(0,0,0,0)',     duration: 0.35, ease: 'power2.inOut' })
  );
});
```

---

## 3. Anime.js — Alternativa Leve

Use quando: bundle mínimo, sem ScrollTrigger, animações simples de entrada.

### 3.1 Setup

```html
<script src="https://cdn.jsdelivr.net/npm/animejs@4.0.2/lib/anime.iife.min.js"></script>
```

### 3.2 Entrance com IntersectionObserver

Anime.js não tem ScrollTrigger — use a API nativa do browser:

```js
function animeReveal(selector, animProps = {}) {
  const elements = document.querySelectorAll(selector);
  if (!elements.length) return;

  const defaults = {
    translateY: [40, 0],
    opacity:    [0, 1],
    duration:   600,
    easing:     'easeOutExpo',
    delay:      anime.stagger(80)
  };

  const observer = new IntersectionObserver(entries => {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      anime({ targets: entry.target, ...defaults, ...animProps });
      observer.unobserve(entry.target);
    });
  }, { threshold: 0.15 });

  elements.forEach(el => {
    el.style.opacity = '0';
    observer.observe(el);
  });
}

animeReveal('.product-card');
animeReveal('.trust-item', { delay: anime.stagger(60) });
```

### 3.3 Timeline de entrada (Hero)

```js
const heroTl = anime.timeline({ easing: 'easeOutExpo', autoplay: true });

heroTl
  .add({ targets: '.eyebrow',      translateY: [20, 0], opacity: [0, 1], duration: 400 })
  .add({ targets: 'h1',            translateY: [60, 0], opacity: [0, 1], duration: 700 }, 150)
  .add({ targets: '.hero-copy',    translateY: [24, 0], opacity: [0, 1], duration: 500 }, 300)
  .add({ targets: '.hero-actions', translateY: [16, 0], opacity: [0, 1], duration: 400 }, 450)
  .add({
    targets:  '.stat',
    translateY: [20, 0], opacity: [0, 1],
    duration: 400,
    delay:    anime.stagger(80)
  }, 550);
```

---

## 4. CSS-Only Animations (sem JS)

Para hover states, loaders e elementos decorativos — não requerem GSAP/Anime.js:

```css
/* Fade-in de entrada com classe utilitária */
.animate-in {
  animation: fadeSlideUp var(--dur-md) var(--ease-out) both;
}

@keyframes fadeSlideUp {
  from { opacity: 0; transform: translateY(var(--slide-md)); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Pulse para CTA (fallback sem JS) */
.cta-pulse {
  animation: pulse 2s var(--ease-in-out) infinite;
}

@keyframes pulse {
  0%, 100% { box-shadow: 0 0 0 0   rgba(182,136,71, 0.5); }
  50%       { box-shadow: 0 0 0 12px rgba(182,136,71, 0);   }
}

/* Shimmer em skeleton loaders */
.skeleton {
  background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
  background-size: 200% 100%;
  animation: shimmer 1.4s infinite;
}

@keyframes shimmer {
  from { background-position: 200% 0; }
  to   { background-position: -200% 0; }
}
```

---

## 5. Template Completo (Boilerplate)

Estrutura de script para colar em qualquer landing page:

```html
<!-- CDN -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js"></script>

<script>
(function () {
  'use strict';

  // ── 0. Setup ────────────────────────────────────────────────────────────────
  gsap.registerPlugin(ScrollTrigger);
  const root   = document.documentElement;
  const durMd  = parseFloat(getComputedStyle(root).getPropertyValue('--dur-md'))    || 0.6;
  const durLg  = parseFloat(getComputedStyle(root).getPropertyValue('--dur-lg'))    || 0.9;
  const stagMd = parseFloat(getComputedStyle(root).getPropertyValue('--stagger-md'))|| 0.1;
  const motionOk = getComputedStyle(root).getPropertyValue('--motion-ok').trim() !== '0';

  if (!motionOk) return; // respeita prefers-reduced-motion

  // ── 1. Hero entrance ────────────────────────────────────────────────────────
  const heroTl = gsap.timeline({ defaults: { ease: 'power3.out' } });
  heroTl
    .from('.eyebrow',          { y: 20, autoAlpha: 0, duration: durMd })
    .from('h1',                { y: 60, autoAlpha: 0, duration: durLg }, '-=0.3')
    .from('.hero-copy',        { y: 24, autoAlpha: 0, duration: durMd }, '-=0.4')
    .from('.hero-actions',     { y: 16, autoAlpha: 0, duration: durMd }, '-=0.35')
    .from('.hero-stats .stat', { y: 20, autoAlpha: 0, duration: durMd, stagger: stagMd }, '-=0.2');

  // ── 2. Parallax ─────────────────────────────────────────────────────────────
  if (document.querySelector('.hero img')) {
    gsap.to('.hero img', {
      yPercent: 30, ease: 'none',
      scrollTrigger: { trigger: '.hero', start: 'top top', end: 'bottom top', scrub: true }
    });
  }

  // ── 3. Scroll reveals ───────────────────────────────────────────────────────
  const reveals = [
    ['h2',             { y: 30, stagger: 0   }],
    ['.product-card',  { y: 50, stagger: stagMd }],
    ['.trust-item',    { y: 40, stagger: 0.08 }],
    ['.deal-box',      { y: 40, stagger: 0   }],
  ];

  reveals.forEach(([sel, opts]) => {
    if (!document.querySelector(sel)) return;
    gsap.from(sel, {
      y: opts.y, autoAlpha: 0, duration: durMd, ease: 'power2.out',
      stagger:  opts.stagger,
      scrollTrigger: { trigger: sel, start: 'top 85%', once: true }
    });
  });

  // ── 4. Counters ─────────────────────────────────────────────────────────────
  document.querySelectorAll('.stat strong').forEach(el => {
    const raw = el.textContent.trim();
    const num = parseFloat(raw);
    if (isNaN(num)) return;
    const sfx = raw.replace(/[\d.]/g, '');
    gsap.from({ val: 0 }, {
      val: num, duration: 1.6, ease: 'power2.out',
      snap: { val: Number.isInteger(num) ? 1 : 0.1 },
      scrollTrigger: { trigger: el, start: 'top 88%', once: true },
      onUpdate() { el.textContent = this.targets()[0].val + sfx; }
    });
  });

  // ── 5. Navbar scroll ────────────────────────────────────────────────────────
  if (document.querySelector('.topbar')) {
    ScrollTrigger.create({
      start: 'top -80px',
      onUpdate: self => {
        gsap.to('.topbar', {
          yPercent: self.direction === 1 ? -100 : 0,
          duration: 0.35, ease: 'power2.inOut'
        });
      }
    });
  }

  // ── 6. CTA pulse ────────────────────────────────────────────────────────────
  gsap.to('.button:not(.secondary)', {
    scale: 1.025, duration: 0.9, ease: 'sine.inOut',
    yoyo: true, repeat: -1, delay: 3
  });

  // ── 7. Card hover ───────────────────────────────────────────────────────────
  document.querySelectorAll('.product-card').forEach(card => {
    card.addEventListener('mouseenter', () =>
      gsap.to(card, { y: -6, duration: 0.25, ease: 'power2.out' })
    );
    card.addEventListener('mouseleave', () =>
      gsap.to(card, { y: 0,  duration: 0.35, ease: 'power2.inOut' })
    );
  });

})();
</script>
```

---

## 6. Performance & Boas Práticas

| Faça | Evite |
|---|---|
| Anime `x`, `y`, `scale`, `rotation`, `opacity` | Anime `width`, `height`, `top`, `left`, `margin` |
| Use `autoAlpha` (GSAP) | Use `opacity` sem controlar `visibility` |
| `will-change: transform` só em elementos que animam | `will-change` em tudo |
| `once: true` no ScrollTrigger de páginas de vendas | Re-trigger a cada scroll |
| Leia tokens do CSS com `getComputedStyle` | Hardcode durações no JS |
| Sempre cheque `prefers-reduced-motion` | Ignorar acessibilidade |
| Agrupe tweens em `gsap.timeline()` | Encadeie com `delay` manual |
| Limpe com `.kill()` se remover elementos dinamicamente | Deixar tweens órfãos |

---

## 7. Referências

- Detalhes completos de easing, timelines, posição e plugins: **[references/gsap-deep.md](references/gsap-deep.md)**
- Padrões avançados de Anime.js com IntersectionObserver: **[references/animejs-deep.md](references/animejs-deep.md)**
- Galeria de efeitos prontos (typewriter, ripple, magneto, marquee): **[references/effects.md](references/effects.md)**

Leia os arquivos de referência apenas quando precisar de padrões avançados além do coberto neste SKILL.md.

---

## 8. Checklist antes de entregar

- [ ] `prefers-reduced-motion` respeitado
- [ ] Tokens de duração lidos do CSS (não hardcoded)
- [ ] Nenhuma animação em propriedades de layout (`width`, `top`, etc.)
- [ ] ScrollTrigger com `once: true` em páginas de vendas
- [ ] CDN carregado antes do script de animação
- [ ] Script envolto em IIFE `(function(){ 'use strict'; ... })()`
- [ ] Testado sem JS (página deve funcionar estaticamente)
