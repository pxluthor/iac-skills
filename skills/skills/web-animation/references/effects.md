# Effects Gallery
## Efeitos prontos para copiar em landing pages HTML

---

## 1. Typewriter

```js
function typewriter(el, options = {}) {
  const text    = el.textContent;
  const speed   = options.speed || 40;       // ms por caractere
  const cursor  = options.cursor !== false;

  el.textContent = '';
  if (cursor) el.style.borderRight = '2px solid currentColor';

  let i = 0;
  const interval = setInterval(() => {
    el.textContent += text[i++];
    if (i >= text.length) {
      clearInterval(interval);
      if (cursor) setTimeout(() => el.style.borderRight = 'none', 800);
    }
  }, speed);
}

// Uso:
typewriter(document.querySelector('.hero-eyebrow'), { speed: 55 });
```

---

## 2. Texto Reveal com máscara (clip-path)

```css
.text-reveal-wrap {
  overflow: hidden;
  display: inline-block;
}
```

```js
// GSAP
gsap.from('.text-reveal-wrap > *', {
  yPercent: 110,
  duration: 0.8,
  ease: 'power3.out',
  stagger: 0.08
});

// Preparação: envolver cada linha/palavra no markup
function wrapLines(selector) {
  document.querySelectorAll(selector).forEach(el => {
    const wrap = document.createElement('div');
    wrap.className = 'text-reveal-wrap';
    el.parentNode.insertBefore(wrap, el);
    wrap.appendChild(el);
  });
}
```

---

## 3. Marquee / Ticker infinito

```css
.marquee {
  overflow: hidden;
  white-space: nowrap;
}
.marquee-track {
  display: inline-flex;
  gap: 48px;
  animation: marquee-scroll 20s linear infinite;
}
@keyframes marquee-scroll {
  from { transform: translateX(0); }
  to   { transform: translateX(-50%); }
}
/* Pausar no hover */
.marquee:hover .marquee-track { animation-play-state: paused; }
```

```html
<div class="marquee">
  <div class="marquee-track">
    <!-- Duplicar conteúdo para loop contínuo -->
    <span>Item 1</span><span>Item 2</span><span>Item 3</span>
    <span>Item 1</span><span>Item 2</span><span>Item 3</span>
  </div>
</div>
```

---

## 4. Ripple em botões

```css
.btn-ripple { position: relative; overflow: hidden; }

.ripple-wave {
  position: absolute;
  border-radius: 50%;
  transform: scale(0);
  background: rgba(255,255,255,0.35);
  pointer-events: none;
  animation: ripple-expand 0.6s ease-out forwards;
}

@keyframes ripple-expand {
  to { transform: scale(4); opacity: 0; }
}
```

```js
document.querySelectorAll('.btn-ripple').forEach(btn => {
  btn.addEventListener('click', e => {
    const rect  = btn.getBoundingClientRect();
    const size  = Math.max(rect.width, rect.height);
    const wave  = document.createElement('span');
    wave.className = 'ripple-wave';
    wave.style.cssText = `
      width: ${size}px; height: ${size}px;
      left: ${e.clientX - rect.left - size/2}px;
      top:  ${e.clientY - rect.top  - size/2}px;
    `;
    btn.appendChild(wave);
    wave.addEventListener('animationend', () => wave.remove());
  });
});
```

---

## 5. Magneto (botão segue o cursor)

```js
document.querySelectorAll('.btn-magneto').forEach(btn => {
  btn.addEventListener('mousemove', e => {
    const rect   = btn.getBoundingClientRect();
    const cx     = rect.left + rect.width  / 2;
    const cy     = rect.top  + rect.height / 2;
    const dx     = (e.clientX - cx) * 0.35;
    const dy     = (e.clientY - cy) * 0.35;
    gsap.to(btn, { x: dx, y: dy, duration: 0.3, ease: 'power2.out' });
  });
  btn.addEventListener('mouseleave', () => {
    gsap.to(btn, { x: 0, y: 0, duration: 0.5, ease: 'elastic.out(1, 0.5)' });
  });
});
```

---

## 6. Number Flip (estilo placar)

```css
.flip-number {
  display: inline-block;
  overflow: hidden;
  height: 1em;
  line-height: 1;
}
.flip-number-inner {
  display: flex;
  flex-direction: column;
  transition: transform 0.5s cubic-bezier(0.16,1,0.3,1);
}
.flip-number span { display: block; height: 1em; }
```

```js
function flipTo(el, newVal) {
  const inner = el.querySelector('.flip-number-inner');
  const span  = document.createElement('span');
  span.textContent = newVal;
  inner.appendChild(span);
  const h = inner.querySelectorAll('span').length - 1;
  inner.style.transform = `translateY(-${h}em)`;
}
```

---

## 7. Parallax em múltiplas camadas

```js
// Velocidades diferentes para profundidade
const layers = [
  { selector: '.bg-far',    speed: 0.15 },
  { selector: '.bg-mid',    speed: 0.30 },
  { selector: '.bg-near',   speed: 0.50 },
  { selector: '.hero img',  speed: 0.25 },
];

layers.forEach(({ selector, speed }) => {
  if (!document.querySelector(selector)) return;
  gsap.to(selector, {
    yPercent: speed * 100,
    ease: 'none',
    scrollTrigger: {
      trigger: 'body',
      start:   'top top',
      end:     'bottom bottom',
      scrub:   true
    }
  });
});
```

---

## 8. Sticky section com pin e progresso

```js
// Fixa a seção enquanto o conteúdo interno anima
const pinnedTl = gsap.timeline({
  scrollTrigger: {
    trigger:  '.pinned-section',
    start:    'top top',
    end:      '+=200%',       // rola por 2x a altura da viewport
    pin:      true,
    scrub:    1,
    markers:  false
  }
});

pinnedTl
  .to('.step-1', { autoAlpha: 0, y: -40 })
  .from('.step-2', { autoAlpha: 0, y:  40 })
  .to('.step-2', { autoAlpha: 0, y: -40 })
  .from('.step-3', { autoAlpha: 0, y:  40 });
```

---

## 9. Toast / Notificação animada

```js
function showToast(message, duration = 3000) {
  const toast = document.createElement('div');
  toast.className = 'toast';
  toast.textContent = message;
  document.body.appendChild(toast);

  gsap.timeline()
    .from(toast, { y: 80, autoAlpha: 0, duration: 0.4, ease: 'back.out(1.7)' })
    .to(toast,   { autoAlpha: 0, duration: 0.3, ease: 'power2.in' }, `+=${duration/1000}`)
    .call(() => toast.remove());
}
```

```css
.toast {
  position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
  background: #151515; color: #fff; padding: 12px 24px;
  border-radius: 8px; font-size: 0.9rem; z-index: 9999;
  pointer-events: none;
}
```

---

## 10. Loading screen com saída animada

```js
function hideLoader() {
  const loader = document.querySelector('.loader');
  if (!loader) return;

  gsap.timeline()
    .to('.loader-logo', { autoAlpha: 0, y: -20, duration: 0.4 })
    .to(loader,         { autoAlpha: 0, duration: 0.5 })
    .call(() => {
      loader.remove();
      // Disparar animações da página aqui
      initPageAnimations();
    });
}

window.addEventListener('load', hideLoader);
```

```css
.loader {
  position: fixed; inset: 0; z-index: 999;
  display: grid; place-items: center;
  background: var(--ink, #151515);
}
```
