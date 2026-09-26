# Images

Drop image files directly in this folder (logos, photos, media assets, etc).
Once pushed to GitHub, every file here is served live by the site itself --
no separate image host needed.

## Getting a usable URL

After you push a file here (e.g. `images/dark-horse-logo.png`), its live URL is:

```
https://darkhorsehoops.net/images/dark-horse-logo.png
```

Just swap in the actual filename you used. That's the URL to paste into the
Google Sheet fields that expect an image link -- `Dark Horse Logo URL`,
`Client Logo URL`, `Secondary Client Mark URL`, `Team Photography URL`, etc.

It can take a minute or two after pushing for the file to actually be live at
that URL (GitHub Pages needs to rebuild first).

## Naming files

Keep names lowercase, no spaces -- use dashes instead. A few examples:

- `dark-horse-logo.png`
- `dark-horse-mark.png` (icon-only version, no wordmark)
- `bonn-logo.png`
- `bonn-crest.png` (icon-only version)

## A note on file size

These get fetched by every visitor's browser, so keep logos reasonably small
(a few hundred KB at most is plenty for a logo -- it doesn't need to be a
huge, high-resolution file). PNG works well for logos with transparency;
JPG is fine for photos.
