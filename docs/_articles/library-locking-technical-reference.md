---
title: Library Locking Technical Reference
---

## TOC

- [Overview](#overview)
- [Restrictions](#restrictions)
  - [`kobo--lock_all`](#kobo--lock_all)
  - [Question](#question-level-restrictions)
  - [Group](#group-level-restrictions)
  - [Form](#form-level-restrictions)
- [XLSForm Configuration](#xlsform-configuration)
  - [`kobo--locking-profiles`](#kobo--locking-profiles)
  - [`settings`](#settings)
  - [`survey`](#survey)
  - [Form Validation](#form-validation)
  - [Caveats](#caveats)
- [JSON Representation](#json-representation)
- [Locking Profiles and Asset Types](#locking-profiles-and-asset-types)
  - [Importing Locked XLSForms](#importing-locked-xlsforms)
- [Useful Links](#useful-links)
- [Terminology](#terminology)

## Overview

> Currently, only locking set within the XLSForm itself is supported, but will be incorporated into the form-builder at some point in the future.

"Library locking" refers to the feature enabling various aspects of a survey to be "[locked](#locked)" when created from a template containing locking attributes. All aspects of a form's editing are available to be locked through the assigning of "locking [profiles](#profile)" at the form, question or group level. These locking profiles can be assigned granular "[restrictions](#restriction)" that group together locking functionalities, or the form can be fully locked down, preventing editing of all aspects.

## Restrictions

There are three levels of restrictions that can be set (based on the [working template](https://docs.google.com/spreadsheets/d/1JI2JQ2UFrPvUh3ZuwiAoMBrol_KshkIARloSTi6UOM4/edit?usp=sharing)):
1. [Question](#question-level-restrictions),
2. [Group](#group-level-restrictions), and
3. [Form](#form-level-restrictions)

Additionally, there is a [`kobo--lock_all`](#kobo--lock_all-1) Boolean that can set in the `settings` sheet that will render the survey completely locked.

### `kobo--lock_all`

If `kobo--lock_all` is set to `True`, then all additional granular restrictions are redundant as the form is _fully_ locked down. If it is set to `False` _or_ omitted from the `settings` sheet, then defined locking profiles can be used to control the locked behaviour:

| kobo--lock_all |
|----------------|
| true           |

The accepted strings for the value of `kobo--lock_all` are the same as in the `survey` sheet that [pyxform supports](https://github.com/XLSForm/pyxform/blob/43ea039250f44cff23b3ad10740fca54dfa12383/pyxform/aliases.py#L127-L142). [No error will be thrown](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-6573a09c489c0f4e13378058c131b10529a9ea0f401ffee216944eab369a95adL264-L268) if an invalid string is used, only the form will not function as intended from the user's perspective.

> Note that the restriction name, such as `choice_add` below, [is predefined](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-1aa08b99a58a277e2c4ddfb29a24c11134e0812bc89bb5606e2fd601dce6ff6aR116-R147) and only the restrictions listed below are valid options.

### Question-level Restrictions

| Name                     | Description                                                      |
|--------------------------|------------------------------------------------------------------|
| choice_add               | Add new choices to a select_* question                           |
| choice_delete            | Remove an existing choice from a select_* question               |
| choice_value_edit        | Edit a choice value (name)                                       |
| choice_label_edit        | Edit a choice label                                              |
| choice_order_edit        | Reorder the choices of a select_* question                       |
| question_delete          | Delete a given question                                          |
| question_label_edit      | Edit an existing label or hint                                   |
| question_settings_edit   | Edit a question's settings (other than constraint or relevant)   |
| question_skip_logic_edit | Edit a question's skip logic settings                            |
| question_validation_edit | Edit a question's validation criteria settings                   |

### Group-level Restrictions

| Name                      | Description                                                                                          |
|---------------------------|------------------------------------------------------------------------------------------------------|
| group_delete              | Delete group modal "Delete everything" button (or delete group button if paired with "group_split")  |
| group_split               | Delete group modal "Ungroup questions" button (or delete group button if paired with "group_delete") |
| group_label_edit          | Edit a group label                                                                                   |
| group_question_add        | Adding or cloning questions inside given group (children groups included)                            |
| group_question_delete     | Delete any question from given group (children groups questions included)                            |
| group_question_order_edit | Changing order of questions inside given group (children groups included)                            |
| group_settings_edit       | Changing "All group settings" from given group "Settings"                                            |
| group_skip_logic_edit     | Changing "Skip Logic" from given group "Settings"                                                    |

### Form-level Restrictions

| Name                | Description                                                                            |
|---------------------|----------------------------------------------------------------------------------------|
| form_appearance     | Changing form appearance from "Layout & Settings"                                      |
| form_replace        | Replacing form using Replace Form modal                                                |
| group_add           | Button for grouping questions                                                          |
| question_add        | Using "Insert cascading select" option and each Add Question and Clone Question button |
| question_order_edit | Changing any questions order                                                           |
| language_edit       | Edit languages in Translations Modal                                                   |
| form_meta_edit      | Edit meta questions from "Layout & Settings"                                           |

## XLSForm Configuration

There are three sheets where locking profiles are defined and set: `survey`, `settings` and [`kobo--locking-profiles`](#kobo--locking-profiles). The sheet of `kobo--locking-profiles` is not officially supported by [pyxform](https://github.com/XLSForm/pyxform) and is KoboToolbox-specific, [handled in FormPack](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R18-R79).

Form-level restrictions are defined in the `settings` sheet and question and group-level restrictions are defined in the `survey` sheet.

From within the `kobo--locking-profiles` sheet, all the locking profiles are defined in a matrix structure, [using](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R75-R77) the keyword "[locked](#locked)" to assign a "[restriction](#restriction)" to a specific "[profile](#profile)". For example:

### `kobo--locking-profiles`

> Define the profiles and assign them restrictions. Note that not all valid restrictions need to be included in the `restriction` column, but a [`FormPackLibraryLockingError`](#formpacklibrarylockingerror) will be thrown if an invalid restriction is included.

| restriction       | profile_1 | profile_2 | profile_3 |
|-------------------|-----------|-----------|-----------|
| choice_add        | locked    |           |           |
| choice_delete     |           | locked    |           |
| choice_label_edit | locked    |           |           |
| choice_order_edit | locked    | locked    |           |
| form_appearance   |           |           | locked    |

### `settings`

> Set form-level restrictions and [`kobo--lock_all`](#kobo--lock_all-1) Boolean (note again that `kobo--lock_all` can be _omitted_ if it's intended to be `False`)

| kobo--locking-profile | kobo--lock_all |
|-----------------------|----------------|
| profile_3             | false          |

> Note that omitting `kobo--lock_all` from the settings sheet is the equivalent of setting it to `False`.

### `survey`

> Set question and group-level restrictions

| type                 | name    | kobo--locking-profile |
|----------------------|---------|-----------------------|
| select_one countries | country | profile_1             |
| select_one cities    | city    | profile_2             |

### Form Validation

The following cases will currently throw a [`FormPackLibraryLockingError`](#formpacklibrarylockingerror):
- If a locking profile name (column header in the `kobo--locking-profiles` sheet) is "locked" (the same as the locking keyword): This is not a necessary requirement from the backend, but more just to prevent confusion in the UX of the XLSForm ([ref](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R172-R175))
- If a restriction is listed in the `kobo--locking-profiles` `restriction` column that is invalid (not in the list of predefined restrictions) ([ref](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R71-R74))
- If there is a sheet called `kobo--locking-profiles` but no `restriction` column ([ref](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R158-R161))
- If no locking profiles are defined (column headers in the `kobo--locking-profiles` sheet) ([ref](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R167-R170))

Validation of the XLSForm library locking features will be expanded [in the future](https://github.com/kobotoolbox/kpi/projects/4#card-59636299).

### Caveats

In some spreadsheet editors, two single dashes (--) are automatically converted to an m-dash (—), therefore making it difficult to type `kobo--` into a cell. The [backend handles this gracefully(?)](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-6573a09c489c0f4e13378058c131b10529a9ea0f401ffee216944eab369a95adR208-R217) and converts all instances of n- and m-dashes into two single dashes (when prefixed with `kobo`). Therefore an XLSForm with the sheet name of "kobo—locking-profiles" will be converted to "kobo--locking-profiles" and similarly for the column headers. When the form is exported from the UI, the new double-dash will be exported and therefore the original and exported XLSForms will not be identical.

## JSON Representation

There are two attributes of the asset where locking information can be accessed and modified: `asset.summary` and `asset.content`.

If [`kobo--locking-profile`](#kobo--locking-profile) is a column name in the `survey` sheet, it will also be listed in the `asset.summary.columns` array.

In the summary, the following two Boolean attributes describe an overview of the form's locking structure (and locking appearance in the list view in the UI):
- `lock_all`, and
- `lock_any`

The [logic](https://github.com/kobotoolbox/kpi/pull/3082/files#diff-527a6215d93dbd35938e13897c85c1f17cd50c19f3d9379925551de4671da71bR61-R76) by which each of those Booleans are set is as follows:
- `lock_all` is `True` _only_ if `kobo--lock_all` is set to `True` in the `settings` sheet, otherwise it's `False`
- `lock_any` is set to `True` if _any_ of the following cases are `True`:
  - `lock_all` is `True`,
  - A `kobo--locking-profile` is set in the `settings` sheet, or
  - _At least one_ `kobo--locking-profile` is set in the `survey` sheet

In the example above, the following will be present in the `asset.summary`:
```
{
  ...,
  "columns": [
    ...,
    "kobo--locking-profile"
  ],
  "lock_all": false,
  "lock_any": true,
  ...
}
```

In the content, an attribute of [`content.kobo--locking-profiles` exists as an array of JSON objects](https://github.com/kobotoolbox/kpi/pull/3082/files#diff-9a5e3fb1b4a1e497cbcdfd20383757cf0542b43ad7fd6af29b67583e43ddd566R760) with the following structure:
```json
[
  {
    "name": "profile_1",
    "restrictions": [
      "choice_add",
      "choice_label_edit",
      "choice_order_edit",
    ]
  }
]
```
In `content.settings`, the following will be present in a JSON object:
```json
{
  "kobo--locking-profile": "profile_3",
  "kobo--lock_all": false
}
```
And finally in `content.survey`, each question that has been assigned a locking profile will have a `kobo--locking-profile` attribute as follows:
```json
[
  {
    "name": "country",
    "type": "select_one",
    "kobo--locking-profile": "profile_1"
  },
  {
    "name": "city",
    "type": "select_one",
    "kobo--locking-profile": "profile_2"
  }
]
```

## Locking Profiles and Asset Types

Of the four asset types (`asset`, `template`, `question` and `block`), [only `template`s and `survey`s handle library locking features](https://github.com/kobotoolbox/kpi/pull/3082/files#diff-9a5e3fb1b4a1e497cbcdfd20383757cf0542b43ad7fd6af29b67583e43ddd566R356-R359) and the locks are enforced _only_ on surveys. Practically, this means the following:

Assume an XLSForm containing valid locking features is imported:
- If imported as a `block` (imported through the Library section), then all traces of locking are excluded and/or [stripped](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-f57f6fb770f92cab2c8dee56f56aa6ae4a3da963caec75e02cc50ea26d67ab11R142-R151) from [the asset](https://github.com/kobotoolbox/kpi/pull/3082/files#diff-8f034d28f38b81acc59871ef611be97065c6d4f41423bb1bac265d7bbbe52175R711-R714). This results in a `block` asset that will be equivalent to a form uploaded without any locking features;
- If imported as a `survey` (imported through the Projects section) or `template` (not yet supported) then all locks are intact:
  - If, from within the form-builder:
    - a question is added to the library, then all locks are stripped from the new `question` asset
    - a group of questions is added to the library as a `block` (not yet support), then all locks are stripped
  - If a `template` is created _from_ the locked `survey` asset, then that `template` will inherit all the locks the `survey` had,
  - If a `survey` is created _from_ a locked `template`, the survey will inherit all the locks that the `template` had

| Original Asset Type | Process                 | Resulting Asset's Status |
|---------------------|-------------------------|--------------------------|
| `survey`            | -                       | locked                   |
| `survey`            | `survey` to `template`  | locked                   |
| `survey`            | `survey` to `question`  | not locked               |
| `survey`            | `survey` to `block`     | not locked               |
| `template`          | -                       | locked                   |
| `template`          | `template` to `survey`  | locked                   |
| `template`          | `template` to `question`| not locked               |
| `template`          | `template` to `block`   | not locked               |
| `block`             | -                       | not locked               |
| `block`             | import to `survey`      | not locked               |
| `block`             | import to `template`    | not locked               |
| `block`             | `block` to `question`   | not locked               |
| `question`          | -                       | not locked               |
| `question`          | import to `survey`      | not locked               |
| `question`          | import to `template`    | not locked               |
| `question`          | `question` to `block`   | not locked               |

### Importing Locked XLSForms

Until uploading of XLSForms directly to `template`s [is implemented](https://chat.kobotoolbox.org/#narrow/stream/7-UX.2FUI/topic/Template.20handling.20%28for.20Library.20Locking%29/near/24726), there are three main ways of obtaining a locked `template`:
- Import a locked XLSForm as a `survey` and clone to a `template`
- Import directly to a template with [temporary hack](https://github.com/kobotoolbox/kpi/commit/6634a308e31ef4d8619823ee1d2d7d39e71f67a3):
  - Include a `calculate` question named `kobo--template-test` in the `survey` sheet
- Manually `PATCH` an "[unlocked](#unlocked)" `template`

## Useful Links

- [Staging server](http://kf.unicef.kbtdev.org/)
- [Original project outline](https://github.com/kobotoolbox/kpi/issues/2991)
- Backend:
  - [kobotoolbox/kpi#3082](https://github.com/kobotoolbox/kpi/pull/3082)
  - [kobotoolbox/formpack#238](https://github.com/kobotoolbox/formpack/pull/238)
- Frontend:
  - [kobotoolbox/kpi#3127](https://github.com/kobotoolbox/kpi/pull/3127)
- [Project board](https://github.com/kobotoolbox/kpi/projects/4)
- [Internal discussion](https://chat.kobotoolbox.org/#narrow/stream/4-Kobo-Dev/topic/Library.20locking)

## Terminology

#### `FormPackLibraryLockingError`
The exception class [defined in FormPack](https://github.com/kobotoolbox/formpack/pull/238/files?file-filters%5B%5D=.py#diff-8d95cd08d33d1507c0f4f5836d1936da466e7fc5ddf6bcf127daaac7761505faR8-R9) that will be thrown in the event of a library-locking-specific error.
#### `kobo--lock_all`
Attribute containing a Boolean value, set in the `settings` sheet and applies all locking restrictions to the form and all questions and groups (rendering granular locking profiles redundant).
#### `kobo--locking-profile`
Column name in the `survey` and `settings` sheets where the locking profile is assigned to a question or group (in `survey`) or to the form (in `settings`).
#### `kobo--locking-profiles`
Sheet name where restrictions are assigned to profiles.
#### `locked`
Keyword used to assign a restriction to a profile in the `kobo--locking-profiles` sheet.
#### Profile
The name assigned to a group of restrictions, defined in the `kobo--locking-profiles` sheet. It is assigned to questions and groups in the `survey` sheet and to the from in the `settings` sheet.
#### Restriction
A granular locking attribute that can be assigned to a profile and control the locking behaviour at the question, group or form level.
#### Unlocked
A form containing no locking attributes.
