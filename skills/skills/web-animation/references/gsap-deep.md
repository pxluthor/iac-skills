# GSAP Deep Reference
## Para Landing Pages & Single Pages HTML

---

## Easing Completo

### Curvas built-in

| Nome | Comportamento | Melhor uso |
|---|---|---|
| `power1.out` | Desacelera suave | Texto, botões |
| `power2.out` | Desacelera médio | Cards, modais |
| `power3.out` | Desacelera forte | Heróis, títulos |
| `power4.out` | Desacelera muito forte | Elementos grandes |
| `back.out(1.7)` | Ultrapassa e volta | Badges, ícones |
| `elastic.out(1, 0.3)` | Elástico | Notificações |
| `bounce.out` | Quica | Elementos lúdicos |
| `expo.out` | Exponencial | Transições rápidas |
| `sine.inOut` | Senoidal suave | Loops contínuos |
| `none` / `linear` | Constante | Scrub, parallax |

### Custom Ease (CSS cubic-bezier → GSAP)

```js
// Importar CustomEase se necessário
// gsap.registerPlugin(CustomEase);
// CustomEase.create('brandEase', 'M0,0 C0.16,1 0.3,1 1,1');

// Alternativa: mapear tokens CSS → GSAP strings
const easingMap = {
  out:     'power3.out',
  inOut:   'power2.inOut',
  spring:  'back.out(1.4)',
  linear:  'none'
};
```

---

## Position Parameter (Timeline)

Controla onde cada tween começa dentro da timeline:

```js
const tl = gsap.timeline();

tl.to('.a', { x: 100 }, 0);           // absoluto: em 0s
tl.to('.b', { y: 50  }, 1.2);         // absoluto: em 1.2s
tl.to('.c', { opacity: 0 }, '+=0.5'); // relativo: 0.5s após o fim anterior
tl.to('.d', { scale: 1.1 }, '-=0.2'); // relativo: 0.2s antes do fim anterior
tl.to('.e', { x: 0 }, '<');           // simultâneo: mesmo início que anterior
tl.to('.f', { y: 0 }, '<0.3');        // 0.3s após o início do anterior
tl.to('.g', { opacity: 1 }, '>');     // após o anterior terminar
```

---

## ScrollTrigger — Configurações Completas

```js
gsap.to('.element', {
  x: 200,
  scrollTrigger: {
    trigger:    '.element',     // elemento que dispara
    start:      'top 80%',      // [trigger-edge] [viewport-edge]
    end:        'bottom 20%',   // quando termina
    scrub:      true,           // sincroniza com scroll (true ou número em segundos de lag)
    pin:        true,           // fixa o elemento durante o scroll
    markers:    false,          // true para debug visual
    once:       true,           // dispara só uma vez
    toggleClass: 'is-active',   // adiciona/remove classe
    onEnter:    () => {},       // callbacks
    onLeave:    () => {},
    onEnterBack: () => {},
    onLeaveBack: () => {},
  }
});
```

### Padrões de start/end mais comuns

| start | Significado |
|---|---|
| `'top 85%'` | Elemento entra quando seu topo atinge 85% do viewport |
| `'top center'` | Elemento entra quando seu topo está no centro |
| `'center center'` | Elemento fica no centro |
| `'top top'` | Elemento toca o topo do viewport |
| `'bottom bottom'` | Elemento sai pela base |

---

## gsap.matchMedia() — Responsivo

```js
const mm = gsap.matchMedia();

mm.add('(min-width: 768px)', () => {
  // animações só em desktop
  gsap.from('.sidebar', { x: -80, autoAlpha: 0, duration: 0.6 });
});

mm.add('(max-width: 767px)', () => {
  // animações só em mobile
  gsap.from('.sidebar', { y: 40, autoAlpha: 0, duration: 0.5 });
});

mm.add('(prefers-reduced-motion: reduce)', () => {
  gsap.globalTimeline.timeScale(0);  // para tudo
});
```

---

## Stagger Avançado

```js
gsap.from('.item', {
  autoAlpha: 0, y: 30,
  stagger: {
    each:   0.08,             // intervalo entre cada item
    from:   'center',         // 'start' | 'end' | 'center' | 'random' | index
    grid:   'auto',           // para grids 2D
    axis:   'x',              // 'x' | 'y' — para grids
    ease:   'power1.inOut',   // easing do próprio stagger
    amount: 0.5               // duração total do stagger (alternativa a each)
  }
});
```

---

## quickTo — Mouse tracking / Frequent Updates

```js
// Ideal para cursor customizado ou elementos que seguem o mouse
const xTo = gsap.quickTo('#cursor', 'x', { duration: 0.4, ease: 'power3' });
const yTo = gsap.quickTo('#cursor', 'y', { duration: 0.4, ease: 'power3' });

window.addEventListener('mousemove', e => {
  xTo(e.clientX);
  yTo(e.clientY);
});
```

---

## Flip — Animações de Layout

```js
// gsap.registerPlugin(Flip);

// 1. Captura estado atual
const state = Flip.getState('.card');

// 2. Muda o layout (DOM, classes, grid)
container.classList.toggle('expanded');

// 3. Anima a diferença
Flip.from(state, { duration: 0.6, ease: 'power2.inOut', stagger: 0.05 });
```

---

## Utilitários Avançados

```js
// Executar após todas as imagens carregarem
window.addEventListener('load', () => { /* animar */ });

// Delay de módulo — útil em SPAs
gsap.delayedCall(2, () => { /* roda após 2s */ });

// Matar todas as animações de um elemento
gsap.killTweensOf('.element');

// Progresso condicional
const tl = gsap.timeline({ paused: true });
// ... adicionar tweens ...
document.querySelector('.btn').addEventListener('click', () => tl.play());

// Exportar duração total (útil para sincronizar com vídeo/áudio)
console.log('Duração total:', tl.totalDuration(), 's');
```

---

## Propriedades SVG

```js
gsap.to('path', {
  drawSVG: '100%',        // plugin DrawSVG — anima traçado
  morphSVG: '#target',   // plugin MorphSVG — transforma forma
  svgOrigin: '200 300',  // origem no espaço do SVG (não misturar com transformOrigin)
  strokeDashoffset: 0,   // alternativa CSS pura ao drawSVG
});
```

---

## clearProps — Limpeza após animação

```js
gsap.to('.element', {
  x: 100, opacity: 0.5,
  onComplete() {
    gsap.set(this.targets(), { clearProps: 'all' }); // remove estilos inline
  }
});
// Ou diretamente na tween:
gsap.to('.element', { x: 100, clearProps: 'x' }); // limpa só x ao terminar
```
