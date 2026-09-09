# Django Assignment — Forms (Spotify)

In this section, you'll work with Django's `forms.Form` class. The goal is to understand how Forms work on their own: rendering, handling GET/POST, validation, cleaning, and widgets.

Focus on the Python and template logic.

---

## Section 1: Forms

### Q1. Your First Form Class — Create Playlist
Let's start with the most basic form possible.

- Create a `forms.py` in your `playlist` app.
- Define a `PlaylistForm(forms.Form)` with two fields:
  - `title` — a `CharField`
  - `description` — a `CharField` with `required=False` (using `widget=forms.Textarea`)
- In your view, create an instance of this form and pass it to a template.
- In the template, render the form using `{{ form }}` inside a `<form>` tag with a submit button.
- For now, just make sure the form **displays correctly** at a URL like `/playlist/create/` — don't worry about handling the submission yet.

**Goal:** Understand that a Django Form is just a Python class with fields, independent of any database model.

---

### Q2. GET vs POST — Handling the Submission
Now let's actually process what the user submits, and understand why forms need to check the request method.

- Update your view for `/playlist/create/` so that:
  - On a **GET** request → show a blank `PlaylistForm`
  - On a **POST** request → create a `PlaylistForm(request.POST)`, and for now, just print `request.POST` display it on the page so you can see the submitted data
- Add `<form method="POST">` in your template, along with `{% csrf_token %}`.
- Test both scenarios:
  - Visiting the page directly (GET) — form should be empty
  - Submitting the form (POST) — you should see your submitted data reflected on the page.

**Goal:** Understand the fundamental GET vs POST pattern that almost every Django form view follows — `if request.method == 'POST':` ... `else:` show blank form.

---

### Q3. Built-in Form Field Classes — Podcast Subscription
Django ships with many field types out of the box. Let's use a variety of them in one form.

- Create a `PodcastSubscriptionForm` with the following fields, each demonstrating a different built-in field class:
  - `email` — `EmailField`
  - `episodes_per_week` — `IntegerField`
  - `favorite_genre` — `ChoiceField` with choices like `Music`, `Comedy`, `True Crime`, `Tech`
  - `notify_new_episodes` — `BooleanField(required=False)`
  - `subscribe_date` — `DateField`
- Render this form (using `{{ form }}` is fine for now) at `/podcast/subscribe/`.
- Handle GET and POST the same way as Q2.

**Goal:** Get familiar with the range of built-in field types and how each renders a different HTML input by default.

---

### Q4. Outputting Forms as HTML — Compare the Options
Django gives you multiple ways to render the same form. Let's compare them side by side.

- Using your `PlaylistForm` from Q1, create **three versions** of the same page (or three sections on one page):
  - Render using `{{ form.as_p }}`
  - Render using `{{ form.as_table }}` (wrap it in a `<table>` tag as Django expects)
  - Render using `{{ form.as_ul }}` (wrap it in a `<ul>` tag)
- Then create a **fourth version** where you manually loop over the fields yourself:
  ```html
  {% for field in form %}
      <div>
          {{ field.label_tag }}
          {{ field }}
      </div>
  {% endfor %}
  ```
- Add a short comment (code comment) noting what you observe is different about the manual loop version compared to the built-in `as_p` / `as_table` / `as_ul` shortcuts.

**Goal:** Understand that `{{ form }}` rendering shortcuts are convenient, but manually looping over fields gives you full control over layout — useful once you start styling forms properly.

---

### Q5. Validating and Cleaning — Playlist Title Rules
Now let's make sure the data submitted is actually valid, and add our own rules on top of Django's defaults.

- In your view for `/playlist/create/`, after receiving a POST request:
  - Check `form.is_valid()`
  - If valid, access the cleaned data via `form.cleaned_data` and display it back to the user (e.g. "Playlist 'X' created!")
  - If invalid, re-render the form so the user can see the errors
- Now add a **custom field-level validation** using a `clean_title` method on `PlaylistForm`:
  - Reject titles shorter than 3 characters, or titles that contain the word `"test"` (case-insensitive) — raise `forms.ValidationError(...)` with a clear message.
- Add a **custom form-level validation** using a `clean()` method:
  - If `description` is provided but `title` is empty (or vice versa — pick a rule that makes sense to you), raise a `ValidationError` explaining that both fields should be filled in together.

**Goal:** Understand the difference between field-level cleaning (`clean_<fieldname>`) and form-level cleaning (`clean()`), and how `form.errors` / `form.cleaned_data` work after calling `is_valid()`.

---

### Q6. Widgets — Styling Podcast Subscription Inputs
Widgets control how a field is rendered as HTML. Let's customize a few.

- Update your `PodcastSubscriptionForm` from Q3 so that:
  - `favorite_genre` uses `widget=forms.RadioSelect` instead of the default dropdown
  - `notify_new_episodes` uses `widget=forms.CheckboxInput` (this is actually the default for `BooleanField`, but explicitly set it so you understand widgets can be swapped)
  - `email` widget has an `attrs` dictionary adding a `placeholder="you@example.com"` and a CSS `class`
  - `episodes_per_week` uses `widget=forms.NumberInput(attrs={'min': 1, 'max': 50})`
- Render the form and confirm each field now looks/behaves differently than the defaults from Q3.

**Goal:** Understand that a **field** defines the type of data and Python-level validation, while a **widget** controls how that field is rendered and what HTML attributes it carries — the two are related but separate concerns.

---

### Q7. Form and Field Validators — Song Request Form
Let's go one level deeper into validation using Django's built-in validators and custom validator functions.

- Create a `SongRequestForm` with:
  - `song_name` — `CharField`, using Django's built-in `MinLengthValidator(2)` and `MaxLengthValidator(100)` passed via the `validators=[...]` argument
  - `requester_email` — `EmailField` (Django validates email format automatically — no extra work needed, but note that this is itself a validator working behind the scenes)
  - `votes_requested` — `IntegerField`, using `MinValueValidator(1)` and `MaxValueValidator(10)`
- Write **one custom validator function** (a plain Python function, outside the form class) called `validate_no_special_characters(value)` that raises a `ValidationError` if `song_name` contains characters like `@`, `#`, `$`, `%`. Attach it to the `song_name` field's `validators` list alongside the length validators.
- Customize at least one field's `error_messages` dictionary (e.g. `error_messages={'required': 'Please tell us which song you want!'}`) so you can see custom error text in place of Django's defaults.
- Test the form with valid data, then intentionally submit invalid data (too-short song name, out-of-range votes, special characters) and confirm each validator triggers correctly.

**Goal:** Understand the difference between validators attached directly to a field (reusable, focused checks) versus the `clean_<field>` / `clean()` methods from Q5 (form-specific logic) — and see how both approaches plug into the same `form.is_valid()` flow.

---
