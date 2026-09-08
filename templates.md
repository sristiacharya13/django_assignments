# Django Assignment — Templates & Static Files (Spotify)

In this section, you'll work on Django Templates, and then learn to serve static assets like CSS and images.

Keep the styling **simple** — internal CSS only for now (we'll move to external stylesheets in the Static section). The goal here is to *understand how templates work*, not to build a pixel-perfect UI.

---

## Section 1: Templates

### Q1. Basic Template Rendering
Before building anything fancy, let's confirm templates are wired up correctly.

- Create a template called `home.html` that renders at `/` (your project's landing page).
- Add a basic HTML skeleton with:
  - A `<h1>` heading — something like `Welcome to Spotify`
  - A list of `<a>` tags, one for each app you're building — **Artist**, **Playlist**, **Podcast** — that link to those apps' pages (a simple `<ul>` of `<li><a href="...">Artist</a></li>` is fine)
- Hook this template up to a view using `render()`.
- Confirm it renders correctly at `/`, and that clicking each link actually takes you to that app's page.

**Goal:** Make sure your `TEMPLATES` setting, folder structure, `render()` call, and basic navigation between apps are all working before moving forward.

---

### Q2. Clickable Divs (App Cards)
Links in a list work, but Spotify's home screen shows apps/sections as clickable **cards**, not a plain list.

- On the same `home.html`, below your `<h1>`, add a `<div>` for each app (Artist, Playlist, Podcast) — think of these as "cards" a user would click to enter that section.
- Each `<div>` should:
  - Display the app name inside it
  - Be wrapped in an `<a>` tag (so the whole card is clickable), that navigates to the app's URL.
- Add minimal internal CSS to make it obvious these are clickable — a border, some padding, and a `cursor: pointer` is enough.

**Goal:** Practice structuring clickable UI blocks with `<div>`, and see two different ways (`<a>` list vs. styled `<div>` cards) of achieving navigation.

---

### Q3. Project-Level Template Inheritance (Base Template with Navbar)
Now let's avoid repeating the same HTML structure (`<html>`, `<head>`, `<body>`) on every page.

- Create a `base.html` at the **project level** (a `templates/` folder configured in `DIRS`, not inside any specific app).
- This `base.html` should include:
  - The common HTML skeleton (`<html>`, `<head>`, `<body>`)
  - A **navbar** with links to different sections of your app — e.g. `Home`, `Playlists`, `Artists`, `Podcasts` (use whichever apps/pages you've planned)
  - A `{% block content %}{% endblock %}` block where child templates will inject their page-specific content
- Update your `home.html` (from Q1) to **extend** `base.html` using `{% extends %}` and fill in the `{% block content %}`.

**Goal:** Understand `{% extends %}` and `{% block %}`, and see how a shared layout works across pages.

---

### Q4. App-Level Template Inheritance (Each App, Its Own Vibe)
Every good app on Spotify has its own personality — let's reflect that with color.

- For each of your apps (e.g. `artists`, `podcasts`, `playlists`), create an **app-level base template** (e.g. `artists/artists.html`) that:
  - Extends the **project-level** `base.html`
  - Overrides the `content` block
  - Adds a distinct **background color** for that app (pick any color per app — Artists could be dark purple, Albums could be teal, etc.)

**Goal:** Understand template inheritance — project → app — and how each level can override or add to the one above it.

---

### Q5. Template Filters
Time to manipulate data directly inside templates.

- In one of your views, pass a list of songs or artists (a list of dictionaries) to the template context.
- In the corresponding template, use **at least 4 different template filters**, for example:
  - `{{ song.title|upper }}` or `|lower`
  - `{{ song.title|truncatewords:3 }}`
  - `{{ artist.name|title }}`
  - `{{ song_list|length }}`
  - `{{ song.release_date|date:"F Y" }}` (pass a date)
  - `{{ artist.bio|default:"No bio available" }}`
- Display each filtered value clearly labeled, so it's easy to see what each filter is doing.

**Goal:** Get comfortable transforming data for display without writing that logic in your view.

---

### Q6. URL Tags in Templates
Hardcoded URLs are a trap — let's avoid that.

- In your `urls.py`, make sure your paths have `name=` attributes (e.g. `path('artist/', views.artist_detail, name='artist-detail')`).
- In your navbar (from Q3) and anywhere else you link between pages, replace hardcoded links like `href="/artist/"` with `{% url 'artist-detail' %}`.

**Goal:** Learn why `{% url %}` tags are the correct way to link pages — your links won't break even if you change the actual URL path later.

---

### Q7. Loops + Conditionals in Templates
Let's combine everything so far into one dynamic page, using real playlist data.

In your view, pass this exact list to the template context as `playlist`:

```python
playlist = [
    {'title': 'BbY WOW', 'plays': 33059939, 'album': 'NO ME ARREPIENTO DE SENTIR TANTO'},
    {'title': 'Beauty And A Beat', 'plays': 23060351, 'album': 'Believe'},
    {'title': 'Earrings', 'plays': 22481881, 'album': 'Sweet Boy'},
    {'title': 'Loser', 'plays': 22170033, 'album': 'Deadbeat'},
    {'title': 'The One That Got Away', 'plays': 22169158, 'album': 'Teenage Dream'},
    {'title': 'Dai Dai', 'plays': 21507155, 'album': 'Dai Dai'},
    {'title': 'Self Aware', 'plays': 20891173, 'album': 'Self Aware'},
    {'title': 'the cure', 'plays': 19826357, 'album': 'you seem pretty sad for a girl so in love'},
    {'title': 'Babydoll', 'plays': 19450905, 'album': "Don't Forget About Me, Demos"},
    {'title': 'back to friends', 'plays': 19424543, 'album': 'I Barely Know Her'},
    {'title': 'Billie Jean', 'plays': 19044517, 'album': 'Thriller'}
]
```

Create a `playlist_detail.html` page and:

**a) Basic loop**
- Use a `{% for %}` loop to display each song's `title`, `plays`, and `album` — a simple list is fine.

**b) `forloop.first` check**
- Inside the same loop, use `{% if forloop.first %}` to show "Now Playing" next to the very first song, and just show the song details for the rest.

**c) if / elif / else on play count**
- Still inside the loop, add a second conditional that labels each song based on its `plays` count using `{% if %}` / `{% elif %}` / `{% else %}`:
  - `plays > 25000000` → show `Trending`
  - `plays > 20000000` → show `Popular`
  - anything else → show `Regular`

**d) Empty state**
- Use `{% empty %}` to show `"No songs added yet. Start building your playlist!"` in case the `playlist` list is ever empty (test this by temporarily passing an empty list from your view).

**Goal:** Practice control-flow template tags (`{% if %}`, `{% elif %}`, `{% else %}`, `{% empty %}`) alongside the special `forloop` variable — all commonly used together in real projects.

---

## Section 2: Static Files

### Q1. Move Internal CSS to a Static File
Let's clean up the styling you added earlier.

- Create a `static/` folder (either app-level or project-level — your choice, based on what you set up in `settings.py`).
- Move the internal `<style>` CSS you wrote in Q1–Q4 above into a `.css` file inside `static/`.
- Load it in your `base.html` using `{% load static %}` and `<link rel="stylesheet" href="{% static 'css/style.css' %}">`.
- Confirm the styling still works exactly as before, just now coming from a static file.

**Goal:** Understand the difference between internal CSS and static files, and how Django locates static assets via `STATICFILES_DIRS` / `STATIC_URL`.

---

### Q2. Add a Logo Image
Every good clone needs a logo.

- Add a Spotify-style logo image into your `static/images/` folder.
- Display it in your navbar (from Q3 in Templates) using the `{% static %}` tag:
  ```html
  <img src="{% static 'images/logo.png' %}" alt="App Logo">
  ```
- Make sure it renders correctly in your navbar links.

**Goal:** Practice serving image assets the same way as CSS — through the static files system.

---
