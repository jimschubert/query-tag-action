# query-tag-action

A GitHub Action allowing users to query tags via `git describe`.

This is useful, for example, if you want to generate a changelog from your last release tag to your newest. Rather than apply that logic to multiple changelog tools, this action will set an output parameter to the last found tag. You should then be able to use that parameter in any changelog utility you'd like.

This action acts as a wrapper around [`git describe`](https://git-scm.com/docs/git-describe), performing an `unshallow` operation on the git repository by default.

The `actions/checkout@v2` action will check out a shallow repository. Although that action's [readme](https://github.com/actions/checkout) documents how to unshallow, many users will never read that readme. For those savvy users who are already performing an unshallow, you may skip that operation in this action by passing the input `skip-unshallow: "true"`.

## Inputs

### `include`

**Optional** [Glob](http://man7.org/linux/man-pages/man7/glob.7.html) pattern of tags to include

The include option is passed to the `git describe --match` option. It is renamed in this action to complement `exclude` because I felt that include/exclude made more sense than match/exclude (Sorry, Linus).

### `exclude`

**Optional** [Glob](http://man7.org/linux/man-pages/man7/glob.7.html) pattern of tags to exclude

The exclude pattern may be of use for those projects which have beta or rc type tags. This allows one to define "Release" patterns via `include` and skip over non-release patterns via `exclude`. For the changelog use case, this allows release changelogs where history is not terminated by release candidate tags.

### `commit-ish`

**Optional** [Commit-ish](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/#_identifier_terminology) object name(s) to describe.

Default: `HEAD~`

A `commit-ish` value is any which may point to a commit in git. You're probably already familiar with these (`HEAD`, `HEAD~`, `HEAD^`, `@{ 2 weeks ago }`, etc.). Check git's documentation for more details.

By default, `git describe` will point to `HEAD` which may result in the current tag on release builds. A user may want the current tag without manipulating `GITHUB_REF` as provided by [GitHub Actions Environment Variables](https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables); pass `commit-ish: "HEAD"` as an input and ignore the warning logged by this action.

### `skip-unshallow`

**Optional** Skip the unshallow operation: "true" or "false"

Default: `false`

This option allows for users who have already performed an unshallow operation to skip the additonal unshallow in this action. This is marked as optional because it will use default behavior if unspecified. 


**NOTE** If you have already performed an unshallow in a previous step, you must pass `skip-unshallow: "true"` in any `query-tag-action` step following that unshallow. A future version may check and handle this logic gracefully without user input.

## Outputs

### `tag`

The tag determined by your inputs.

## Example usage

uses: jimschubert/query-tag-action@v1
with:
  who-to-greet: 'Mona the Octocat'


```yaml
name: Tagged
on: [release]

jobs:

  my_job:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2
    # Optionally: unshallow as a separate operation
    # - name: Unshallow
    #   run: git fetch --prune --unshallow
    - name: Find Tag
      id: tagger
      uses: jimschubert/query-tag-action@v1
      with:
        include: 'v*'
        exclude: '*-rc*'
        commit-ish: 'HEAD~'
        # if you unshallow in a separate step, use the following option:
        # skip-unshallow: 'true'
    - name: Show Tag
      id: display
      run: |
        echo 'Output from Find Tag: ${{steps.tagger.outputs.tag}}'
```

### Testing

No automated tests are provided here because this action just builds a standard `git describe` command line invocation.

The example from above will produce the following command:

```bash
git fetch --prune --unshallow && git describe --tags --abbrev=0 --match 'v*' --exclude '*-rc*' HEAD~
```

The above command is built by the following parts:

* `skip-unshallow` determines whether to include the `git fetch --prune --unshallow && ` command part
* `include` determines whether to generate `--match` and defines the value `v*`
* `exclude` determines whether to generate `--exclude` and defines the value `*-rc*`
* `commit-ish` determines whether to generate the command option `HEAD~`.

Please see [tagged.yml](./.github/workflows/tagged.yml) for some use cases.

## License

This project is [licensed](./LICENSE) under Apache 2.0.
