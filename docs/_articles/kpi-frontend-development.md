---
title: KPI Frontend Development
---

## Code style

We use multiple different libraries that forces the code to be consistent. Please make sure your code editor has all possible plugins installed!

- we use spaces 🙊
- [JSDoc](https://jsdoc.app/) currently we comment everything, but will be dropping it when we move to TypeScript
- [editorconfig](https://editorconfig.org/) ([our config](https://github.com/kobotoolbox/kpi/blob/master/.editorconfig)) - best set it to autofix on file save
- [prettier](https://prettier.io) ([our config](https://github.com/kobotoolbox/kpi/blob/master/.prettierrc.json)) - as for now please use it only on files you've edited in a great way, or on new files
- [eslint](https://eslint.org/) ([our config](https://github.com/kobotoolbox/kpi/blob/master/.eslintrc.json))
- [stylelint](https://stylelint.io/) ([our config](https://github.com/kobotoolbox/kpi/blob/master/.stylelintrc.json))
- [coffeelint](http://www.coffeelint.org/) ([our config](https://github.com/kobotoolbox/kpi/blob/master/coffeelint.json))
- we prefer `() => {}` over `function () {}` for unnamed callback functions (for brevity).
- for S(CSS) we follow [BEM](http://getbem.com/introduction/) methodology (use our built in `bem`/`makeBem` utility), also:
  - try avoiding nested class names (mostly unnecessary)
  - try avoiding the `&` shortcut (makes it harder to find class names in the code)

## Workflow

We follow Cleanup First™ methodology. It boils down to making a cleanup-only PR before starting any work. A step-by-step guide would be:

1. Think about which files your changes will touch.
2. Make a PR that includes only cleanup<sup>1</sup> changes in those files.
3. Make a new branch for the actual changes from the cleanup branch.
4. If you end up needing to cleanup more files as you work, make a new cleanup PR.
5. If you end up cleaning more files than needed, it's ok.

That way we get less merge conflicts, and less complex PRs to review.

<sup>1</sup> by "cleanup" we mean stylistic changes, applying prettier, moving things around, renaming, etc.

## Files architecture

Some rules we follow:

- only one React component per file
- generally we prefer smaller files
- when creating new React component:
  - you should create all it's files next to it, e.g.:
    ```
    jsapp/js/components/submissions/table.es6
    jsapp/js/components/submissions/table.tests.es6
    jsapp/js/components/submissions/table.scss
    ```
  - import styles file in the component `.es6` file
- group `.es6` files by feature, e.g. `library/`, `submissions/`
- generic/common React components go to `jsapp/js/components/common` directory
- don't use relative paths in imports

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

## TypeScript migration

We are incrementally migrating our codebase to TypeScript. Here's a bunch of useful (and still very much evolving!) notes:

- When you rename `es6` file, if it's a component, rename it to `tsx`, else go with `ts`.
- When you encounter an untyped file (e.g. your new `.ts`/`.tsx` file has an import that is not TypeScript) there are two approaches:
  1. Preferred: Switch the untyped file to TypeScript (i.e. change the file extension and write types for everything inside).
  2. Faster: Leave untyped file as is, but create `<file name>.d.ts` file next to it with types definitions.
- Let's define types for all the endpoints responses in `dataInterface.d.ts` for now. Ultimately we would probably move them to their respective files with actions definitions, but this plan is not yet fully developed.
- When writing types for a Reflux store, you need to create a class that extends `Reflux.Store`, see `assetStore.ts` as a good example file.
- TypeScript has two different kinds of modules: ambient and normal. This is best described with example - imagine I have some `.d.ts` file with type definition - thanks to TypeScript, we can use the `Asset` interface in all the places without importing it (this is the ambient module):
  ```typescript
  interface Asset {
    type: string
  }
  ```
  And now we introduce an `enum AssetType` with all possible asset types in `constants.ts` file. We want to use it at `Asset` interface for `type` property. We could do this:
  ```typescript
  import {AssetType} from 'js/constants'
  interface Asset {
    type: AssetType
  }
  ```
  But if a typescript definition file has any import it stops being "ambient" module and becomes a "normal" one, meaning that we no longer can use `Asset` without importing it. This is definitely not nice. But there is a nice override (a dynamic import) that allows importing some type from other file without compromising the ambient status:
  ```typescript
  interface Asset {
    type: import('js/constants').AssetType
  }
  ```
- all `interface`s, `type`s and `enum`s should be named using PascalCase
