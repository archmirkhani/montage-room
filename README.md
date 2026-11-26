# 몽타주실 — DRAW WHAT YOU HEARD?!

같은 증언을 듣고도 사람마다 다른 얼굴을 떠올린다는 것을 실험하는 인터랙티브 몽타주 게임입니다.
An interactive montage-drawing game about how the same words create different faces.

## Folder structure

```
montage-room/
├── index.html            ← the website (must stay at the root)
├── .nojekyll             ← tells GitHub Pages to serve files as they are
├── css/style.css
├── js/config.js          ← shared-gallery settings (you fill this in)
├── js/game.js            ← all game logic
├── assets/images/        ← intro (111, 222), police (polis), witnesses (sh1–sh4),
│                            suspects (spiderman, patrick), result art (suc, car)
├── assets/audio/         ← talk_sound1/2, police-siren, this-is-the-police, patrick-sound
└── supabase/schema.sql   ← database setup for the shared gallery
```

All paths are relative, so the site works at `https://<user>.github.io/<repository>/`.

## 1. Publish on GitHub Pages

1. Create a new **public** repository on GitHub (e.g. `montage-room`).
2. Upload **the contents of this folder** (not the folder itself) so that `index.html` is at the top level of the repository.
   *Add file → Upload files*, drag everything in, including `.nojekyll`, then *Commit changes*.
   (If your file browser hides `.nojekyll`, the site still works; it only avoids Jekyll processing.)
3. Go to *Settings → Pages*. Under **Build and deployment**, choose *Deploy from a branch*, branch `main`, folder `/ (root)`, then *Save*.
4. After about a minute, the site is live at `https://<your-user-name>.github.io/<repository-name>/`.

The game, drawing tools, colour editor, scoring, witnesses, sounds and result animations all work at this point.
**The shared gallery needs step 2.**

## 2. Turn on the shared gallery (Supabase, free)

GitHub Pages only hosts files; it cannot store visitors' drawings. The gallery therefore uses a free
[Supabase](https://supabase.com) database. Every visitor, on any device, then sees the same gallery.

1. Sign up at supabase.com and click **New project** (any name, any region close to your users; the free plan is enough).
2. Open **SQL Editor → New query**, paste the whole contents of `supabase/schema.sql`, and click **Run**.
   This creates the `gallery` table with rules that let visitors **view and add** drawings but **never edit or delete** them.
3. Open **Project Settings → API** (or **Data API**) and copy:
   - **Project URL** (looks like `https://abcdefghijk.supabase.co`)
   - the **anon / public** key (a long text starting with `eyJ…`, or `sb_publishable_…`)
4. Open `js/config.js` on GitHub (pencil icon → edit), paste the two values, and commit:

   ```js
   window.MONTAGE_CONFIG = {
     supabaseUrl: 'https://abcdefghijk.supabase.co',
     supabaseAnonKey: 'eyJhbGciOi...'
   };
   ```

5. Reload your GitHub Pages site, play once, submit a drawing, then open **공동 갤러리** from a different device to confirm it appears.

### About keys

- The **anon / publishable** key is designed to be public, and is safe here because `schema.sql` limits it to *view* and *add* only.
- **Never** put the `service_role` / secret key in `config.js`, or anywhere else in this repository.

### What is stored

Per submission: the drawing (small thumbnail + larger JPEG), case number, suspect name, similarity score,
success/failure, submission time, and an anonymous label like `USER 7K2Q` (a random code kept in that browser).
No names, e-mail addresses or IP addresses are stored by the game.

### Moderation

To remove a drawing: Supabase dashboard → **Table Editor → gallery** → select the row → **Delete**.

### Capacity

The free plan's 500 MB database holds roughly 2,000–5,000 drawings (about 60–250 KB each).

## How the gallery chooses its server

`js/game.js` checks, in order:

1. **Inside claude.ai** (the original Claude artifact): it uses the artifact's built-in shared storage.
2. **Anywhere else** (GitHub Pages, your own computer): it uses Supabase if `js/config.js` is filled in.
3. If neither is available, the game still works and the gallery shows a "not configured" message.
   Submitted drawings are then **not** saved.

The Claude-artifact gallery and the Supabase gallery are separate databases: drawings submitted on one do not appear on the other.

## Notes

- **Fonts:** the page loads Nanum Gothic, Nanum Pen Script, Special Elite and VT323 from Google Fonts, so it needs an internet connection (normal on GitHub Pages). Without it, system fonts are used.
- **Every refresh starts a new game:** blank canvas, new random suspect. Unfinished drawings are never saved; submitted drawings stay in the gallery.
- **Similarity scoring:** the score is an approximate comparison of colour distribution, silhouette, and where lines and filled areas sit, measured against the suspect image. It is not AI face recognition, and the same drawing always gets the same score. 80% or higher counts as a capture.
- **Optional custom bucket icon:** put a file named `bucket.png` in `assets/images/` and the paint-bucket tool uses it automatically. Without it, the built-in pixel icon is shown.
- **Local testing:** open the folder with a local web server (for example `python3 -m http.server`) rather than double-clicking `index.html`, because browsers restrict audio loading from `file://` pages.
