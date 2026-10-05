# CSS 101: Self-Paced Tutorial

Missed the live session? This tutorial covers the same material, in the same order, at your own pace. You'll take a page of plain, unstyled HTML (the **Monster Management Bureau**) and style it step by step until it looks like `monsters-finished.html`. Explanations are included along the way.

**Time:** about 75–90 minutes. Take breaks between parts.
**You need:** VS Code, plus Chrome or Edge (any recent version).
**Background:** if you know C#, you know enough. Wherever it helps, the tutorial compares CSS to things you already use.

[CSS-Training.zip](https://github.com/mobiletonster/blogposts/raw/refs/heads/main/code/web/CSS-Training.zip)
---

## Contents

0. [Setup](#0-setup)
1. [The bare page and DevTools](#1-the-bare-page-and-devtools)
2. [Selectors are queries](#2-selectors-are-queries)
3. [Cascade and specificity](#3-cascade-and-specificity)
4. [STEP 1: Variables and base styles](#4-step-1-variables-and-base-styles)
5. [STEP 2: The box model](#5-step-2-the-box-model)
6. [STEP 3: Layout with Flexbox and Grid](#6-step-3-layout-with-flexbox-and-grid)
7. [STEP 4: Making it feel alive](#7-step-4-making-it-feel-alive)
8. [Your turn: a solo Style-Off](#8-your-turn-a-solo-style-off)
9. [How this maps to Blazor, and what's next](#9-how-this-maps-to-blazor-and-whats-next)
10. [Troubleshooting](#10-troubleshooting)
11. [Quick reference](#11-quick-reference)

---

## 0. Setup

### The files

```
CSS-Training/
├── monsters-start.html      ← the page you'll style. It links to styles.css
├── styles.css               ← empty. This is the file you build
├── monsters-finished.html   ← the finished result, for comparison
├── images/                  ← one avatar per monster
└── checkpoints/
    ├── step-1.css           ← answer key after STEP 1
    ├── step-2.css           ← answer key after STEP 2
    ├── step-3.css           ← answer key after STEP 3
    └── step-4.css           ← answer key after STEP 4 (the finished page)
```

### The workflow

1. Open the `CSS-Training` folder in VS Code.
2. Open `monsters-start.html` in Chrome or Edge. Double-clicking the file works.
3. Edit `styles.css` in VS Code, **save**, then **refresh** the browser (F5). Repeat.

> **Tip:** the VS Code extension **Live Server** reloads the browser for you every time you save. It's optional but nice to have.

### If you get stuck

Each checkpoint file contains everything up to the end of that step. Copy its contents over your `styles.css`, refresh, and carry on. You can't break anything for good.

> **Before you start,** open `monsters-finished.html` too and click **Lights out**. That's where you're heading. Then compare it with `monsters-start.html`. The two files are **identical except for one line** in the `<head>`, the one that says which stylesheet to load. Every difference you see is CSS.

---

## 1. The bare page and DevTools

Open `monsters-start.html`. It looks like a document from 1996: big black headings, blue underlined links, bulleted lists.

### Even "no CSS" has CSS

Press **F12** to open DevTools, go to the **Elements** tab, and click the `<h1>`. In the **Styles** pane on the right, you'll see rules labeled **user agent stylesheet**. Every browser ships with a built-in stylesheet. That's why headings are big and lists have bullets even though you haven't written any CSS. Everything you write builds on top of those defaults, or overrides them.

### The Console is your playground

Switch to the **Console** tab and try these one at a time:

```js
$0                                  // the element you selected in the Elements panel
getComputedStyle($0).fontSize       // the value the browser actually used, e.g. "32px"
document.designMode = "on"          // now click any text on the page and type
document.designMode = "off"         // back to normal
```

`$0` always refers to whatever element is selected in the Elements panel. It's a quick way to poke at something you see on the page.

> **Key idea:** DevTools is where you **experiment**. Changes there disappear when you refresh. Once something works, copy it into `styles.css`.

---

## 2. Selectors are queries

Every CSS rule has two parts: **which elements** (the selector) and **what to do to them** (the declarations).

```css
h3 { color: purple; }
/* ↑ selector   ↑ declaration (property: value) */
```

Here's the important part: **CSS selectors and JavaScript's `document.querySelectorAll()` use exactly the same language.** If you can query it, you can style it. So you'll learn selectors in the Console first, where you get instant feedback.

> **C# analogy:** a selector works like a LINQ query over a tree of elements. `.card[data-status="sleeping"] h3` reads a lot like `monsters.Where(m => m.Status == Sleeping).Select(m => m.Name)`.

### A highlighter helper

Paste this into the Console. It outlines everything a selector matches in hot pink, and returns the count:

```js
const hl  = s => { clr(); document.querySelectorAll(s).forEach(e => e.style.outline = '3px solid hotpink'); return $$(s).length; };
const clr = () => document.querySelectorAll('*').forEach(e => e.style.outline = '');
```

> `$$()` is a DevTools shortcut for `document.querySelectorAll()` that returns a normal array. In real code you'd write `document.querySelectorAll('.card')`.

### The selector types

Run each line and watch the page:

```js
hl('h3')                          // TYPE: every <h3> element
hl('.card')                       // CLASS: every element with class="card"
hl('#roster')                     // ID: the one element with id="roster"
hl('[data-status="sleeping"]')    // ATTRIBUTE: elements whose data-status is "sleeping"
hl('.card h3')                    // DESCENDANT: <h3>s anywhere inside a .card
hl('nav > ul > li')               // CHILD: <li>s that are DIRECT children of nav's <ul>
```

What each one means:

- **Type** (`h3`) matches by tag name.
- **Class** (`.card`) matches by the `class` attribute. Elements can have several classes, like `class="card featured"`. Classes are the everyday workhorse of CSS.
- **ID** (`#roster`) matches by `id`. IDs should be unique on a page. You'll see in Part 3 why styling by ID can cause trouble.
- **Attribute** (`[data-status="sleeping"]`) matches any attribute. The `data-*` attributes in the HTML are custom data, and they're perfect for styling based on state. The page uses them for each monster's status and type.
- **Descendant**: a **space** between two selectors means "anywhere inside", at any depth.
- **Child**: `>` means "directly inside", exactly one level down.

### Combining selectors

| You write | It means |
|---|---|
| `.card.featured` (no space) | has **both** classes (AND) |
| `.card, .badge` (comma) | **either** one (OR) |
| `.card:hover` | a card the mouse is over (a *pseudo-class*, a state) |
| `.card:first-child` | a card that is the first child of its parent |
| `.card:not([data-type=ghost])` | cards that do **not** match the part in brackets |
| `.card:has(.low)` | cards that **contain** something matching `.low` |

### Challenge: Selector Golf

For each target, find the **shortest** selector that matches **exactly** those elements, and nothing else. Use `hl()` to test. The count it returns tells you whether you're right. Answers are hidden; click to reveal.

| # | Target | Count |
|---|---|---|
| 1 | All the monster cards | 6 |
| 2 | Only the sleeping monsters | 2 |
| 3 | Only the nav links | 4 |
| 4 | Every other card | 3 |
| 5 | Vampires that are sleeping | 1 |
| 6 | Monsters with low scare power | 2 |
| 7 | Every monster that isn't a vampire | 4 |
| 8 | The "Fears" line on each card | 6 |
| Bonus | The avatars of both vampires | 2 |
| Bonus 2 | Every other *lurking* monster | 2 |

<details>
<summary><strong>Answers</strong> (try first!)</summary>

1. `.card`
2. `[data-status=sleeping]`. For simple values, the quotes are optional.
3. `nav a`
4. `.card:nth-child(2n)`. **Trap:** `li:nth-child(2n)` matches 11 elements, because nav items and the "Fears" lines are `<li>`s too. The modern alternative is `:nth-child(2n of .card)`, which counts only elements that match `.card`.
5. `[data-type=vampire][data-status=sleeping]`. Two attribute selectors stacked with **no space** means AND.
6. `.card:has(.low)`. `:has()` lets you select a parent based on what it contains. That used to need JavaScript.
7. `.card:not([data-type=vampire])`
8. `.meta li:last-child`. **Trap:** `li:last-child` matches 8, because it also catches the last nav link and Mumbling Mort's whole card (the last `<li>` in the roster).
9. **Bonus:** `[data-type=vampire] img`, a descendant selector on top of an attribute selector.
10. **Bonus 2:** `:nth-child(odd of [data-status=lurking])`. Plain `:nth-child` counts every sibling. `of …` counts only the lurking ones, so you get Count Vladimir and Howling Hank.

</details>

> **Takeaway:** a selector that's too broad hits elements you didn't mean, just like an over-eager LINQ `Where`. Prefer selectors that say what you mean (`.meta li:last-child`) over ones that happen to work today.

Run `clr()` when you're done to remove the pink outlines.

---

## 3. Cascade and specificity

Sooner or later, two rules will target the same element with different values. CSS settles it with the **cascade**. It checks these in order:

1. **`!important`** wins over normal rules. (Use it rarely. See below.)
2. **Specificity**: the more specific selector wins.
3. **Source order**: if everything else is tied, the rule that comes **later** wins.

> **C# analogy:** the cascade works like ASP.NET Core configuration layering. Values in `appsettings.json` get overridden by `appsettings.Development.json`, and environment variables override both. Specificity decides which layer a rule belongs to.

### Calculating specificity

Count three things in the selector and write them as **(IDs . classes . elements)**:

| Selector | IDs | Classes, attributes, pseudo-classes | Elements | Specificity |
|---|---|---|---|---|
| `h3` | 0 | 0 | 1 | **0.0.1** |
| `.card h3` | 0 | 1 | 1 | **0.1.1** |
| `.roster .card.featured` | 0 | 3 | 0 | **0.3.0** |
| `#roster .card` | 1 | 1 | 0 | **1.1.0** |

Compare them **like version numbers**. `1.1.0` beats `0.3.0`, because a single ID outranks any number of classes. That's why experienced CSS authors avoid styling by ID: it's very hard to override later.

### Try it: predict, then check

For each round, paste both rules at the **bottom of `styles.css`**, decide which color you think wins, then save and refresh. Right-click the element and choose **Inspect**: DevTools shows the losing rule **crossed out** in the Styles pane. **Delete the rules before the next round.**

| Round | Rules | Look at |
|---|---|---|
| 1 | `.card h3 { color: blue }`<br>`h3 { color: red }` | any monster name |
| 2 | `#roster .card { border: 3px solid red }`<br>`.roster .card.featured { border: 3px solid green }` | Count Vladimir's card |
| 3 | `.badge { color: red }`<br>`.badge { color: green }` | any badge |
| 4 | `body { color: green }`<br>`* { color: purple }` | any monster name |
| 5 | `main h3 { color: red }`<br>`h3 { color: blue !important }` | any monster name |

<details>
<summary><strong>Answers</strong></summary>

1. **Blue.** 0.1.1 beats 0.0.1. Coming later doesn't help red.
2. **Red.** 1.1.0 beats 0.3.0. One ID beats three classes.
3. **Green.** The specificity is tied, so the later rule wins.
4. **Purple.** This one surprises people. `*` has zero specificity, but it matches the `<h3>` **directly**. The green comes from `body` only through **inheritance**, and an inherited value always loses to any rule that matches the element itself.
5. **Blue.** `!important` jumps the queue. Treat it like `#pragma warning disable`: legal, but every use deserves a code review comment.

</details>

### The modern fix: `@layer`

Paste this (and delete it afterwards):

```css
@layer base { #roster h3 { color: red } }
h3 { color: blue }
```

**Blue wins**, even though the red rule has an ID. Rules **outside** a layer beat rules **inside** one, whatever their specificity. Larger codebases declare layers such as `@layer reset, base, components, utilities;` so the override order is decided on purpose, instead of by specificity fights and `!important`.

> **Clean up:** make sure `styles.css` is back to just the header comment before you move on.

---

## 4. STEP 1: Variables and base styles

Now you start building for real. Everything from here on stays in `styles.css`.

### CSS variables (custom properties)

Any property whose name starts with `--` is a **variable**. You define it once and use it anywhere with `var()`:

```css
:root { --accent: #ff7a1a; }       /* define it */
h1    { color: var(--accent); }    /* use it */
```

`:root` means the `<html>` element, the top of the tree. Variables **inherit**, so anything defined there is available everywhere. Change the value once and every use updates. Think of them as constants, except you can still change them at runtime, which is exactly how dark mode will work.

### `light-dark()` and `color-scheme`

```css
--bg: light-dark(#f6f3f9, #120d18);
```

`light-dark()` holds **both** values: the light one first, the dark one second. The browser picks one based on the element's `color-scheme`. So to switch the whole page to dark, you change only `color-scheme`, not every color.

> **The old way:** write every color twice, once normally and again in a `body.dark { … }` or `@media (prefers-color-scheme: dark)` block. That's two lists to keep in sync. `light-dark()` keeps each light and dark pair side by side.

### Type this into `styles.css`

```css
/* ===== STEP 1: Variables + base ===== */
:root {
  color-scheme: light;    /* the "Lights out" button switches this to dark */

  /* light-dark(light value, dark value): each color is written once */
  --bg:      light-dark(#f6f3f9, #120d18);
  --surface: light-dark(#ffffff, #1d1526);
  --text:    light-dark(#22182e, #ece6f3);
  --muted:   light-dark(#6a5f78, #a89bb8);
  --border:  light-dark(#e0d8ea, #33263f);

  --accent:   #ff7a1a;    /* jack-o'-lantern orange */
  --lurking:  #3fa34d;
  --sleeping: #7a5cc7;
  --banished: #b23a48;
}

body.dark { color-scheme: dark; }

body {
  margin: 0;
  font-family: system-ui, "Segoe UI", sans-serif;
  line-height: 1.5;
  color: var(--text);
  background: var(--bg);
}

h1, h2, h3 { margin: 0 0 .5rem; line-height: 1.2; }
```

What the rest of it does:

- `font-family: system-ui, …` uses the operating system's own UI font (Segoe UI on Windows). The list is a **fallback chain**: if one font isn't available, the browser tries the next.
- `line-height: 1.5` sets the space between lines to 1.5× the font size. It has no unit, so it scales with the text.
- `margin: 0` on `body` removes the browser's default 8px border around the page.
- `.5rem` means half the **root font size** (usually 16px, so 8px). `rem` units scale if a user turns up their browser's font size, which is better for accessibility than fixed `px`.

### ✅ Checkpoint

Save and refresh. You should see a nicer font and a faint lavender background. It's still a plain document, and that's expected. Compare with `checkpoints/step-1.css` if anything looks off.

**Try it:** in the Console, run `document.body.classList.toggle('dark')`. The page already goes dark, because `light-dark()` is doing its job. Run it again to switch back.

---

## 5. STEP 2: The box model

### Everything is a box

Temporarily add this line to the bottom of `styles.css`:

```css
* { outline: 1px solid red; }
```

Save and refresh. Every element turns out to be a **rectangle**, including headings, list items, links and images. Layout in CSS is all about sizing and positioning those boxes. **Delete the line** when you've had a look.

Each box has four layers, from the inside out:

```
┌──────────── margin ────────────┐   space OUTSIDE the border (pushes neighbors away)
│ ┌────────── border ──────────┐ │   the visible edge
│ │ ┌──────── padding ───────┐ │ │   space INSIDE the border
│ │ │        content         │ │ │   the text or image itself
│ │ └────────────────────────┘ │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘
```

In DevTools, select any element. The **Computed** tab (or the bottom of the Styles pane) shows this diagram with real numbers.

### Why `box-sizing: border-box` matters

By default, `width: 200px` sets the width of the **content only**. Padding and border are added on top: a 200px box with 20px of padding is really **240px** wide. That surprises nearly everybody. With `box-sizing: border-box`, `width` includes the padding and border, so 200px means 200px. Almost every modern stylesheet starts by turning it on for every element.

### One variable per status

The cards need a color for each status: green for lurking, purple for sleeping, red for banished. That color shows up in three places (the card's edge, the badge and the avatar ring). Instead of writing nine rules, each status sets **one** variable:

```css
[data-status="lurking"] { --status: var(--lurking); }
```

Then the card, badge and avatar all use `var(--status, var(--muted))`. The second argument is a **fallback**, used if `--status` isn't set, much like `??` in C#. Because variables inherit, everything **inside** a card sees that card's `--status`. To add a new status later, you write one line.

### Type this into `styles.css` (below STEP 1)

```css
/* ===== STEP 2: Box model (cards become boxes) ===== */
*, *::before, *::after { box-sizing: border-box; }

main {
  max-width: 1000px;
  margin: 0 auto;
  padding: 1.5rem;
}

.roster, .meta {
  list-style: none;
  margin: 0;
  padding: 0;
}

/* Each status sets ONE variable... */
[data-status="lurking"]  { --status: var(--lurking); }
[data-status="sleeping"] { --status: var(--sleeping); }
[data-status="banished"] { --status: var(--banished); opacity: .7; }

/* ...and everything inside the card uses it (fallback: --muted) */
.card {
  padding: 1rem;
  margin-bottom: 1rem;
  background: var(--surface);
  border: 1px solid var(--border);
  border-left: 6px solid var(--status, var(--muted));
  border-radius: 10px;
}

.type  { margin: 0 0 .5rem; color: var(--muted); }
.meta  { margin-top: .75rem; font-size: .9rem; }
.low   { color: var(--banished); font-weight: 600; }

.badge {
  display: inline-block;
  padding: .1rem .6rem;
  border-radius: 999px;
  font-size: .75rem;
  font-weight: 600;
  color: #fff;
  background: var(--status, var(--muted));
}

.avatar {
  display: block;
  width: 72px;
  height: 72px;
  margin-bottom: .75rem;
  border-radius: 50%;              /* square box, round picture */
  border: 3px solid var(--status, var(--muted));
}

[data-status="banished"] .avatar { filter: grayscale(1); }
```

The notable lines:

- **`max-width: 1000px; margin: 0 auto`** stops the content getting too wide on big screens. `auto` left and right margins split the leftover space evenly, which centers the block.
- **`*::before, *::after`** are *pseudo-elements*, extra boxes CSS can add before or after an element's content. You'll use one in STEP 4.
- **`list-style: none; padding: 0`** removes the bullets and indentation that the browser's default stylesheet gives every `<ul>`.
- **`border: 1px solid …` followed by `border-left: 6px solid …`** is the cascade within a single rule. The later declaration overrides just the left side.
- **`display: inline-block`** on the badge lets it sit in a line of text like a word, but still accept padding like a box.
- **`border-radius: 999px`** is bigger than the badge could ever be tall, which guarantees perfect pill-shaped ends.
- **The avatar size:** in the HTML, each `<img>` has `width="120" height="120"`, which is why the avatars were large on the bare page. CSS beats those HTML attributes, so `width: 72px` wins. `border-radius: 50%` turns the square box into a circle. Try changing it to `20%` to see the difference.
- **`filter: grayscale(1)`** drains the color from banished monsters.

### ✅ Checkpoint

Each monster is now a card, stacked in one column, with a round avatar, a colored left edge and a status pill. Banished Barty Bones is faded and gray. Compare with `checkpoints/step-2.css` if anything looks off.

---

## 6. STEP 3: Layout with Flexbox and Grid

Modern CSS has two layout systems, and the rule of thumb for choosing is simple:

- **Flexbox** for **one direction**: a row *or* a column. Good for toolbars, navigation and button groups.
- **Grid** for **two dimensions**: rows *and* columns. Good for card layouts and whole-page structure.

### The header (Flexbox)

Build this one **a line at a time**, saving and refreshing after each, so you can watch what each line does.

**1.** Turn the header into a flex row:

```css
/* ===== STEP 3: Layout (Flexbox header, Grid cards) ===== */
#site-header {
  display: flex;
}
```

The title, nav and button jump into a single row. `display: flex` makes the element a **flex container**, and its **direct children** become flex items laid out along a row.

**2.** Fill in the rest of the header rule, then add the title rule:

```css
#site-header {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: .75rem 1.5rem;
  background: #2a1b3d;
  color: #fff;
  border-bottom: 4px solid var(--accent);
}

#site-header h1 { margin: 0; font-size: 1.25rem; }
```

- `align-items: center` centers the items **vertically**, along the cross axis.
- `gap` puts space **between** items. You don't need margins, or a special case for the last item.

**3.** Push the nav to the right:

```css
nav { margin-left: auto; }        /* auto margin soaks up free space */
```

Why does that work? Inside a flex container, an `auto` margin **takes up all the leftover space**. A big invisible margin opens up to the left of the nav, which pushes it and the button to the right edge.

**4.** Make the links horizontal and style them:

```css
nav ul {
  display: flex;
  gap: .25rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

nav a {
  display: block;
  padding: .4rem .8rem;
  border-radius: 6px;
  color: #d9cfe6;
  text-decoration: none;
}

nav a.active { background: rgb(255 255 255 / .12); color: #fff; }
```

- The `<ul>` is a flex container too, so its `<li>`s line up in a row. Flex containers can be nested freely.
- `display: block` on the links makes the whole padded area clickable, not just the text.
- `rgb(255 255 255 / .12)` is white at 12% opacity. On a dark header it shows up as a subtle highlight.

> **DevTools tip:** in the Elements panel, a small **flex** badge appears next to any flex container. Click it to draw an overlay showing the items and gaps.

### The cards (Grid)

**1.** Start with a fixed three-column grid:

```css
.roster {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}
```

`1fr` means "one **fr**action of the free space", so `repeat(3, 1fr)` gives three equal columns. Now make the browser window narrow: the three columns get squashed.

**2.** Make it responsive. Change the columns line to:

```css
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
```

Read it as: "fit as many columns as you can, each at least 240px wide and sharing any extra space equally." Resize the window now. The grid reflows from three columns to two to one **without a single media query**.

**3.** Tidy up and add a featured card:

```css
.card { margin-bottom: 0; }        /* grid gap handles spacing now */
.card.featured { grid-column: span 2; }
```

- STEP 2 gave cards a bottom margin to space them out. The grid's `gap` does that job now, so the margin goes. `gap` replaces the old approach of margins plus a `:last-child` exception.
- `grid-column: span 2` makes Count Vladimir's card (the one with `class="card featured"`) two columns wide.

> **DevTools tip:** a **grid** badge appears next to the roster in the Elements panel. Click it to see the column lines.

### ✅ Checkpoint

It looks like a real app: a dark header with the nav on the right, and a responsive grid of cards. Compare with `checkpoints/step-3.css` if anything looks off.

---

## 7. STEP 4: Making it feel alive

### First, the payoff: dark mode

Click **Lights out**. It works already, before you've written any STEP 4 code. Here's why. The button runs one line of JavaScript:

```js
document.body.classList.toggle('dark')
```

That adds or removes the `dark` class on `<body>`. The rule `body.dark { color-scheme: dark; }` from STEP 1 then flips every `light-dark()` color.

> **Key idea:** **JavaScript changes state (a class), and CSS decides what that state looks like.** This separation keeps UI code clean. Blazor components work the same way: you bind a class, and the stylesheet does the rest.

### Type this into `styles.css` (below STEP 3)

```css
/* ===== STEP 4: Make it feel alive ===== */
body { transition: background .3s, color .3s; }

nav a:hover { background: rgb(255 255 255 / .12); color: #fff; }

/* Keyboard focus ring: shows for Tab, not for mouse clicks */
:is(nav a, #theme-toggle):focus-visible {
  outline: 3px solid var(--accent);
  outline-offset: 2px;
}

#theme-toggle {
  font: inherit;
  padding: .5rem 1rem;
  border: 0;
  border-radius: 6px;
  cursor: pointer;
  color: #22182e;
  background: var(--accent);
  transition: scale .15s;
}

#theme-toggle:active { scale: .95; }

/* Individual transform properties: translate, rotate, scale */
.card { transition: translate .2s, box-shadow .2s; }

.card:hover {
  translate: 0 -4px;
  box-shadow: 0 8px 20px rgb(0 0 0 / .15);
}

/* Lurking monsters glow... */
[data-status="lurking"] .badge { animation: glow 1.6s ease-in-out infinite; }

@keyframes glow {
  0%, 100% { box-shadow: 0 0 0 0 color-mix(in srgb, var(--status) 60%, transparent); }
  50%      { box-shadow: 0 0 0 6px transparent; }
}

/* ...and sleeping monsters snore */
[data-status="sleeping"] .badge::after {
  content: " z";
  display: inline-block;
  animation: snore 2s ease-in-out infinite;
}

@keyframes snore {
  0%   { opacity: 0; translate: 0 0; }
  50%  { opacity: 1; }
  100% { opacity: 0; translate: 4px -6px; }
}

/* Avatars wobble when you hover the card */
.avatar { transition: rotate .3s, scale .3s; }
.card:hover .avatar { rotate: -8deg; scale: 1.08; }

/* Small screens: media query range syntax */
@media (width <= 560px) {
  .card.featured { grid-column: auto; }
  #site-header { flex-wrap: wrap; }
}

/* Respect the OS "reduce motion" setting */
@media (prefers-reduced-motion: reduce) {
  [data-status] .badge,
  [data-status] .badge::after { animation: none; }
  body, .card, .avatar        { transition: none; }
}
```

Save, refresh, and hover over some cards. Then read on to see what each part does.

### Transitions: animate between two states

```css
.card { transition: translate .2s, box-shadow .2s; }
.card:hover { translate: 0 -4px; … }
```

Without the `transition`, the card would **jump** 4px up on hover. With it, the browser animates the change over 0.2 seconds, in both directions. You put the `transition` on the **normal** state, not on `:hover`, so the card also animates back down when the mouse leaves.

The `body` transition is why the theme now **fades** between light and dark instead of snapping.

### Individual transform properties

`translate`, `rotate` and `scale` are three separate properties.

> **The old way:** `transform: translateY(-4px) rotate(-8deg) scale(1.08)`, one string holding several functions. Changing just one part meant rewriting the whole string, and you couldn't transition them separately. The separate properties work like separate properties on a C# object instead of one formatted string.

### Hover one element, style another

```css
.card:hover .avatar { rotate: -8deg; scale: 1.08; }
```

Read it right to left: "an `.avatar` inside a `.card` that is being hovered." You hover the **card** but style the **avatar**. That's very handy, and it's simply a descendant selector from Part 2.

### Keyframe animations: loop on their own

Transitions need a trigger, such as hover. **Keyframe animations** run by themselves:

```css
@keyframes glow {
  0%, 100% { box-shadow: 0 0 0 0 color-mix(…); }   /* start and end: tight glow */
  50%      { box-shadow: 0 0 0 6px transparent; }   /* halfway: spread out and faded */
}
[data-status="lurking"] .badge { animation: glow 1.6s ease-in-out infinite; }
```

`@keyframes` defines the steps, and `animation` applies them: name, duration, easing (how it speeds up and slows down) and how many times (`infinite`).

- **`color-mix(in srgb, var(--status) 60%, transparent)`** takes the status color at 60% strength. It reuses the variable instead of hard-coding an `rgb()` value.
- **The snore** uses `::after`, a **pseudo-element**. `content: " z"` makes CSS insert a "z" after the badge text that **isn't in the HTML at all**. The animation then floats it up and fades it out.

### `:focus-visible`: focus styles for keyboard users

Click in the page, then press **Tab** a few times. An orange ring moves through the nav links and the button. Now **click** a link with the mouse: no ring.

> **The old way:** `:focus` showed the ring on mouse clicks too, and many sites removed focus styles completely because they looked odd. That made those sites unusable without a mouse. `:focus-visible` shows the ring only when the browser decides it's useful, which mostly means keyboard navigation.

`:is(nav a, #theme-toggle)` is shorthand. It means the same as writing `nav a:focus-visible, #theme-toggle:focus-visible`.

### Media query range syntax

```css
@media (width <= 560px) { … }
```

The rules inside apply only when the window is 560px wide or less. On small screens, the featured card stops spanning two columns and the header is allowed to wrap onto a second line.

> **The old way:** `@media (max-width: 560px)`. The new form reads like a normal comparison.

### Respecting `prefers-reduced-motion`

Some people get dizzy or feel ill from motion on screen, so operating systems have a "reduce motion" setting. `@media (prefers-reduced-motion: reduce)` lets your CSS honor it. Here it switches off the looping animations and the transitions.

**Test it:** in DevTools, press **Ctrl+Shift+P**, type **"reduced motion"**, and choose **Emulate CSS prefers-reduced-motion: reduce**. The glow and snore stop.

Notice that the block doesn't need `!important`. Its selectors have the same specificity as the original rules, and they come **later** in the file, so source order wins. That's Part 3 in action.

### ✅ Checkpoint

You're done. Your page should now match `monsters-finished.html`:

- Cards lift and avatars wobble on hover.
- Lurking badges glow and sleeping badges snore.
- The theme fades between light and dark.
- Tab shows focus rings.

If anything looks off, compare with `checkpoints/step-4.css`.

---

## 8. Your turn: a solo Style-Off

In the live session, pairs restyle the page from scratch in 7 minutes. Try it yourself:

1. Copy your `styles.css` somewhere safe, for example `styles-mine.css`.
2. Empty `styles.css`.
3. Set a **10-minute** timer and give the page a completely different look. **CSS only, and no HTML changes.**

Some ideas, if you're stuck:

- **Retro terminal:** black background, green monospace text, and `text-shadow` for glow.
- **Haunted parchment:** a sepia `--bg`, a serif font, and cards slightly rotated (`rotate: -1deg`), with alternate cards rotated the other way.
- **Trading cards:** a tall `aspect-ratio`, a big centered avatar, and a status-colored header bar.
- **Something chaotic:** go wild with `:nth-child()` and `:hover`.

Then think about this question, which is where the live session hands over to the design review discussion. Everyone in the room started from **the same HTML** and ended up with completely different results. If each of those landed as a pull request with no agreed design up front, which one would you merge, and how much rework would it take? **Agreeing on the design before building saves rework.**

---

## 9. How this maps to Blazor, and what's next

### In our Blazor code

- **Scoped CSS:** a file named `Component.razor.css` applies **only** to that component. At build time, Blazor adds a unique attribute to the component's elements (something like `b-x7k2abc`) and rewrites your selectors to match, for example `h3[b-x7k2abc]`. Styles can't leak into other components. Inspect a Blazor component in DevTools to see the attributes.
- **`::deep`:** scoped styles normally stop at the component's edge. Prefixing a selector with `::deep` lets a parent's styles reach into its child components.
- **Design tokens:** put shared variables (colors, spacing) in `app.css` under `:root`, just like STEP 1. Every component can then use `var(--accent)`.
- **State through classes:** binding a class in Razor (`class="card @(IsActive ? "active" : "")"`) is the same pattern as the Lights out button. C# changes the state, and CSS decides how it looks.

### Good next topics

- **Native CSS nesting:** write `.card { & h3 { … } }` in plain CSS, with no Sass needed.
- **`@scope`:** limit styles to part of the page without long class names.
- **Container queries** (`@container`): components that respond to the size of **their container**, not the whole window. Ideal for reusable components.
- **[modern-css.com](https://modern-css.com/category/css/)**: side-by-side comparisons of old and new ways to do common things.

---

## 10. Troubleshooting

| Problem | Fix |
|---|---|
| I saved, but nothing changed | Did you **refresh**? If so, try a hard refresh (**Ctrl+F5**) to bypass the cache. Check that you edited `styles.css` in the same folder as `monsters-start.html`. |
| One rule doesn't work | Inspect the element. If the rule is **crossed out**, another rule is winning (Part 3). If it has a **yellow warning triangle**, there's a typo in the property or value. Look for a missing `;` or `}` just above it. |
| Everything below a certain line stopped working | A missing `}` usually breaks everything after it. Check that your braces are balanced. |
| The avatars don't show | The `images` folder must sit next to `monsters-start.html`. |
| Dark mode doesn't change the colors | `light-dark()` needs a 2024-or-later browser. Update Chrome or Edge. |
| I'm hopelessly lost | Copy the matching file from `checkpoints/` over `styles.css` and carry on from there. |

---

## 11. Quick reference

### Selectors

| Syntax | Meaning |
|---|---|
| `h3` | type |
| `.card` | class |
| `#roster` | id |
| `[data-status=sleeping]` | attribute |
| `a b` | `b` anywhere inside `a` |
| `a > b` | `b` that is a direct child of `a` |
| `a + b` | `b` immediately after `a` |
| `a ~ b` | `b` anywhere after `a`, among its siblings |
| `.a.b` | has both `a` and `b` (AND) |
| `.a, .b` | either one (OR) |
| `:is(a, b) c` | shorthand for `a c, b c` |
| `:hover` `:focus-visible` | interaction state |
| `:first-child` `:nth-child(2n)` | position |
| `:nth-child(2n of .card)` | position, counting only `.card`s |
| `:not(x)` | doesn't match `x` |
| `:has(x)` | contains `x` |

### The cascade, in order

1. `!important`
2. Layers: rules outside `@layer` beat rules inside one.
3. Specificity (IDs . classes . elements), compared like version numbers.
4. Source order: the later rule wins.
5. Inherited values lose to any rule that matches the element directly.

### Box model and layout

- Box layers: content → padding → border → margin. Always set `*, *::before, *::after { box-sizing: border-box; }`.
- **Flexbox**, for one direction: `display: flex; align-items: center; gap: …`. Use `margin-left: auto` to push items to the end.
- **Grid**, for two dimensions: `display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: …`.

### Modern habits

| Instead of | Use |
|---|---|
| Repeating colors for dark mode | `light-dark(light, dark)` + `color-scheme` |
| `transform: translateY() rotate() scale()` | `translate`, `rotate`, `scale` |
| `@media (max-width: 560px)` | `@media (width <= 560px)` |
| `:focus` | `:focus-visible` |
| Animations that always run | `@media (prefers-reduced-motion: reduce)` |
| Hard-coded tints | `color-mix(in srgb, var(--x) 60%, transparent)` |
| `!important` to win a fight | `@layer` |
| Margins plus `:last-child` exceptions | `gap` |

### DevTools

| Command | What it does |
|---|---|
| `$0` | the selected element |
| `$$('sel')` | all matches, as an array |
| `getComputedStyle($0).prop` | the value the browser actually applied |
| `document.designMode = "on"` | makes the whole page editable |
| Ctrl+Shift+P → "Rendering" / "reduced motion" | emulate dark mode, print or reduced motion |
| The **flex** and **grid** badges in the Elements panel | layout overlays |
