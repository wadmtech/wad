# TechNova 2026 – HTML/CSS Exam Prep Kit

*Practice papers, a 10-minute model solution, and a cheat sheet for Flexbox, Grid and other styling.*

## 1. Honest strategy

Building all 7 pages from scratch in 10 minutes is not realistic. In a timed test you will most likely be asked for **1–2 pages**, or the full site where pages are copy-paste variants of one template. So this kit gives you:

- **Paper A**: one combined page, 10 minutes (covers everything the 7 pages test).
- **Drills B, C, D**: three micro-papers that add up to 10 minutes, for isolated practice.
- **Paper E**: a speed plan if you must deliver all 7 pages.

Ask your examiner whether CSS must be **internal** (`<style>`) or **external** (`style.css`). The model solution uses internal CSS because it is fastest; to go external, move the CSS to `style.css` and add `<link rel='stylesheet' href='style.css'>`.

## 2. Practice papers

### Paper A: Combined single page (10 min, 25 marks)

Create `technova.html` for **TechNova 2026** (Theme: *Innovate. Integrate. Inspire.*; 12–13 March 2026; Main Auditorium).

| # | Requirement | Marks |
| --- | --- | --- |
| 1 | Logo, navigation bar with 5 links (flex, hover effect), banner image | 4 |
| 2 | Title, theme, date/venue, short intro | 3 |
| 3 | Highlights: 500+ Participants, 15 Events, Cash Prizes, Industry Experts, as 4 boxes in a row (flex, wraps on small screens) | 4 |
| 4 | 4 technical event cards in a responsive grid, each with image, description, rules (list), prize | 5 |
| 5 | Schedule table with 6 rows (Time, Event, Venue), styled header, zebra stripes | 4 |
| 6 | Registration form: name, college, department (dropdown), year (radio), email, phone, technical event (dropdown), non-technical events (checkbox), address (textarea), comments, Submit and Reset | 5 |

**Time split:** skeleton 1 min, CSS 3 min, header/highlights/cards 3 min, table 1 min, form 2 min.

### Drill B: Registration form (4 min)

All fields from the Registration page, grouped using `<fieldset>` and `<legend>` into Personal, Contact, Event Selection and Additional Information. Use `required`, `type='email'`, `type='tel'` with `pattern='[0-9]{10}'`, and `placeholder`.

### Drill C: Schedule table (2 min)

The 6-row timetable with `<thead>`/`<tbody>`, a `<caption>`, bordered cells, colored header, zebra rows and a row hover highlight. **Extra:** add a lunch row with `colspan='3'`.

### Drill D: Event cards with Grid (4 min)

Four non-technical event cards (Photography, Quiz, Treasure Hunt, Gaming), each with a heading, description, rules list, fee and prize. Use `display:grid` with 2 columns on desktop and 1 column on mobile through a `@media` query. Add a hover lift (`transform`, `transition`).

### Paper E: All 7 pages: speed plan

1. **Minute 0–2:** write one template with `<head>`, `<nav>` (identical on every page), `<main>` and `<footer>`, plus the full CSS. Save it as `style.css`.
2. Copy the template 6 times and rename the files: `index`, `about`, `technical`, `nontechnical`, `schedule`, `register`, `contact`.
3. Fill in `<main>` per page (about 1 min each). Reuse the card, table and form snippets from Paper A.
4. Mark the current page's link with `class='active'`.
5. Check that **every nav link works**. Broken links lose marks.

## 3. Model solution for Paper A

```html
<!DOCTYPE html>
<html lang='en'>
<head>
<meta charset='UTF-8'>
<meta name='viewport' content='width=device-width, initial-scale=1'>
<title>TechNova 2026</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:Arial,sans-serif;line-height:1.6;background:#f4f6fb;color:#222}
nav{display:flex;align-items:center;justify-content:center;gap:24px;background:#1a237e;padding:10px}
nav img{height:40px}
nav a{color:#fff;text-decoration:none;padding:6px 10px;border-radius:4px}
nav a:hover{background:#ffca28;color:#000}
header{text-align:center;background:#283593;color:#fff;padding:20px}
header img.banner{width:100%;max-height:220px;object-fit:cover;border-radius:8px}
h2{text-align:center;margin:20px 0 10px}
.highlights{display:flex;flex-wrap:wrap;gap:15px;padding:15px;justify-content:center}
.highlights div{flex:1 1 180px;background:#fff;padding:20px;text-align:center;border-radius:8px;box-shadow:0 2px 6px rgba(0,0,0,.2)}
.highlights b{display:block;font-size:1.6em;color:#1a237e}
.events{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:20px;padding:20px}
.card{background:#fff;padding:15px;border-radius:8px;box-shadow:0 2px 6px rgba(0,0,0,.2);transition:transform .3s}
.card:hover{transform:translateY(-6px)}
.card img{width:100%;height:130px;object-fit:cover;border-radius:6px}
.card ul{margin-left:20px}
table{border-collapse:collapse;width:90%;margin:15px auto;background:#fff}
th,td{border:1px solid #999;padding:8px;text-align:left}
th{background:#1a237e;color:#fff}
tr:nth-child(even){background:#e8eaf6}
tr:hover{background:#fff59d}
form{max-width:520px;margin:20px auto;background:#fff;padding:20px;border-radius:8px}
fieldset{margin-bottom:15px;padding:10px}
label{display:block;margin-top:8px}
input[type=text],input[type=email],input[type=tel],select,textarea{width:100%;padding:8px;margin-top:4px}
button{padding:10px 20px;border:0;border-radius:5px;background:#1a237e;color:#fff;cursor:pointer}
button[type=reset]{background:#c62828}
@media(max-width:600px){nav{flex-wrap:wrap}}
</style>
</head>
<body>

<nav>
  <img src='logo.png' alt='College Logo'>
  <a href='#home'>Home</a><a href='#events'>Events</a>
  <a href='#schedule'>Schedule</a><a href='#register'>Register</a>
  <a href='#contact'>Contact</a>
</nav>

<header id='home'>
  <img class='banner' src='banner.jpg' alt='TechNova banner'>
  <h1>TechNova 2026</h1>
  <h3>Theme: Innovate. Integrate. Inspire.</h3>
  <p>12–13 March 2026 | Main Auditorium</p>
  <p>A national-level technical symposium for budding engineers.</p>
</header>

<section class='highlights'>
  <div><b>500+</b>Participants</div>
  <div><b>15</b>Events</div>
  <div><b>₹</b>Cash Prizes</div>
  <div><b>10+</b>Industry Experts</div>
</section>

<h2 id='events'>Technical Events</h2>
<section class='events'>
  <article class='card'>
    <img src='paper.jpg' alt='Paper Presentation'>
    <h3>Paper Presentation</h3>
    <p>Present innovative research ideas.</p>
    <ul><li>Team of max 2</li><li>8 minutes per team</li></ul>
    <p><strong>Prize:</strong> ₹5000</p>
  </article>
  <article class='card'>
    <img src='web.jpg' alt='Web Design'>
    <h3>Web Design Contest</h3>
    <p>Build attractive websites.</p>
    <ul><li>HTML and CSS only</li><li>60 minutes</li></ul>
    <p><strong>Prize:</strong> ₹4000</p>
  </article>
  <article class='card'>
    <img src='code.jpg' alt='Coding'>
    <h3>Coding Challenge</h3>
    <p>Solve programming problems.</p>
    <ul><li>Individual</li><li>No internet</li></ul>
    <p><strong>Prize:</strong> ₹5000</p>
  </article>
  <article class='card'>
    <img src='expo.jpg' alt='Project Expo'>
    <h3>Project Expo</h3>
    <p>Showcase innovative projects.</p>
    <ul><li>Team of max 4</li><li>Working model</li></ul>
    <p><strong>Prize:</strong> ₹6000</p>
  </article>
</section>

<h2 id='schedule'>Event Schedule</h2>
<table>
  <caption>Day 1 Timetable</caption>
  <thead><tr><th>Time</th><th>Event</th><th>Venue</th></tr></thead>
  <tbody>
    <tr><td>9:00 AM</td><td>Inauguration</td><td>Auditorium</td></tr>
    <tr><td>10:00 AM</td><td>Paper Presentation</td><td>Seminar Hall</td></tr>
    <tr><td>11:30 AM</td><td>Coding Challenge</td><td>Lab 1</td></tr>
    <tr><td>1:30 PM</td><td>Quiz Competition</td><td>Hall A</td></tr>
    <tr><td>3:00 PM</td><td>Project Expo</td><td>Exhibition Hall</td></tr>
    <tr><td>4:30 PM</td><td>Prize Distribution</td><td>Auditorium</td></tr>
  </tbody>
</table>

<h2 id='register'>Registration</h2>
<form action='#' method='post'>
  <fieldset><legend>Personal Information</legend>
    <label>Full Name <input type='text' name='name' required></label>
    <label>College Name <input type='text' name='college' required></label>
    <label>Department
      <select name='dept'><option>CSE</option><option>IT</option><option>ECE</option><option>EEE</option><option>MECH</option></select>
    </label>
    <label>Year of Study</label>
    <input type='radio' name='year' value='1' checked> I
    <input type='radio' name='year' value='2'> II
    <input type='radio' name='year' value='3'> III
    <input type='radio' name='year' value='4'> IV
  </fieldset>
  <fieldset><legend>Contact Information</legend>
    <label>Email <input type='email' name='email' required></label>
    <label>Phone <input type='tel' name='phone' pattern='[0-9]{10}' placeholder='10-digit number' required></label>
  </fieldset>
  <fieldset><legend>Event Selection</legend>
    <label>Technical Event
      <select name='tech'><option>Paper Presentation</option><option>Web Design Contest</option><option>Coding Challenge</option><option>Project Expo</option></select>
    </label>
    <label>Non-Technical Events</label>
    <input type='checkbox' name='nt' value='photo'> Photography
    <input type='checkbox' name='nt' value='quiz'> Quiz
    <input type='checkbox' name='nt' value='hunt'> Treasure Hunt
    <input type='checkbox' name='nt' value='game'> Gaming
  </fieldset>
  <fieldset><legend>Additional Information</legend>
    <label>Address <textarea name='address' rows='3'></textarea></label>
    <label>Comments <textarea name='comments' rows='3'></textarea></label>
  </fieldset>
  <button type='submit'>Submit</button>
  <button type='reset'>Reset</button>
</form>

<footer id='contact' style='text-align:center;padding:15px;background:#1a237e;color:#fff'>
  <p>Coordinator: Dr. A. Kumar | techNova@college.edu | 98765 43210</p>
</footer>

</body>
</html>
```

## 4. Cheat sheet

### 4.1 Page skeleton and semantic tags

```html
<!DOCTYPE html>
<html lang='en'>
<head>
  <meta charset='UTF-8'>
  <meta name='viewport' content='width=device-width, initial-scale=1'>
  <title>Page Title</title>
  <link rel='stylesheet' href='style.css'>
</head>
<body> ... </body>
</html>
```

| Tag | Use |
| --- | --- |
| `<header> <nav> <main> <section> <article> <aside> <footer>` | Semantic page structure |
| `<h1>`–`<h6>`, `<p>`, `<br>`, `<hr>` | Text and breaks |
| `<ul>/<ol>/<li>`, `<dl><dt><dd>` | Lists |
| `<a href='page.html'>` | Link. Add `target='_blank'` for a new tab, `href='#id'` for a section jump, `href='mailto:x@y.com'`, `href='tel:+91...'` |
| `<img src alt width height>` | Image. `alt` is always required |
| `<strong> <em> <mark> <small>` | Inline emphasis |
| `<div>` and `<span>` | Generic block and inline containers |

### 4.2 Tables

```html
<table>
  <caption>Title</caption>
  <thead><tr><th>Time</th><th colspan='2'>Details</th></tr></thead>
  <tbody><tr><td rowspan='2'>9 AM</td><td>A</td><td>B</td></tr></tbody>
  <tfoot>...</tfoot>
</table>
```

CSS: `border-collapse:collapse` (removes the double borders), `tr:nth-child(even){}` for zebra stripes, `tr:hover{}`.

### 4.3 Forms

| Element | Notes |
| --- | --- |
| `<form action='' method='post'>` | Wrapper |
| `<input type='text \| email \| tel \| password \| number \| date \| url'>` | Text-like fields |
| `type='radio'` | Same `name` means one choice |
| `type='checkbox'` | Multiple choices |
| `<select><option>` | Dropdown. Use `<optgroup>` to group |
| `<textarea rows cols>` | Multi-line text |
| `<button type='submit \| reset'>` or `<input type='submit \| reset'>` | Buttons |
| Attributes: `required`, `placeholder`, `pattern`, `min`, `max`, `maxlength`, `value`, `checked`, `disabled`, `readonly` | Validation and defaults |
| `<label for='id'>` with `id` on the input | Links label to field (or wrap the input inside the label) |
| `<fieldset><legend>` | Groups related fields |

### 4.4 CSS essentials

**Where:** inline (`style=''`), internal (`<style>`), external (`<link>`). Priority: inline > ID > class > tag.

**Selectors:** `p`, `.class`, `#id`, `div p` (descendant), `div > p` (child), `h1,h2` (group), `a:hover`, `li:first-child`, `tr:nth-child(even)`, `input[type=text]`, `p::first-letter`.

**Box model:** content, then padding, border, margin. Always start with:

```css
*{margin:0;padding:0;box-sizing:border-box}
```

**Quick properties:**

| Goal | CSS |
| --- | --- |
| Center text | `text-align:center` |
| Center a block | `margin:0 auto; width:80%` |
| Rounded corners | `border-radius:8px` |
| Shadow | `box-shadow:0 2px 6px rgba(0,0,0,.2)` |
| Background image | `background:url('b.jpg') center/cover no-repeat` |
| Image fit | `width:100%; height:150px; object-fit:cover` |
| Gradient | `background:linear-gradient(to right,#1a237e,#3949ab)` |
| Remove link underline | `text-decoration:none` |
| Remove bullets | `list-style:none` |
| Smooth hover | `transition:.3s` plus `transform:scale(1.05)` on `:hover` |
| Units | `px` fixed, `%` relative to parent, `rem`/`em` relative to font size, `vw`/`vh` relative to viewport |
| Display | `block`, `inline`, `inline-block`, `flex`, `grid`, `none` |

**Positioning:** `static` (default), `relative` (offset from its own spot), `absolute` (relative to the nearest positioned ancestor), `fixed` (stays on screen, e.g. a sticky navbar with `top:0;width:100%`), `sticky` (sticks after scrolling to its threshold).

### 4.5 Flexbox (1-D: rows or columns)

Apply to the **container**:

```css
.row{display:flex;
  flex-direction:row;            /* row | column | row-reverse */
  justify-content:space-between; /* main axis: flex-start | center | space-around | space-evenly */
  align-items:center;            /* cross axis: stretch | flex-start | flex-end */
  flex-wrap:wrap;                /* allow wrapping */
  gap:20px;}
```

Apply to **children:**

| Property | Meaning |
| --- | --- |
| `flex:1` | Grow equally to share space |
| `flex:1 1 200px` | grow, shrink, basis. This makes a wrapping responsive card row |
| `align-self:flex-end` | Override alignment for one item |
| `order:2` | Reorder |

**Use for:** navbars, highlight boxes, centering, footers.

**Perfect centering:** `display:flex; justify-content:center; align-items:center; height:100vh;`

### 4.6 Grid (2-D: rows and columns)

```css
.grid{display:grid;
  grid-template-columns:repeat(3,1fr);   /* 3 equal columns */
  /* responsive without media query: */
  grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
  gap:20px;}
```

| Property | Meaning |
| --- | --- |
| `grid-template-columns:200px 1fr 1fr` | Mixed fixed and flexible columns |
| `grid-column:span 2` | Item spans 2 columns |
| `grid-row:span 2` | Item spans 2 rows |
| `justify-items`, `align-items`, `place-items:center` | Align content inside cells |
| `grid-template-areas` | Named layout (see below) |

**Classic page layout:**

```css
.page{display:grid;grid-template-columns:200px 1fr;
  grid-template-areas:'head head' 'side main' 'foot foot';}
header{grid-area:head} aside{grid-area:side} main{grid-area:main} footer{grid-area:foot}
```

**Flex vs Grid:** one direction (nav, row of boxes) → **Flex**. Rows and columns together (card gallery, page layout) → **Grid**.

### 4.7 Responsive design

```css
@media (max-width:600px){
  .grid{grid-template-columns:1fr}
  nav{flex-direction:column}
}
```

Always include the viewport meta tag. Use `max-width:100%` on images.

### 4.8 Common mistakes that lose marks

- Missing `alt` on images, or broken image paths (keep images in the same folder).
- Forgetting `name` on radio buttons, so they do not group.
- Forgetting `border-collapse:collapse` on tables.
- Navigation links that do not match file names exactly (case-sensitive).
- Using `<br>` for spacing instead of `margin`/`padding`.
- Mixing `height:100%` without a parent height. Use `min-height` or `vh`.
- Not saving as `.html`, or opening the wrong file.

### 4.9 Last-minute checklist

1. Tab title, logo, banner present.
2. Nav on every page with working links.
3. Every required field in the form, plus both Submit and Reset buttons.
4. Table has a header row and borders.
5. At least one use of flex, one of grid, one `:hover`, and one `@media` query (examiners like to see these).
6. Open it in the browser and resize the window once.
