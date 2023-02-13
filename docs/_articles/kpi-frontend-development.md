---
title: KPI Frontend Development
---

## Code style

- Use Prettier on all new and modified code. Be aware that older code may not conform. Use an editor that respects `.editorconfig`.
- Use CSS modules. Do not use BEM style class names, unless appropriate for complex or global CSS. Do not use the `makeBem` utility.
- Use TypeScript. If modifying a non-TypeScript file, update it to TypeScript.
- Use React functional components and hooks.
- Prefer one React component per file and smaller files.
- Organize code by feature. Use directory structures like `/settings/emails` instead of `/components`.
- Include type of file in filename. Such as `foo.interface.ts`, `foo.reducer.ts`, `foo.module.scss`, or `foo.component.tsx`.
- Include storybook stories as sibling file. `foo.component.stories.tsx`.
- Avoid dependencies when practical. Prefer `fetch` (see `api.ts` file) over `JQuery.ajax`.
- For state, use React's `useState`, `useReducer`, `useContext`. Or use MobX. Do not use Reflux. When working with Reflux, convert it to MobX which may be easier than converting to React's state management.
- Avoid deep relative paths in imports. Do not import `../../../foo/bar/far.ts`.

## Workflow

We follow Cleanup First™ methodology. It boils down to making a cleanup-only PR before starting any work. A step-by-step guide would be:

1. Think about which files your changes will touch.
2. Make a PR that includes only cleanup<sup>1</sup> changes in those files.
3. Make a new branch for the actual changes from the cleanup branch.
4. If you end up needing to cleanup more files as you work, make a new cleanup PR.
5. If you end up cleaning more files than needed, it's ok.

That way we get less merge conflicts, and less complex PRs to review.

<sup>1</sup> by "cleanup" we mean stylistic changes, applying prettier, moving things around, renaming, etc.

## Adding new icons

Some notes first:

- we generate an icons font from `svg` files using [webfonts-generator](https://www.npmjs.com/package/webfonts-generator) package
- the source of the icons is the Sketch file from our Google Drive: `/Kobo Product Design/Design Current/04. Design/Comps Sketch/Icons.sketch`
- all the icons need to have the same artboard size (`webfonts-generator` requirement)
- after [kpi#3305](https://github.com/kobotoolbox/kpi/issues/3305) is done, this will be much simpler 😇

Steps to take when adding new icon:

1. Open `Icons.sketch`
2. Create new artboard `Icons/<icon-name>`, make sure it is exportable (easiest way is to clone an existing one)
3. Draw the icon or drag&drop existing SVG file, make sure the paths are combined and the color is black (`#000000`)
4. Export all icons
5. Copy new icon from `Icons/<icon-name>.svg` to `jsapp/svg-icons` in KPI repository
6. Run the new `svg` file through [ImageOptim.app](https://imageoptim.com) or some alternative non-destructive SVG compressor
7. Run `npm run generate-icons`
8. Use your new icon: `<i className='k-icon k-icon-<icon-name>'/>`
