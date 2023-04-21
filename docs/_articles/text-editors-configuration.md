---
title: Recommended Text Editors Configuration
---

## Front-end

We use a number of different linting tools to keep the code consistent. To take full advantage of them your text editor needs some plugins.

### Sublime Text (4)

Recommended packages:
- `EditorConfig`
- `JsPrettier`
- `LSP`
- `LSP-typescript`
- `LSP-css`
- `Sass`
- `SublimeLinter`
- `SublimeLinter-eslint`
- `SublimeLinter-stylelint`

#### JsPrettier

Our current codebase is very un-pretty, so it's best to not prettify whole files when working with some old ones. We can disable formatting when saving in the package configuration file:

```js
{
  "auto_format_on_save": false,
  "auto_format_on_save_requires_prettier_config": true
}
```

### VS Code

Open [`kpi.code-workspace`](https://github.com/kobotoolbox/kpi/blob/beta/kpi.code-workspace), install the recommended extensions. `npm install` to make ESLint, Prettier, Stylelint and TypeScript work well. For more details, see the PR description and comments here: [Toolchain: Better out-of-the-box experience in VS Code](https://github.com/kobotoolbox/kpi/pull/4230)
