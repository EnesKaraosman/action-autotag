# Autotag Flutter App

This is a fork from [jacopocarlini/action-autotag](https://github.com/jacopocarlini/action-autotag) for **Flutter** application.

This action will read a `pubspec.yaml` file and compare the `version` attribute to the project's known tags. If a corresponding tag does not exist, it will be created.

Usually, the version number is three numbers separated by dots, followed by an optional build number separated by a +, such as `1.2.21+7`. **The optional build number is NOT ignored by default when the tag is created, use `include_build_number` to customize (default true)**.

This tag works well in combination with:

- [actions/create-release](https://github.com/actions/create-release) (Auto-release)
- [author/action-publish](https://github.com/author/action-publish) (Auto-publish JavaScript/Node modules)
- [author/action-rollback](https://github.com/author/action-rollback) (Auto-rollback releases on failures)
- [author/template-cross-runtime](https://github.com/author/template-cross-runtime) (a cross-runtime JavaScript repo template)

## Usage

The following is an example `.github/workflows/main.yml` that will execute when a `push` to the `main` branch occurs.

```yaml
name: Create Tag

on:
  push:
    branches:
      - main

jobs:
  create_tag:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: eneskaraosman/action-autotag
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

To make this work, the workflow must have the checkout action _before_ the autotag action.

This **order** is important!

```yaml
- uses: actions/checkout@v4
- uses: eneskaraosman/action-autotag
```

> If the repository is not checked out first, the autotagger cannot find the pubspec.yaml file.

## Configuration

The `GITHUB_TOKEN` must be passed in. Without this, it is not possible to create a new tag. Make sure the autotag action looks like the following example:

```yaml
- uses: eneskaraosman/action-autotag@stable
  with:
    GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

The action will automatically extract the token at runtime. **DO NOT MANUALLY ENTER YOUR TOKEN.** If you put the actual token in your workflow file, you'll make it accessible (in plaintext) to anyone who ever views the repository (it will be in your git history).

### Optional Configurations

There are several options to customize how the tag is created.

1. `package_root`

    By default, autotag will look for the `pubspec.yaml` file in the project root. If the file is located in a subdirectory, this option can be used to point to the correct file.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        package_root: "/path/to/subdirectory"
    ```

2. `tag_prefix`

    By default in flutter, the version number is three numbers separated by dots, followed by an optional build number separated by a +, such as `1.2.21+7`. **The optional build number is ignored when the tag is created**. A prefix can be used to add text before the tag name. For example, if `tag_prefix` is set to `v`, then the tag would be labeled as `v1.0.0`.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_prefix: "v"
    ```

3. `tag_suffix`

    Text can also be applied to the end of the tag by setting `tag_suffix`. For example, if `tag_suffix` is `(beta)`, the tag would be `1.0.0(beta)`. Please note this example violates semantic versioning and is merely here to illustrate how to add text to the end of a tag name if you _really_ want to.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_suffix: "(beta)"
    ```

4. `tag_message`

    This is the annotated commit message associated with the tag. By default, a
    changelog will be generated from the commits between the latest tag and the new tag (HEAD). Setting this option will override it witha custom message.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_message: "Custom message goes here."
    ```

5. `changelog_structure`

    Provide a custom changelog format when not using `tag_message`.
    This can interpolate strings, supported strings are `{{message}}`, `{{messageHeadline}}`, `{{author}}` and `{{sha}}`.
    Defaults to `**{{message}}** {{sha}})\n`.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        changelog_structure: "**{{messageHeadline}}** {{author}}\n"
    ```

6. `version`

    Explicitly set the version instead of automatically detecting from `pubspec.yaml`.
    Useful for non-Flutter projects where version may be output by a previous action.

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        version: "${{ steps.previous_step.outputs.version }}"
    ```

7. `include_build_number`

    `pubspec.yaml` file may contain build number after "+" character. Like `1.5.0+53`. The build number is added to the tag by default, to ignore it set this parameter to false

    ```yaml
    - uses: eneskaraosman/action-autotag@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        include_build_number: false
    ```

## Developer Notes

If you are building an action that runs after this one, be aware this action produces several [outputs](https://help.github.com/en/articles/metadata-syntax-for-github-actions#outputs):

1. `tagname` will be empty if no tag was created, or it will be the value of the new tag.
2. `tagsha`: The SHA of the new tag.
3. `taguri`: The URI/URL of the new tag reference.
4. `tagmessage`: The messge applied to the tag reference (this is what shows up on the tag screen on Github).
5. `version` will be the version attribute found in the `pubspec.yaml` file.

---

## Credits

Forked by [jacopocarlini](https://github.com/jacopocarlini)

This action was originally created by [Corey Butler](https://github.com/coreybutler).
