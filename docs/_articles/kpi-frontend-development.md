---
title: KPI Front-end Development
---

## Helpful(?) tips

- When debugging, consider eliminating (or confirming) the per-asset cache [(#4835)](https://github.com/kobotoolbox/kpi/pull/4835) as the source of the problem as early as possible. Do so by forcing [`refresh = true` in `loadAsset`](https://github.com/kobotoolbox/kpi/blob/eb21fe685397b2acc17fd36e2b4d7646c99c8bf2/jsapp/js/actions.es6#L427). For a list of issues related to this caching, refer to https://github.com/kobotoolbox/kpi/issues?q=%234835. We intend to replace this caching mechanism, but probably not until 2025. [[internal discussion]](https://chat.kobotoolbox.org/#narrow/stream/4-Kobo-Dev/topic/Deduping.20asset.2Fpermission.20API.20calls/near/424092)

## Code style

Our goals:
- TypeScript (and everything typed)
- React functional components and hooks
- CSS Modules

### More details

General:
- Use Prettier on all new and modified code.
  - Treat it as imperfect cleanup and tidy up the code (so it's easier to read) after it runs.
  - Be aware that older code may not conform.
- Use an editor that respects `.editorconfig`.

Dependencies:
- Use React functional components and hooks.
- For state, use React's `useState`, `useReducer`, `useContext`. When working with Reflux, we are ok with converting it to MobX if it may be a lot easier than converting to React's state management.
- Avoid dependencies when practical.
  - Prefer `fetch*` ([see `api.ts` file](https://github.com/kobotoolbox/kpi/blob/main/jsapp/js/api.ts)) over `JQuery.ajax`.
  - Prefer ES6 built-ins over `lodash`.
- Use `classnames` as `import cx from 'classnames';` to set class names conditionally.

File architecture:
- Organize code by feature, route or some other logical division. Use directory structures like `/settings/emails` instead of `/components`.
- Some directories we use:
  - `js/hooks` - for custom hooks that are shared between multiple components in different routes/features
  - `js/components/common` - for simple general components like button or checkbox
- Prefer one React component per file.
- Prefer smaller files. Avoid splitting into multiple files if it would be harder to work with/understand the divided code.
- Include type of file in filename. Such as:
  - `foo.interface.ts`
  - `foo.reducer.ts`
  - `foo.actions.ts`
  - `foo.module.scss`
  - `foo.component.tsx`
  - `useFoo.hook.ts`
  - `foo.types.ts`
  - `foo.utils.ts`
  - `foo.store.ts`
  - `foo.tests.ts`
- Move TypeScript types and interfaces to separate file if it would be beneficial (e.g. long complex interface requires a lot of scrolling to get to the actual component code).
- Include Storybook stories or tests as sibling file (e.g. `button.stories.tsx` next to `button.component.tsx`).
- Avoid deep relative paths in imports. Do not import `../../../foo/bar/far.ts`.

JS:
- Use TypeScript. If modifying a non-TypeScript file, update it to TypeScript.
- Use `isSomething`/`hasSomething` naming convetion for booleans. Some common ones we often use:
  - `isLoading` - means waiting for some async fetch of data (may switch value back and forth).
  - `isFirstLoadComplete` - means that all the data necessary for displaying a component was gathered, and the UI was displayed to the user (we also have `isInitialised` in our codebase, but it's not as precise, and thus deprecated)
- Comments are good, we like comments. It's good to write a short description of a component or utlity function, or to explain some complex code step-by-step.
  - Bonus: if you're trying to understand some code for the first time, it's probably a perfect moment to add some comments for the next adventurer :).
- We use JSDoc comments to describe classes, functions, variables, and properties. We use regular comments for everything else. Many editors understand and use JSDoc comments internally and thus are helpful.
- Use `t()` for every Front-end facing static string.
  - `/kpi/jsapp/js/i18nMissingStrings.es6` file holds all the strings we want to translate but don't appear in our Front-end code.
- At minimum write tests for utility functions.

CSS:
- Use CSS modules. Do not use BEM style class names, unless appropriate for complex or global CSS. Do not use the deprecated `makeBem` utility.
- We use autoprefixer, so no need to add prefixes manually.
- Avoid adding new colors to stylesheets. We have an existing [list of all available colors](https://github.com/kobotoolbox/kpi/blob/main/jsapp/scss/colors.scss). If the design contains a color that is very similar to existing one - use that color. If it's completely new color, please discuss adding it to the list with Design Team or Front-End Lead.

## Common components

These are Buttons, Checkboxes, and many other common/atomic UI elements. The aim is to write DRY code and build beautiful, consistent UI. We gather them all in `js/components/common` directory, but you can see them all in action at our [storybook](https://storybook.kbtdev.org).

Please:
- Make sure you know what we have.
- Avoid creating custom one-shots that are very similar to existing common components.
- Adding new option to common component (size, color, etc.) shouldn't be done lightly.
  - Make sure to update the story file for the component if you ever add that option.
- If in doubt, please refer to Design Team or Front-End Lead.

## Workflow

We follow Cleanup First™ methodology. It boils down to making a cleanup-only PR before starting any work. A step-by-step guide would be:

1. Think about which files your changes will touch.
2. Make a PR that includes only cleanup<sup>1</sup> changes in those files.
3. Make a new branch for the actual changes from the cleanup branch.
4. If you end up needing to cleanup more files as you work, make a new cleanup PR.
5. If you end up cleaning more files than needed, it's ok.

That way we get less merge conflicts, and less complex PRs to review.

<sup>1</sup> by "cleanup" we mean stylistic changes, applying linter and prettier, moving things around, renaming, etc.

## Adding new icons

Some notes first:

- we generate an icons font from `svg` files using [webfonts-generator](https://www.npmjs.com/package/webfonts-generator) package
- the source of the icons is the Sketch file from our Google Drive: `/Kobo Product Design/Design Current/04. Design/Comps Sketch/Icons.sketch`
- all the icons need to have the same artboard size (`webfonts-generator` requirement)
- after [kpi#4109](https://github.com/kobotoolbox/kpi/issues/4109) is done, this will be much simpler 😇

Steps to take when adding new icon:

1. Open `Icons.sketch`
2. Create new artboard `Icons/<icon-name>` with the same size as the other. Make sure it is exportable (easiest way is to clone an existing one)
3. Draw the icon or drag&drop existing SVG file, make sure the paths are combined and the color is black (`#000000`)
4. Export all icons
5. Copy new icon from `Icons/<icon-name>.svg` to `jsapp/svg-icons` in KPI repository
6. Run the new `svg` file through [ImageOptim.app](https://imageoptim.com) or some alternative non-destructive SVG compressor
7. Run `npm run generate-icons`
8. Use your new icon, e.g. `<Icon name='alert'/>` (or through some other component that uses icons internally)

## Getting translations

Getting latest translations strings on local environment requires few steps:

1. In your `kpi` directory, navigate into `/locale`
2. Do a `git pull`
3. In your `kobo-install` directory, enter container: `./run.py -cf exec kpi bash`
4. Run `./manage.py compilemessages`
