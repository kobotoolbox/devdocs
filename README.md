[Developer Documentation](https://kobotoolbox.github.io)

## Updating Documentation

You can make simple changes using the GitHub editor ([example 📝](https://github.com/kobotoolbox/kobotoolbox.github.io/edit/main/docs/_articles/kpi-frontend-development.md)).

To run a preview server locally, you'll need `npm`, `ruby`, and `jekyll`. [^1]

[^1]: **macos** comes with a system ruby, but it might not let you install gems like jekyll. The jekyll docs recommend something that failed after brew tried to compile lots of code. Instead I went with (1) `brew install ruby` (no version pin). (2) Use the brew ruby and move the gems folder — in `~/.bashrc` or `~/.zshrc`, `export GEM_HOME=$HOME/gems; export PATH="$(brew --prefix)/opt/ruby/bin:$HOME/gems/bin:$PATH"`. (3) `gem install jekyll`.

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
