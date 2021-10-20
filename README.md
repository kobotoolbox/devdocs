[Developer Documentation](https://kobotoolbox.github.io)

## Updating Documentation

### Building local instance

1. `npm install`
2. `npm run jekyll-serve`
3. Open [localhost:2038](http://localhost:2038)

### Adding article

1. Create new `md` file at `docs/_articles/`
   - _Note:_ the file name becomes the url, so please use `snake-case` <3
2. Article file needs a frontmatter with `title`
   ```
   ---
   title: How to Train a Dragon?
   ---
   ```
3. Images go to `docs/images/`, and you add them with markdown:
   ```
   ![alt text](url "A title")
   ```

### Updating styles

Two ways:

- `npm run styles-build`: build styles once, apply prefixes
- `npm run styles-watch`: build styles continously, no prefixes applied

_Note_: styles are built at pre-commit hook, to ensure you commit your changes and to ensure you don't edit `docs/style.css` directly (sorry!).
