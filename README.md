# TheEverythingLibrary

A free, public library of interactive web widgets — fancy, stylish, tasteful components that anyone can **copy, paste, and ship with zero dependencies**.

**Live site:** https://carolllmy.github.io/the-everything-library/

Every widget is a self-contained HTML snippet (markup + CSS + JS in one block) that works the moment you paste it into any page. No build step, no framework, no npm install. The whole site is one `index.html`.

## Using a widget

1. Browse or search ("a button that follows my cursor", "something that celebrates", "loading placeholder"…)
2. Click a tile to see the live preview
3. Hit **Copy code** and paste the snippet anywhere in your page's HTML

That's it. Each snippet:

- uses class names prefixed `tel-` so it won't collide with your styles
- loads no external assets or libraries
- works on light and dark pages

Some widgets have a **Customize** note in their detail view (e.g. the typewriter's phrase list, the count-up's target numbers).

## The widgets (20)

| Category | Widgets |
|---|---|
| Buttons | Magnetic, Confetti, Ripple, Shimmer, Border Beam |
| Text | Gradient Flow, Typewriter, Glitch, Count-Up Stat, Scramble, Wave |
| Cards | Spotlight, 3D Tilt, Glass |
| Controls | Springy Toggle, Star Rating, Bubble Slider |
| Loaders | Orbit, Skeleton Shimmer, Progress Ring |

Every widget is shareable by direct link, e.g. [`#magnetic-button`](https://carolllmy.github.io/the-everything-library/#magnetic-button).

## Requesting a widget

Search for what you need — if nothing matches, a request form appears. I read every request and they drive what gets added next.

## How the search works

No AI API — a small client-side engine: tokenization with stopword removal, light stemming, exact/prefix/substring/typo-tolerant (Levenshtein) matching against a rich per-widget tag vocabulary, with a phrase bonus and a precision guard so unrelated queries return zero results instead of noise.

## Adding a widget (contributors / future me)

Append one object to the `WIDGETS` array in `index.html`:

```js
{
  id:"kebab-case-id", cat:"buttons|text|cards|controls|loaders",
  name:"Display Name",
  desc:"One-line description.",
  notes:"Optional customize hint shown in the modal (HTML allowed).",
  tags:[/* 12–17 tags: synonyms, intents, use cases — this is the search quality lever */],
  code:`...self-contained snippet...`
}
```

Snippet conventions:

- prefix every class with `tel-`
- zero external assets or libraries
- no backticks or `${}` inside widget JS (snippets live in template literals)
- escape closing script tags as `<\/script>`
- wrap the demo in a centered container with ~200px min-height

## Running locally

Open `index.html` in a browser. The hidden request log is at `index.html?admin=1`.

To receive widget requests by email, create a free form at [formspree.io](https://formspree.io) and paste the endpoint into the `FORMSPREE_ENDPOINT` constant at the top of the script.

## License

[MIT](LICENSE) — free to use, share, remix.
