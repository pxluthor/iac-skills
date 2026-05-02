# Anime.js Deep Reference
## Para Landing Pages HTML — Bundle Mínimo

---

## Easing Completo

```js
// Funções built-in
'linear'
'easeInQuad'   | 'easeOutQuad'   | 'easeInOutQuad'
'easeInCubic'  | 'easeOutCubic'  | 'easeInOutCubic'
'easeInQuart'  | 'easeOutQuart'  | 'easeInOutQuart'
'easeInQuint'  | 'easeOutQuint'  | 'easeInOutQuint'
'easeInSine'   | 'easeOutSine'   | 'easeInOutSine'
'easeInExpo'   | 'easeOutExpo'   | 'easeInOutExpo'
'easeInCirc'   | 'easeOutCirc'   | 'easeInOutCirc'
'easeInElastic'| 'easeOutElastic'| 'easeInOutElastic'
'easeInBack'   | 'easeOutBack'   | 'easeInOutBack'
'easeInBounce' | 'easeOutBounce' | 'easeInOutBounce'
'spring(mass, stiffness, damping, velocity)'  // ex: 'spring(1, 80, 10, 0)'
'cubicBezier(x1, y1, x2, y2)'
'steps(n)'
```

---

## IntersectionObserver — Scroll Reveal Completo

```js
class ScrollReveal {
  constructor(options = {}) {
    this.defaults = {
      threshold: 0.15,
      translateY: [40, 0],
      opacity:    [0, 1],
      duration:   600,
      easing:     'easeOutExpo',
      delay:      0,
      stagger:    80,          // ms entre itens
    };
    this.cfg = { ...this.defaults, ...options };
    this.observer = new IntersectionObserver(
      this._handler.bind(this),
      { threshold: this.cfg.threshold }
    );
  }

  watch(selector) {
    const els = document.querySelectorAll(selector);
    els.forEach(el => {
      el.style.opacity = '0';
      this.observer.observe(el);
    });
    return this;
  }

  _handler(entries) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      anime({
        targets:    entry.target,
        translateY: this.cfg.translateY,
        opacity:    this.cfg.opacity,
        duration:   this.cfg.duration,
        easing:     this.cfg.easing,
        delay:      this.cfg.delay,
      });
      this.observer.unobserve(entry.target);
    });
  }
}

// Uso:
const reveal = new ScrollReveal();
reveal
  .watch('h2')
  .watch('.product-card')
  .watch('.trust-item');
```

---

## Stagger com grupos

```js
// Animar todos os cards juntos com stagger
function revealGroup(selector, staggerMs = 80) {
  const els = document.querySelectorAll(selector);
  if (!els.length) return;

  const observer = new IntersectionObserver(entries => {
    if (!entries.some(e => e.isIntersecting)) return;
    anime({
      targets:    els,
      translateY: [40, 0],
      opacity:    [0, 1],
      duration:   600,
      easing:     'easeOutExpo',
      delay:      anime.stagger(staggerMs)
    });
    observer.disconnect();
  }, { threshold: 0.1 });

  // Observar o container pai
  observer.observe(els[0].closest('section') || els[0]);
  els.forEach(el => el.style.opacity = '0');
}
```

---

## Anime.js v4 (ESM / API moderna)

```js
// Se usar módulo ES:
import { animate, createTimeline, stagger, utils } from
  'https://cdn.jsdelivr.net/npm/animejs/+esm';

// animate() substituiu anime() na v4
const anim = animate('.hero h1', {
  translateY: [60, 0],
  opacity:    [0, 1],
  duration:   800,
  easing:     'easeOutExpo',
});

// Timeline
const tl = createTimeline({ defaults: { easing: 'easeOutExpo' } });
tl
  .add('.eyebrow',  { translateY: [20, 0], opacity: [0,1], duration: 400 })
  .add('.hero h1',  { translateY: [60, 0], opacity: [0,1], duration: 700 }, 150)
  .add('.hero p',   { translateY: [24, 0], opacity: [0,1], duration: 500 }, 350);
```

---

## Contador numérico

```js
function animeCounter(el) {
  const raw = el.textContent.trim();
  const num = parseFloat(raw);
  if (isNaN(num)) return;
  const suffix = raw.replace(/[\d.]/g, '');

  const obj = { count: 0 };

  const observer = new IntersectionObserver(entries => {
    if (!entries[0].isIntersecting) return;
    anime({
      targets:  obj,
      count:    num,
      round:    Number.isInteger(num) ? 1 : 10,
      duration: 1600,
      easing:   'easeOutExpo',
      update:   () => { el.textContent = obj.count + suffix; }
    });
    observer.disconnect();
  }, { threshold: 0.8 });

  observer.observe(el);
  el.textContent = '0' + suffix;
}

document.querySelectorAll('.stat strong').forEach(animeCounter);
```

---

## Propriedades SVG

```js
// Traçado animado (stroke-dashoffset)
anime({
  targets: 'path',
  strokeDashoffset: [anime.setDashoffset, 0],
  duration: 1500,
  easing: 'easeInOutCubic'
});

// Morphing de forma
anime({
  targets: '#shape',
  d: [
    { value: 'M0,0 L100,0 L100,100 Z' },
    { value: 'M50,0 L100,100 L0,100 Z' }
  ],
  duration: 1000,
  loop: true,
  direction: 'alternate'
});
```
