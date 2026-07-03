# Part 5: Frontend

---

## Chapter 35: HTML5 & CSS3

### Introduction

HTML5 and CSS3 form the bedrock of modern web development. HTML5 provides semantic structure, multimedia support, and rich APIs, while CSS3 delivers advanced styling with animations, transitions, and layouts. Together they create the user-facing layer of every web application.

### Why It Matters

Every frontend framework — React, Vue, Angular — compiles down to HTML and CSS. Understanding the fundamentals ensures you can debug layout issues, optimize rendering, build accessible interfaces, and work effectively regardless of the framework du jour.

> 🏛️ **Real World Analogy:** Think of HTML as the foundation and walls of a house (structure), CSS as the paint, wallpaper, and decor (presentation), and JavaScript as the electrical wiring and plumbing (behavior). A solid web application needs all three working together harmoniously.

### HTML5 Semantic Elements

| Element | Purpose |
|---------|---------|
| `<header>` | Introductory content, navigation, logo |
| `<nav>` | Navigation links block |
| `<main>` | Primary content (one per page) |
| `<section>` | Thematic grouping of content |
| `<article>` | Self-contained composition |
| `<aside>` | Indirectly related content |
| `<footer>` | Closing information, copyright |
| `<figure>` / `<figcaption>` | Illustrations with captions |
| `<mark>` | Highlighted text |
| `<time>` | Machine-readable date/time |

> ⚠️ **Warning:** A common mistake is using `<div>` for everything instead of semantic elements. This harms SEO and accessibility. Screen readers and search engines rely on landmarks like `<nav>`, `<main>`, and `<article>` to navigate content.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Semantic HTML Example</title>
</head>
<body>
  <header>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/blog">Blog</a></li>
      </ul>
    </nav>
  </header>
  <main>
    <article>
      <header>
        <h1>Article Title</h1>
        <time datetime="2026-07-04">July 4, 2026</time>
      </header>
      <section>
        <p>Main content here.</p>
      </section>
    </article>
    <aside>
      <h2>Related Links</h2>
      <ul><li><a href="#">Related post</a></li></ul>
    </aside>
  </main>
  <footer>
    <p>&copy; 2026 Full-Stack AI Book</p>
  </footer>
</body>
</html>
```

> ✅ **Best Practice:** Always include `alt` attributes on images for accessibility and SEO. Decorative images should have `alt=""` (empty) so screen readers skip them. Informative images need descriptive alt text. This is both a WCAG requirement and positively impacts image search rankings.

### HTML5 Multimedia

```html
<video controls width="640" poster="thumbnail.jpg">
  <source src="video.mp4" type="video/mp4" />
  <source src="video.webm" type="video/webm" />
  <p>Your browser does not support HTML5 video.</p>
</video>

<audio controls>
  <source src="audio.mp3" type="audio/mpeg" />
  <source src="audio.ogg" type="audio/ogg" />
</audio>
```

> 💡 **Pro Tip:** Provide multiple `<source>` formats (MP4 + WebM for video, MP3 + OGG for audio) to ensure browser compatibility. Always include fallback text for users with older browsers that don't support these elements.

### HTML5 APIs

- **Canvas** — Programmatic 2D drawing
- **Geolocation** — `navigator.geolocation.getCurrentPosition()`
- **LocalStorage / SessionStorage** — Key-value client-side storage
- **Drag & Drop** — Native drag-and-drop API
- **History API** — `pushState` / `popState` for SPA routing
- **Web Workers** — Background thread execution
- **WebSockets** — Full-duplex communication
- **Intersection Observer** — Lazy loading, infinite scroll
- **Resize Observer** — Element resize tracking

> ❓ **Interview Question:** Explain the difference between LocalStorage, SessionStorage, and Cookies. When would you use each? LocalStorage persists until explicitly cleared, SessionStorage lives only for the tab session, and Cookies are sent with every HTTP request (ideal for server-read values like session IDs).

### CSS3 Selectors

`css
[data-active] { }
[type="text"] { }
[class^="btn-"] { }
[class$="-primary"] { }
[class*="icon"] { }
:first-child, :last-child, :nth-child(2n)
:not(.disabled)
:is(h1, h2, h3)
:has(img)
:focus-visible
::before, ::after
`

### CSS Custom Properties

`css
:root {
  --primary: #2563eb;
  --spacing: 1rem;
}
.card {
  background: var(--primary);
  padding: var(--spacing);
}
`

> ✅ **Best Practice:** Prefer CSS custom properties over preprocessor variables for runtime theming. They cascade, can be overridden per component, and are natively supported everywhere. Always provide a fallback: `var(--primary, #2563eb)`.

### Common Mistakes

- **Not setting ox-sizing: border-box globally** — causes unexpected sizing
- **Overusing !important** — creates specificity wars
- **Ignoring the <meta viewport> tag** — breaks responsive layout
- **Inline styles for everything** — kills maintainability
- **Missing lt attributes** — accessibility failure

### Best Practices

- Use semantic HTML for screen readers and SEO
- Keep CSS specificity flat with utility-first or BEM naming
- Always set :focus-visible styles for keyboard users
- Use relative units (
em, em, h, w) over px where possible
- Reduce HTTP requests — inline critical CSS, defer non-critical

### 📝 Revision Notes

- HTML5 semantic elements improve SEO, accessibility, and code readability
- CSS custom properties enable runtime theming and cascade naturally
- Always set `<meta name="viewport">` for responsive rendering
- Use relative units (rem, em, vw) over fixed pixels for responsive design
- Prefer `box-sizing: border-box` globally to avoid sizing surprises

### 🛠️ Practical Exercises

1. **Build a semantic blog layout** — Create a blog page using `<header>`, `<nav>`, `<main>`, `<article>`, `<aside>`, and `<footer>` with proper ARIA landmarks
2. **CSS custom properties theme switcher** — Build a light/dark theme toggle using CSS custom properties and a JavaScript class toggle
3. **Responsive card component** — Create a card that displays horizontally on desktop and stacks vertically on mobile using only CSS (no frameworks)
4. **HTML5 form with validation** — Build a registration form with newer `<input>` types (email, tel, url, date) and the Constraint Validation API
5. **Accessibility audit** — Run axe DevTools on a local HTML page and fix all critical and serious violations found

---

## Chapter 36: Flexbox & CSS Grid

### Introduction

Flexbox and Grid are the two modern CSS layout engines. Flexbox excels at one-dimensional layouts (rows OR columns), while Grid handles two-dimensional layouts (rows AND columns).

> 🏛️ **Real World Analogy:** Flexbox is like a single bookshelf — you arrange books in one direction (horizontal or vertical). Grid is like a chessboard — you place pieces anywhere on a two-dimensional grid of rows and columns. Use the right tool for the layout dimension.

### Flexbox

`css
.container {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}
.item {
  flex: 1;
  align-self: flex-start;
  order: -1;
}
`

| Property | Values | Axis |
|----------|--------|------|
| justify-content | lex-start, lex-end, center, space-between, space-around, space-evenly | Main |
| lign-items | stretch, lex-start, lex-end, center, aseline | Cross |
| lign-content | Same as justify-content | Cross (multi-line) |
| lex-wrap | 
owrap, wrap, wrap-reverse | Cross |

> ⚠️ **Warning:** Forgetting `flex-wrap: wrap` is one of the most common Flexbox mistakes. Without it, flex items will shrink to fit the container — or overflow it — instead of wrapping to the next line. Always set `flex-wrap` unless you explicitly want a single-line layout.

### CSS Grid

`css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}
.grid-item {
  grid-column: 1 / 3;
  grid-row: 1 / 2;
}

.layout {
  display: grid;
  grid-template-areas:
    "header  header  header"
    "sidebar content content"
    "footer  footer  footer";
  grid-template-columns: 250px 1fr 1fr;
}
header { grid-area: header; }
aside  { grid-area: sidebar; }
main   { grid-area: content; }
footer { grid-area: footer; }
`

### Grid Functions & Keywords

| Function | Purpose |
|----------|---------|
| 
epeat(3, 1fr) | Repeated column/row definitions |
| minmax(200px, 1fr) | Flexible range |
| uto-fit / uto-fill | Responsive without media queries |
| min-content / max-content | Intrinsic sizing |
| it-content(300px) | Clamp between min and max |

`css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}
`

> 💡 **Pro Tip:** Use `auto-fit` and `auto-fill` with `minmax()` for responsive grids that adapt without media queries. The difference: `auto-fit` collapses empty tracks to 0, while `auto-fill` preserves their space.

### Flexbox vs Grid

| Aspect | Flexbox | Grid |
|--------|---------|------|
| Dimensions | 1D (row OR column) | 2D (rows AND columns) |
| Content vs Layout | Content-first | Layout-first |
| Item overlap | Difficult | Easy with grid-column / grid-row |
| Gap support | gap (modern browsers) | gap |
| Use case | Navbars, card rows, centering | Page layouts, dashboards, galleries |

> ❓ **Interview Question:** When would you choose CSS Grid over Flexbox, and vice versa? Grid is best for two-dimensional layouts where you control both rows and columns (e.g., full page layouts, dashboards). Flexbox is ideal for one-dimensional layouts — aligning items in a row or column (e.g., navbars, card rows, centering content).

### Common Mistakes

- Using Flexbox for full-page layouts when Grid is cleaner
- Forgetting lex-wrap: wrap then wondering why items overflow
- Mixing gap in older browsers without a fallback
- Nesting too many flex containers (performance + complexity)

### Best Practices

- Use Flexbox for component-level layouts
- Use Grid for page-level, two-dimensional layouts
- Combine both — Grid for outer layout, Flexbox for inner components
- Prefer gap over margin hacks
- Use uto-fit / uto-fill for truly responsive grids

### 📝 Revision Notes

- Flexbox works in one dimension (row or column); Grid works in two (rows and columns)
- Use `justify-content` for main-axis alignment, `align-items` for cross-axis alignment
- `grid-template-areas` provides visual, semantic layout definitions
- `auto-fit` and `auto-fill` with `minmax()` create responsive grids without media queries
- Combine Grid for page layout with Flexbox for component-level alignment

### 🛠️ Practical Exercises

1. **Holy Grail layout with Grid** — Build the classic header/sidebar/content/footer layout using `grid-template-areas`
2. **Card grid with auto-fit** — Create a responsive gallery that shows 1, 2, 3, or 4 columns depending on viewport width using `repeat(auto-fit, minmax(250px, 1fr))`
3. **Flexbox navbar** — Build a responsive navigation bar with logo on the left and links on the right that collapses to a column on mobile
4. **Centering challenge** — Center a child element both horizontally and vertically inside a parent using Flexbox (bonus: do it with Grid's `place-items`)

---

## Chapter 37: Responsive Design

### Introduction

Responsive Web Design (RWD) ensures applications render well across devices — phones, tablets, laptops, and desktops.

> 🏛️ **Real World Analogy:** Responsive design is like water — it takes the shape of whatever container it's in. A single HTML page should flow and adapt to any screen size, just like water fills a glass, a bowl, or a bottle.

### Media Queries

`css
.card { width: 100%; }

@media (min-width: 768px) {
  .card { width: 50%; }
}

@media (min-width: 1024px) {
  .card { width: 33.33%; }
}

@media (prefers-color-scheme: dark) {
  body { background: #111; color: #eee; }
}

@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
`

### Common Breakpoints

| Device | Width |
|--------|-------|
| Phone (portrait) | < 576px |
| Phone (landscape) | >= 576px |
| Tablet | >= 768px |
| Laptop | >= 992px |
| Desktop | >= 1200px |

> ⚠️ **Warning:** Never design for only one viewport size. Always test on actual mobile devices, not just browser DevTools. DevTools simulates size but not touch behavior, network conditions, or pixel density. Always set `touch-action: manipulation` to remove the 300ms tap delay on mobile.

### Responsive Images

`html
<img src="small.jpg" srcset="small.jpg 480w, medium.jpg 768w, large.jpg 1200w"
  sizes="(max-width: 768px) 100vw, 50vw" alt="Responsive image" />

<picture>
  <source media="(min-width: 1024px)" srcset="wide.jpg" />
  <source media="(min-width: 768px)" srcset="tablet.jpg" />
  <img src="mobile.jpg" alt="Art-directed image" />
</picture>
`

### Mobile-First vs Desktop-First

| Approach | Media Queries | Pros | Cons |
|----------|-------------|------|------|
| Mobile-first | min-width | Smaller baseline, progressive enhancement | Some desktop styles feel like overrides |
| Desktop-first | max-width | Matches traditional mental model | Bloated mobile CSS |

> ✅ **Best Practice:** Always design mobile-first. It forces prioritization of core content and naturally produces smaller, faster pages. Use `min-width` media queries so mobile styles are the baseline and desktop styles are enhancements.

### Common Mistakes

- Designing only for one viewport size
- Using fixed pixel widths for containers
- Hiding content on mobile instead of reorganizing
- Not testing touch targets (min 44x44px)
- Forgetting 	ouch-action: manipulation to remove 300ms tap delay

### Best Practices

- Start with a single-column mobile layout
- Use clamp() for fluid typography: ont-size: clamp(1rem, 2.5vw, 2rem)
- Set html { font-size: 100%; } — respect user's default font size
- Test on real devices, not just browser DevTools
- Use container queries for component-level responsiveness

### 📝 Revision Notes

- Mobile-first uses `min-width` media queries; desktop-first uses `max-width`
- Use `clamp()` for fluid typography that scales between minimum and maximum values
- Responsive images with `srcset` and `sizes` save bandwidth on mobile devices
- Touch targets should be at least 44x44px for accessibility
- Container queries allow component-level responsiveness independent of the viewport

### 🛠️ Practical Exercises

1. **Mobile-first landing page** — Build a landing page starting with a single-column mobile layout and progressively enhance for tablet and desktop
2. **Responsive image gallery** — Use `<picture>` and `srcset` to serve different image sizes and formats (WebP/AVIF) based on viewport width
3. **Fluid typography scale** — Implement a typography scale using `clamp()` for all heading levels and test across multiple viewport sizes
4. **Container query card** — Build a component that changes layout based on its container's width using `@container`

---

## Chapter 38: Tailwind CSS & Bootstrap

### Introduction

CSS frameworks accelerate development. Bootstrap pioneered the component-based approach; Tailwind CSS popularized utility-first methodology.

> 🏛️ **Real World Analogy:** Bootstrap is like a furniture catalog — you pick pre-assembled pieces (components) and arrange them. Tailwind CSS is like a hardware store — you buy raw materials (utilities) and build custom furniture. Both are valid; choose based on whether you value speed or customization.

### Tailwind CSS

`html
<div class="max-w-md mx-auto mt-8 p-6 bg-white rounded-xl shadow-lg">
  <h2 class="text-2xl font-bold text-gray-900">Welcome Back</h2>
  <p class="mt-2 text-gray-600">Sign in to your account</p>
  <form class="mt-6 space-y-4">
    <input type="email"
      class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
      placeholder="Email" />
    <button type="submit"
      class="w-full px-4 py-2 bg-blue-600 text-white font-semibold rounded-lg hover:bg-blue-700 transition">
      Sign In
    </button>
  </form>
</div>
`

#### Tailwind Config

`javascript
module.exports = {
  content: ["./src/**/*.{html,js,jsx,tsx}"],
  theme: {
    extend: {
      colors: { brand: { 500: \"#6366f1\", 600: \"#4f46e5\" } },
      fontFamily: { sans: [\"Inter\", \"sans-serif\"] },
    },
  },
};
`

> ⚠️ **Warning:** Always configure the `content` paths in `tailwind.config.js` correctly. If paths are wrong, Tailwind's JIT compiler won't scan your files, and classes you use in your templates will be missing in the production build. This is the #1 Tailwind debugging issue.

#### Key Utility Categories

| Category | Examples |
|----------|----------|
| Layout | container, lex, grid, lock, hidden |
| Spacing | p-4, m-2, gap-4, space-x-2 |
| Typography | 	ext-lg, ont-bold, leading-relaxed |
| Sizing | w-full, h-16, max-w-4xl |
| Background | g-white, g-gradient-to-r |
| Border | order, 
ounded-lg, order-gray-200 |
| Effects | shadow-lg, opacity-50, lur-sm |
| Transforms | scale-110, 
otate-45 |
| Transitions | 	ransition, duration-300, ease-in-out |
| Responsive | sm:, md:, lg:, xl:, 2xl: |
| State | hover:, ocus:, ctive:, disabled: |
| Dark mode | dark:bg-gray-900 |

> ✅ **Best Practice:** Extract repeated Tailwind utility patterns into reusable components or using `@apply` in your component library. Long class strings (10+) indicate you should refactor into a component. Use `theme.extend` in config instead of arbitrary values for consistent design tokens.

### Bootstrap 5

`html
<link href=\"https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css\" rel=\"stylesheet\" />

<div class=\"container mt-5\">
  <div class=\"row\">
    <div class=\"col-12 col-md-6 col-lg-4\">
      <div class=\"card shadow-sm\">
        <div class=\"card-body\">
          <h5 class=\"card-title\">Card Title</h5>
          <p class=\"card-text\">Some quick example text.</p>
          <a href=\"#\" class=\"btn btn-primary\">Go somewhere</a>
        </div>
      </div>
    </div>
  </div>
</div>
`

> 💡 **Pro Tip:** When using Tailwind with React, create a custom component library that wraps utility classes. For example, a `Button` component with props for variant, size, and disabled state encapsulates the Tailwind classes in one place. This gives you the flexibility of utilities with the maintainability of component abstraction.

### Tailwind vs Bootstrap

| Aspect | Tailwind CSS | Bootstrap |
|--------|-------------|-----------|
| Philosophy | Utility-first | Component-first |
| Learning curve | Steeper upfront | Easier to start |
| File size | Minimal (purged) | Larger (full library) |
| Customization | Config-driven | Sass variable overrides |
| Design consistency | Manual, but flexible | Consistent out of box |
| Bundle size (min+gzip) | ~3KB (purged) | ~20KB (JS + CSS) |
| Best for | Design-forward teams, custom UIs | Rapid prototyping, admin panels |

### Common Mistakes (Tailwind)

- Not purging unused classes in production
- Creating unmaintainable long class strings
- Fighting the framework instead of configuring 	heme.extend

> ❓ **Interview Question:** Compare Tailwind CSS and Bootstrap. When would you choose one over the other? Tailwind offers more design flexibility with utility classes and results in smaller bundles after purging. Bootstrap is faster to prototype with pre-built components and is better for admin panels or when consistent out-of-the-box design is needed.

### Best Practices (Tailwind)

- Use @apply only for reusable components
- Extract repeated patterns into components
- Leverage 	heme.extend instead of arbitrary values
- Always configure content paths correctly for purging

### 📝 Revision Notes

- Tailwind CSS is utility-first; Bootstrap is component-first
- Purge unused CSS with the `content` configuration in `tailwind.config.js`
- Use `theme.extend` to customize design tokens without overriding defaults
- Bootstrap's grid is 12-column based; Tailwind uses arbitrary grid utilities
- Both frameworks support responsive prefixes: `sm:`, `md:`, `lg:` (Tailwind) vs `col-md-6` (Bootstrap)

### 🛠️ Practical Exercises

1. **Build a Tailwind landing page** — Create a complete marketing landing page (hero, features, testimonials, footer) without writing any custom CSS
2. **Bootstrap dashboard layout** — Build a responsive admin dashboard with sidebar, header, and content area using Bootstrap's grid and components
3. **Custom Tailwind theme** — Configure a custom color palette, font family, and spacing scale in `tailwind.config.js` and build a consistent UI
4. **Recreate same design in both frameworks** — Build the same card component in Tailwind and Bootstrap, then compare the markup and file sizes

---

## Chapter 39: JavaScript & the DOM

### Introduction

JavaScript is the language of the web. Understanding JavaScript deeply is non-negotiable for full-stack developers.

> 🏛️ **Real World Analogy:** JavaScript is like a Swiss Army knife — it's a multi-tool that can manipulate the DOM, communicate with servers, handle user input, manage state, and run on both client and server. Mastering its quirks (closures, `this`, event loop) is what separates proficient developers from beginners.

### Core Concepts

- **Primitives**: string, 
umber, oolean, 
ull, undefined, symbol, igint
- **Objects**: {}, [], Map, Set, Date, RegExp
- **Type coercion**: implicit vs explicit (== vs ===)
- **Scoping**: global, function, block (let/const)
- **Closures**: function + its lexical scope
- **	his**: implicit binding, explicit (call, pply, ind), arrow functions
- **Prototypes**: prototypal inheritance vs classical
- **Event loop**: call stack, task queue, microtasks

`javascript
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}
const counter = createCounter();
console.log(counter.increment());
`

> ⚠️ **Warning:** A common performance pitfall is manipulating the DOM inside a loop with `appendChild` or `innerHTML +=`. Each DOM manipulation triggers a reflow/repaint. Instead, use `document.createDocumentFragment()` or build the HTML string and insert once. Batch reads together and writes together to avoid layout thrashing.

### DOM Manipulation

`javascript
const element = document.getElementById(\"app\");
const items = document.querySelectorAll(\".item\");
const parent = element.closest(\".container\");
element.textContent = \"Hello\";
element.classList.add(\"active\");
element.style.color = \"red\";

const div = document.createElement(\"div\");
div.textContent = \"New\";
document.body.appendChild(div);
`

### Event Handling

`javascript
element.addEventListener(\"click\", (e) => {
  e.preventDefault();
  e.stopPropagation();
  console.log(e.target);
});

document.querySelector(\"ul\").addEventListener(\"click\", (e) => {
  if (e.target.tagName === \"LI\") {
    console.log(\"Item clicked:\", e.target.textContent);
  }
});
`

> ✅ **Best Practice:** Use `addEventListener` over `onclick` (or other inline event handlers) because it allows multiple listeners on the same element, supports `passive` and `once` options, and separates behavior from markup. The `{ passive: true }` option improves scroll performance by telling the browser not to wait for `preventDefault()`.

### Event Types

| Category | Events |
|----------|--------|
| Mouse | click, dblclick, mousedown, mouseup, mousemove, mouseenter, mouseleave |
| Keyboard | keydown, keyup, keypress |
| Form | submit, change, input, ocus, lur |
| Window | load, 
esize, scroll, eforeunload |
| Drag | dragstart, dragover, drop, dragend |
| Touch | 	ouchstart, 	ouchmove, 	ouchend |

> 💡 **Pro Tip:** Use event delegation whenever you have dynamic lists or repeated elements. Instead of attaching an event listener to every `<li>`, attach one to the parent `<ul>` and use `e.target.closest()` to identify which child was clicked. This saves memory and automatically handles dynamically added elements.

### Common Mistakes

- Using == instead of ===
- Not understanding 	his context in callbacks
- Memory leaks from uncleaned event listeners
- Blocking the main thread with synchronous operations

### Best Practices

- Always use strict equality (===)
- Prefer const over let; never use ar
- Use event delegation for dynamic lists
- Debounce or throttle scroll/resize handlers

### 📝 Revision Notes

- JavaScript has 7 primitive types and object types; type coercion can cause subtle bugs
- Closures enable data privacy and factory functions
- The event loop (call stack → task queue → microtask queue) determines execution order
- Event delegation improves performance by using a single parent listener
- Memory leaks occur from uncleaned event listeners, intervals, and detached DOM references

### 🛠️ Practical Exercises

1. **Closure-based counter** — Create a `createCounter` function with `increment`, `decrement`, and `reset` methods using closures (no class syntax)
2. **Event delegation list** — Build a to-do list where clicking any item marks it as complete using a single event listener on the parent `<ul>`
3. **Debounced search input** — Implement a search input that fetches results only after the user stops typing for 300ms
4. **DOM manipulation benchmark** — Create 1000 elements using innerHTML vs createElement + appendChild and measure the performance difference

---

## Chapter 40: ES6+ Features

### Introduction

ECMAScript 2015 (ES6) and subsequent releases modernized JavaScript with classes, modules, promises, destructuring, arrow functions, and more.

> 🏛️ **Real World Analogy:** ES6+ features are like upgrading from a basic toolkit to a power tool set. Destructuring is like having pre-sorted drawers (you grab only what you need). Arrow functions eliminate the `this` confusion. Async/await turns callback spaghetti into clean, synchronous-looking code.

### Key Features

`javascript
const PI = 3.14;
let count = 0;

const add = (a, b) => a + b;
const square = x => x ** 2;

const name = \"World\";
console.log(\Hello, \!\);

const [first, second] = [1, 2, 3];
const { name: userName, age = 25 } = user;

const merged = [...arr1, ...arr2];
const clone = { ...original };

function greet(name = \"Guest\") { return \Hi \\; }
`

> ⚠️ **Warning:** Arrow functions don't have their own `this` binding — they inherit it from the enclosing scope. This makes them great for callbacks but unsuitable as methods in objects or classes where you need dynamic `this`. Use regular functions for object methods and constructors.

### Promises & Async/Await

`javascript
const fetchData = (url) => {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then((res) => {
        if (!res.ok) reject(new Error(res.statusText));
        return res.json();
      })
      .then(resolve)
      .catch(reject);
  });
};

async function loadUser(id) {
  try {
    const user = await fetchData(\/api/users/\\);
    const posts = await fetchData(\/api/users/\/posts\);
    return { user, posts };
  } catch (error) {
    console.error(\"Failed to load:\", error);
    throw error;
  }
}
`

> ❓ **Interview Question:** Explain the difference between Promises and async/await. How do they relate? Async/await is syntactic sugar over Promises — it makes asynchronous code read like synchronous code. The `await` keyword can only be used inside an `async` function, and error handling is done with try/catch instead of `.catch()`.

### Modules

`javascript
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export default class Calculator { }

// app.js
import Calculator, { PI, add as sum } from \"./math.js\";
`

### Modern Array Methods

| Method | Returns | Purpose |
|--------|---------|---------|
| map | New array | Transform every element |
| ilter | New array | Keep matching elements |
| 
educe | Single value | Accumulate to a result |
| ind | Element (or undefined) | First match |
| indIndex | Index (or -1) | Index of first match |
| some | Boolean | Any element passes test |
| every | Boolean | All elements pass test |
| lat | New array | Flatten nested arrays |
| latMap | New array | Map then flat(1) |

### Newer ES Features (ES2020+)

| Feature | Example | Year |
|---------|---------|------|
| Optional chaining | user?.address?.street | 2020 |
| Nullish coalescing | 
ame ?? \"Guest\" | 2020 |
| Promise.allSettled | Promise.allSettled([...]) | 2020 |
| Logical assignment |  &&= b,  ||= b,  ??= b | 2021 |
| Numeric separators | 1_000_000 | 2021 |
| Array .at() | rr.at(-1) | 2022 |
| Top-level await | wait fetch(url) in module | 2022 |

### Common Mistakes

- Mixing 
ull and undefined with nullish coalescing
- Forgetting optional chaining on deeply nested objects
- Using map without returning a value

> ✅ **Best Practice:** Use optional chaining (`?.`) and nullish coalescing (`??`) together for safe deep property access: `const street = user?.address?.street ?? 'No address'`. This replaces verbose `if` checks and prevents "Cannot read property of undefined" errors.

### Best Practices

- Default to const; use let only when rebinding
- Prefer sync/await over .then().catch()
- Use destructuring in function parameters
- Use ?. and ?? to avoid verbose null checks

### 📝 Revision Notes

- ES6 introduced `let`/`const`, arrow functions, destructuring, spread/rest, template literals, and modules
- Promises have three states: pending, fulfilled, rejected
- Async/await makes Promise-based code more readable
- Modern array methods (`map`, `filter`, `reduce`, `find`) replace imperative loops
- Optional chaining `?.` and nullish coalescing `??` prevent common runtime errors

### 🛠️ Practical Exercises

1. **Async data fetching** — Write a function that fetches user data and their posts in parallel using `Promise.all()` and displays them
2. **Array method chaining** — Given an array of user objects, chain `filter`, `map`, and `reduce` to get the total spending of active users
3. **Module refactor** — Take a single-file JavaScript utility and split it into ES modules with named and default exports
4. **Safe property access** — Write a function that safely accesses `user.settings.notifications.email` with optional chaining and provides defaults with nullish coalescing

---

## Chapter 41: TypeScript

### Introduction

TypeScript is a typed superset of JavaScript that catches type-related bugs at compile time and provides better IDE support.

> 🏛️ **Real World Analogy:** TypeScript is to JavaScript what a spell checker is to a word processor. You can write without it, but with it you catch errors before anyone sees them. In large codebases, TypeScript acts like a safety net — catching type mismatches, missing properties, and API contract violations at compile time.

### Why It Matters

In 2023, TypeScript was used by 89% of professional JavaScript developers. For large codebases, TypeScript reduces runtime errors by 15-30%.

> ⚠️ **Warning:** Avoid using `any` as a type — it defeats the purpose of TypeScript by disabling type checking entirely. Prefer `unknown` when you don't know the type, then narrow it with type guards. Use `@typescript-eslint/no-explicit-any` in your ESLint config to catch accidental `any` usage.

### Core Types

`	ypescript
let name: string = \"Alice\";
let age: number = 30;
let isActive: boolean = true;
let items: string[] = [\"a\", \"b\"];
let pair: [string, number] = [\"age\", 30];

enum Direction { Up = \"UP\", Down = \"DOWN\" }

type Status = \"idle\" | \"loading\" | \"error\";
type HasName = { name: string };
type HasAge = { age: number };
type Person = HasName & HasAge;
`

> ✅ **Best Practice:** Use interfaces for object shapes in public APIs (they support declaration merging and provide better error messages). Use type aliases for unions, intersections, mapped types, and computed properties. When in doubt, start with `interface` and switch to `type` only when you need union types or mapped types.

### Interfaces vs Types

| Feature | Interface | Type Alias |
|---------|-----------|------------|
| Declaration merging | Yes | No |
| Extends | extends | & intersection |
| Computed properties | No | Yes |
| Mapped types | No | Yes |
| Class implements | Yes | Yes |

`	ypescript
interface User {
  id: number;
  name: string;
  email: string;
  address?: Address;
  readonly createdAt: Date;
}

interface Admin extends User {
  role: \"admin\";
  permissions: string[];
}

type ApiResponse<T> = {
  data: T;
  error: null | string;
  status: number;
};
`

### Generics

`	ypescript
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

interface Repository<T> {
  getById(id: string): Promise<T>;
  create(data: Omit<T, \"id\">): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Partial<T> = { [K in keyof T]?: T[K] };
`

### Type Narrowing

`	ypescript
function process(value: string | number | null) {
  if (value === null) { }
  else if (typeof value === \"string\") { }
  else { }
}

type Shape =
  | { kind: \"circle\"; radius: number }
  | { kind: \"rect\"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case \"circle\": return Math.PI * shape.radius ** 2;
    case \"rect\":   return shape.width * shape.height;
  }
}
`

### Utility Types

| Type | Purpose |
|------|---------|
| Partial<T> | All properties optional |
| Required<T> | All properties required |
| Readonly<T> | All properties readonly |
| Pick<T, K> | Select specific keys |
| Omit<T, K> | Exclude specific keys |
| Record<K, V> | Object type with key/value types |
| Exclude<T, U> | Exclude types from union |
| Extract<T, U> | Extract types from union |
| ReturnType<T> | Extract return type of function |
| Parameters<T> | Extract parameter types |

### Common Mistakes

- Using ny everywhere instead of proper types
- Not enabling strict: true in 	sconfig.json
- Overusing type assertions (s)
- Ignoring 
ull and undefined in type definitions

> ✅ **Best Practice:** Always enable `strict: true` in `tsconfig.json`. This enables all strict type-checking options including `noImplicitAny` and `strictNullChecks`. Starting with strict mode prevents hundreds of bugs that would only surface at runtime.

### Best Practices

- Enable strict: true from the start
- Prefer interface for public API contracts, 	ype for unions and mapped types
- Use unknown instead of ny
- Leverage satisfies operator (TS 4.9+)

### 📝 Revision Notes

- TypeScript adds static types to JavaScript; it compiles down to plain JS
- Interfaces support declaration merging; type aliases support unions and mapped types
- Generics enable reusable, type-safe functions and data structures
- Utility types like `Partial`, `Pick`, `Omit`, and `Record` reduce boilerplate
- Always enable `strict: true` and prefer `unknown` over `any`

### 🛠️ Practical Exercises

1. **Type-safe API client** — Define interfaces for API responses and write a generic fetch wrapper that returns typed data
2. **Discriminated union** — Model a payment system with union types (CreditCard, PayPal, BankTransfer) and write a type-narrowing function
3. **Generic repository** — Implement a generic in-memory repository with CRUD methods and proper TypeScript generics
4. **Utility type challenge** — Create custom utility types: `DeepPartial<T>`, `NestedPick<T, K>`, and `NonFunctionKeys<T>`

---

## Chapter 42: React — Hooks, Context API, Redux Toolkit, React Query

### Introduction

React is the dominant frontend library for building user interfaces. Modern React is built on hooks, functional components, and a rich ecosystem for state management and data fetching.

> 🏛️ **Real World Analogy:** React components are like LEGO bricks — you build small, reusable pieces and snap them together to create complex UIs. State management (useState/useReducer) is the instruction manual, and React's virtual DOM is like a smart assistant that only rebuilds the parts that changed.

### Why It Matters

React's component model, virtual DOM, and declarative approach have influenced every modern UI framework.

### Functional Components & Hooks

`	sx
import { useState, useEffect, useCallback, useMemo, useRef } from \"react\";

export const Counter = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue);
  const prevCountRef = useRef(initialValue);

  useEffect(() => {
    document.title = \Count: \\;
    prevCountRef.current = count;
  }, [count]);

  const increment = useCallback(() => setCount((p) => p + 1), []);
  const isEven = useMemo(() => count % 2 === 0, [count]);

  return (
    <div>
      <p>Count: {count} ({isEven ? \"even\" : \"odd\"})</p>
      <p>Previous: {prevCountRef.current}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
};
`

> ⚠️ **Warning:** Never call hooks conditionally, inside loops, or after a return statement. The Rules of Hooks require that hooks are always called in the same order on every render. Violating this causes bugs where state values get mismatched between renders. ESLint plugin `eslint-plugin-react-hooks` catches these violations automatically.

### Built-in Hooks Reference

| Hook | Purpose |
|------|---------|
| useState | Local component state |
| useEffect | Side effects (API calls, subscriptions, DOM) |
| useContext | Consume context value |
| useReducer | Complex state logic with reducer pattern |
| useCallback | Memoized callback |
| useMemo | Memoized computed value |
| useRef | Mutable ref (DOM node, previous value) |
| useImperativeHandle | Customize exposed ref handle |
| useLayoutEffect | Synchronous effect after DOM mutations |
| useDebugValue | Display label in React DevTools |
| useDeferredValue | Defer re-rendering for non-urgent state |
| useTransition | Mark state updates as non-urgent |
| useId | Unique ID generation for a11y |

### Context API

`	sx
import { createContext, useContext, useState } from \"react\";

const AuthContext = createContext(undefined);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const login = async (email, password) => {
    const res = await fetch(\"/api/login\", {
      method: \"POST\",
      body: JSON.stringify({ email, password }),
    });
    const data = await res.json();
    setUser(data.user);
  };

  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error(\"useAuth must be used within AuthProvider\");
  return context;
};
`

### Redux Toolkit

`	ypescript
import { configureStore, createSlice, PayloadAction } from \"@reduxjs/toolkit\";
import { TypedUseSelectorHook, useDispatch, useSelector } from \"react-redux\";

interface Todo { id: string; text: string; completed: boolean; }

const todosSlice = createSlice({
  name: \"todos\",
  initialState: { items: [] as Todo[], filter: \"all\" as \"all\"|\"active\"|\"completed\" },
  reducers: {
    addTodo: (state, action: PayloadAction<string>) => {
      state.items.push({ id: crypto.randomUUID(), text: action.payload, completed: false });
    },
    toggleTodo: (state, action: PayloadAction<string>) => {
      const todo = state.items.find((t) => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
    setFilter: (state, action) => { state.filter = action.payload; },
  },
});

export const { addTodo, toggleTodo, setFilter } = todosSlice.actions;
export const store = configureStore({ reducer: { todos: todosSlice.reducer } });

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
`

> ✅ **Best Practice:** Configure sensible defaults for React Query at the provider level: `staleTime: 5 * 60 * 1000` (5 min), `gcTime: 30 * 60 * 1000` (30 min), and `retry: 2`. Override per-query as needed. This prevents excessive refetching while ensuring data is reasonably fresh. Use `refetchOnWindowFocus: false` if background refetching on tab switch is undesirable.

### React Query (TanStack Query)

`	sx
import { useQuery, useMutation, useQueryClient } from \"@tanstack/react-query\";

export const UsersList = () => {
  const { data, isLoading, error } = useQuery({
    queryKey: [\"users\"],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000,
    retry: 2,
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <ul>{data?.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
};

export const AddUser = () => {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: [\"users\"] }),
  });
  return (
    <button onClick={() => mutation.mutate({ name: \"New User\" })}>
      {mutation.isPending ? \"Adding...\" : \"Add User\"}
    </button>
  );
};
`

> 💡 **Pro Tip:** You rarely need Redux Toolkit if you already use React Query for server state and Context for app-wide state. Redux Toolkit shines in complex client-side state scenarios like multi-step wizards, real-time collaboration state, or undo/redo features. Start simple with React Query + Context and add Redux only when needed.

### React Query vs Redux Toolkit vs Context API

| Aspect | Context API | Redux Toolkit | React Query |
|--------|------------|---------------|-------------|
| Purpose | Prop drilling solution | Client-side state | Server state (caching) |
| Boilerplate | Low | Medium | Low |
| Performance | Re-renders all consumers | Selective subscriptions | Built-in caching + dedup |
| Async | Manual | createAsyncThunk | Built-in |
| Caching | None | Manual | Sophisticated |
| Best for | Theme, auth, locale | Complex client state | API data, cache, pagination |

> ✅ **Best Practice:** Use React Query for server data, Context for lightweight app-wide state (auth, theme), and Redux Toolkit only for complex client-side state management. This separation of concerns keeps each tool doing what it does best.

### Common Mistakes

- Calling hooks conditionally (violates Rules of Hooks)
- Forgetting dependency array in useEffect
- Mutating state directly instead of using setter
- Over-optimizing with useMemo / useCallback prematurely

> ❓ **Interview Question:** What is the difference between `useMemo` and `useCallback`? Both are memoization hooks. `useMemo` caches the result of a computation: `const total = useMemo(() => expensiveCalc(items), [items])`. `useCallback` caches a function reference: `const handleClick = useCallback(() => doSomething(id), [id])`. Use `useMemo` for expensive calculations and `useCallback` to prevent unnecessary re-renders of child components.

### Best Practices

- Co-locate state as close to where it's used
- Use React Query for all server data fetching
- Write custom hooks for reusable logic
- Always provide a key when rendering lists
- Use Suspense for loading states (React 18+)

### 📝 Revision Notes

- React hooks (useState, useEffect, useCallback, useMemo) replace class lifecycle methods
- Context API solves prop drilling but causes re-renders in all consumers
- Redux Toolkit uses Immer for immutable state updates and creates slices
- React Query handles caching, deduplication, background refetching, and pagination
- Choose tools by concern: React Query (server), Context (app-wide), Redux (complex client)

### 🛠️ Practical Exercises

1. **Custom useDebounce hook** — Create a custom hook that debounces a value and use it in a search input connected to an API
2. **Context + useReducer auth flow** — Build an authentication system using Context and useReducer with login, logout, and token refresh
3. **React Query infinite scroll** — Implement infinite scrolling with `useInfiniteQuery` from TanStack Query and a virtualized list
4. **Redux Toolkit todo app** — Build a todo app with createSlice, configureStore, and typed hooks (useAppDispatch/useAppSelector)
5. **Performance optimization** — Profile a component with React DevTools, identify unnecessary re-renders, and fix them with `React.memo`, `useMemo`, and `useCallback`

---

## Chapter 43: Next.js

### Introduction

Next.js is the most popular React framework for production. It provides SSR, SSG, API routes, and file-based routing.

> 🏛️ **Real World Analogy:** Next.js rendering strategies are like restaurant service models. SSG is a buffet — everything is pre-prepared (cooked at build time). SSR is a chef cooking to order — fresh for each request. ISR is meal prep — you cook batches and reheat on a schedule. Choose based on how fresh the content needs to be.

> ⚠️ **Warning:** Don't fetch data directly inside React components in Next.js without using the appropriate data-fetching pattern. Client-side fetching inside components means no SSR/SSG benefits, hurting SEO and initial load time. Use `getStaticProps`, `getServerSideProps`, or server components (App Router) for data that should be available at render time.

### Rendering Strategies

`	sx
// SSG — at build time
export async function getStaticProps() {
  const data = await fetch(\"https://api.example.com/posts\");
  const posts = await data.json();
  return { props: { posts }, revalidate: 60 };
}

// SSR — on every request
export async function getServerSideProps(context) {
  const data = await fetch(\https://api.example.com/post/\\);
  const post = await data.json();
  return { props: { post } };
}
`

| Strategy | When to Use | Freshness |
|----------|-------------|-----------|
| SSG | Static content, blogs, marketing | Stale until rebuild |
| ISR | Content that updates occasionally | Stale up to 
evalidate |
| SSR | Dynamic, personalized content | Always fresh |
| CSR | Dashboard, authenticated pages | After client JS loads |

### File-Based Routing

`
pages/index.tsx            -> /
pages/blog/[slug].tsx      -> /blog/:slug
pages/api/users/[id].ts    -> /api/users/:id

app/layout.tsx             -> shared layout
app/page.tsx               -> /
app/blog/[slug]/page.tsx   -> /blog/:slug
app/api/users/route.ts     -> /api/users
`

### API Routes (App Router)

`	sx
import { NextRequest, NextResponse } from \"next/server\";

export async function GET(request: NextRequest) {
  const users = await db.listUsers();
  return NextResponse.json(users);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const user = await db.createUser(body);
  return NextResponse.json(user, { status: 201 });
}
`

### Middleware

`	sx
import { NextResponse } from \"next/server\";
import type { NextRequest } from \"next/server\";

export function middleware(request: NextRequest) {
  const token = request.cookies.get(\"token\")?.value;
  if (request.nextUrl.pathname.startsWith(\"/admin\") && !token) {
    return NextResponse.redirect(new URL(\"/login\", request.url));
  }
  return NextResponse.next();
}

export const config = { matcher: [\"/admin/:path*\"] };
`

### Common Mistakes

- Not using 
ext/image for images
- Fetching data in components instead of SSG/SSR methods
- Forgetting 
evalidate with getStaticProps

> ✅ **Best Practice:** Always use `next/image` for images instead of `<img>`. It provides automatic lazy loading, responsive srcset generation, WebP/AVIF conversion, and CLS prevention via required `width` and `height` attributes. This single change can improve your Lighthouse performance score by 10-15 points.

### Best Practices

- Use SSG + ISR for public content-heavy pages
- Use SSR only for truly dynamic, personalized content
- Use 
ext/image with lazy loading
- Implement generateStaticParams for dynamic SSG routes
- Use Middleware for auth and redirects

### 📝 Revision Notes

- Next.js supports SSG (build-time), SSR (request-time), ISR (time-based revalidation), and CSR
- App Router uses nested layouts, server components, and file-based routing
- Middleware runs at the edge for auth redirects, i18n, and bot detection
- `next/image` optimizes images with lazy loading, responsive sizes, and modern formats
- API Routes in App Router use the Web `Request`/`Response` API

### 🛠️ Practical Exercises

1. **Next.js blog with ISR** — Build a blog using SSG with `revalidate` for ISR, dynamic routes for posts, and `generateStaticParams` for pre-generated paths
2. **Middleware auth guard** — Implement a middleware that checks for authentication cookies and redirects unauthenticated users to a login page
3. **App Router dashboard** — Create a dashboard with a shared `layout.tsx`, nested routes for settings/profile, and a loading skeleton
4. **API route with validation** — Build a Next.js API route that accepts POST requests, validates the body with Zod, and returns proper error responses

---

## Chapter 44: Authentication (Frontend)

### Introduction

Frontend authentication involves managing user identity, sessions, tokens, and protected routes.

> 🏛️ **Real World Analogy:** Authentication tokens are like concert wristbands. An access token is a VIP pass (short-lived, grants entry). A refresh token is your receipt (longer-lived, you use it to get a new pass when the first expires). localStorage is like keeping the wristband visible — convenient but vulnerable. httpOnly cookies are like keeping it under your sleeve — safer from pickpockets (XSS).

### Authentication Patterns

`	sx
export const useAuth = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem(\"access_token\");
    if (token) {
      fetchUser(token)
        .then(setUser)
        .catch(() => localStorage.removeItem(\"access_token\"))
        .finally(() => setLoading(false));
    } else setLoading(false);
  }, []);

  const login = async (email, password) => {
    const { accessToken, refreshToken } = await loginApi(email, password);
    localStorage.setItem(\"access_token\", accessToken);
    localStorage.setItem(\"refresh_token\", refreshToken);
    const user = await fetchUser(accessToken);
    setUser(user);
  };

  const logout = () => {
    localStorage.removeItem(\"access_token\");
    localStorage.removeItem(\"refresh_token\");
    setUser(null);
  };

  return { user, loading, login, logout, isAuthenticated: !!user };
};
`

### Token Storage Comparison

| Storage | Scope | Persistence | XSS Vulnerability |
|---------|-------|-------------|-------------------|
| localStorage | Tab to Tab | Yes | High (JS access) |
| sessionStorage | Per tab | Tab lifetime | High (JS access) |
| httpOnly cookie | Domain | Configurable | Low (no JS access) |
| In-memory | App lifetime | No | Low |

> ⚠️ **Warning:** Storing tokens in `localStorage` makes them accessible to any JavaScript running on the page. If an XSS vulnerability exists, attackers can steal all tokens. Prefer `httpOnly` cookies which are inaccessible to JavaScript, or use in-memory storage with a refresh token in an `httpOnly` cookie.

### Common Mistakes

- Storing tokens in localStorage without XSS protection
- Not handling token refresh failures gracefully
- Trusting client-side auth checks as sole security measure

> ❓ **Interview Question:** How do you securely store JWT tokens on the client? The most secure approach is using `httpOnly`, `Secure`, `SameSite=Strict` cookies for the refresh token, with the access token held in memory (JavaScript variable). This prevents XSS attacks from accessing tokens while allowing the server to set/clear cookies via `Set-Cookie` headers.

### Best Practices

- Always validate tokens on the server
- Use short-lived access tokens (15 min) with refresh tokens
- Implement refresh token rotation
- Use httpOnly, Secure, SameSite=Strict cookies where possible
- Clear all stored tokens on logout

### 📝 Revision Notes

- Access tokens are short-lived (15 min); refresh tokens are long-lived (7 days)
- `httpOnly` cookies are inaccessible to JavaScript, preventing XSS token theft
- Always validate tokens server-side — never trust client-side checks alone
- Implement refresh token rotation to invalidate stolen refresh tokens
- Rate-limit login endpoints to prevent brute force attacks

### 🛠️ Practical Exercises

1. **JWT auth flow** — Build a login page that stores an access token in memory and a refresh token in an httpOnly cookie, with automatic token refresh on 401 responses
2. **Protected routes with React Router** — Implement a `ProtectedRoute` component that redirects unauthenticated users to the login page
3. **Auth context + provider** — Create a React AuthContext with login, logout, and session restoration on page reload
4. **Refresh token interceptor** — Write an Axios interceptor that catches 401 errors, refreshes the token, and retries the original request

---

## Chapter 45: Performance, SEO & Accessibility

### Introduction

A successful web application must be fast, discoverable, and usable by everyone.

> 🏛️ **Real World Analogy:** Performance, SEO, and Accessibility form a tripod supporting user experience. If one leg is weak, the whole experience suffers. A fast page that's inaccessible excludes millions of users. A well-ranking page that loads slowly loses visitors to bounce rates. All three must be balanced for a production-quality web application.

### Performance Metrics (Core Web Vitals)

| Metric | Good | Poor |
|--------|------|------|
| LCP | <= 2.5s | > 4.0s |
| FID | <= 100ms | > 300ms |
| CLS | <= 0.1 | > 0.25 |
| TTFB | <= 800ms | > 1.8s |
| INP | <= 200ms | > 500ms |

### Performance Optimization Techniques

`javascript
const HeavyComponent = React.lazy(() => import(\"./HeavyComponent\"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
`

`html
<img loading=\"lazy\" src=\"image.jpg\" alt=\"description\" width=\"800\" height=\"600\" />
<link rel=\"preload\" href=\"/fonts/inter.woff2\" as=\"font\" type=\"font/woff2\" crossorigin />
<link rel=\"preconnect\" href=\"https://api.example.com\" />
`

| Technique | Impact | Effort |
|-----------|--------|--------|
| Image optimization (WebP/AVIF) | High | Low |
| Code splitting | High | Medium |
| CDN caching | High | Medium |
| Server-side rendering | High | High |
| Lazy loading images | Medium | Low |

> ⚠️ **Warning:** Not compressing images is the most common performance killer. Unoptimized images make up 60-80% of a typical page's weight. Always serve modern formats (WebP/AVIF), use responsive `srcset`, and apply lazy loading with `loading="lazy"`. Tools like Sharp, Squoosh, and `next/image` handle this automatically.

### SEO Fundamentals

`	sx
export const metadata = {
  title: { template: \"%s | My App\", default: \"My App\" },
  description: \"Best app for everything\",
  openGraph: {
    title: \"My App\",
    description: \"Best app for everything\",
    images: [{ url: \"/og-image.png\", width: 1200, height: 630 }],
  },
  robots: { index: true, follow: true },
};
`

#### Technical SEO Checklist

- Semantic HTML (h1-h6 hierarchy, landmarks)
- Unique, descriptive <title> per page
- Meta descriptions (<160 chars)
- Canonical URLs
- sitemap.xml and 
obots.txt
- Structured data (JSON-LD Schema.org)
- Fast page speed (Core Web Vitals)

### Accessibility (a11y)

`html
<a href=\"#main-content\" class=\"skip-link\">Skip to main content</a>
<button aria-label=\"Close dialog\">&times;</button>
<div aria-live=\"polite\" aria-atomic=\"true\">{{ notification }}</div>
<label for=\"email\">Email address</label>
<input id=\"email\" type=\"email\" aria-describedby=\"email-hint\" />
`

#### WCAG 2.2 Levels

| Level | Conformance |
|-------|-------------|
| A | Minimum — essential for some users |
| AA | Standard — remove major barriers |
| AAA | Highest — enhance usability for all |

`css
:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
`

### Common Mistakes

- Not compressing images
- Blocking render with large JavaScript bundles
- Not leveraging browser caching
- Missing lt text on images
- Low color contrast (below WCAG AA 4.5:1)
- Missing form labels

> ❓ **Interview Question:** What are Core Web Vitals and why do they matter? Core Web Vitals are Google's set of real-world performance metrics: LCP (loading speed — under 2.5s), FID/INP (interactivity — under 200ms), and CLS (visual stability — under 0.1). They directly impact search rankings and user experience metrics like bounce rate and conversion.

### Best Practices

- Use Lighthouse, WebPageTest, and Chrome DevTools for performance audits
- Set performance budgets and enforce in CI
- Run axe-core or WAVE for automated a11y testing
- Perform manual keyboard-only navigation tests
- Use prefers-reduced-motion to respect user preferences

### 📝 Revision Notes

- Core Web Vitals: LCP (2.5s), INP (200ms), CLS (0.1) — these are Google ranking factors
- Code splitting with `React.lazy()` and dynamic imports reduces initial bundle size
- Semantic HTML and proper heading hierarchy are the foundation of SEO
- WCAG 2.2 defines three conformance levels: A, AA, AAA
- Accessibility is not optional — 15% of the global population has some disability

### 🛠️ Practical Exercises

1. **Lighthouse audit** — Run a Lighthouse audit on a personal project, document all issues, and fix them to achieve a score of 90+ in all categories
2. **Accessibility remediation** — Take a non-accessible page, audit it with axe DevTools, and fix all critical/serious violations (missing labels, low contrast, missing alt text, keyboard navigation)
3. **Performance budget** — Set up a performance budget (max 200KB JS, 500KB images, 2s TTFB) and use Webpack/Vite bundle analyzer to find optimization opportunities
4. **Structured data implementation** — Add JSON-LD structured data (Article, FAQ, or Product schema) to a page and test it with Google's Rich Results Test

---
