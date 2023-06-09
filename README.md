<p align="center">
  <a href="https://github.com/jsmith/changes-since-last-tag/actions"><img alt="typescript-action status" src="https://github.com/jsmith/changes-since-last-tag/workflows/test/badge.svg"></a>
</p>

> **WARNING: The GitHub list tags API was used to fetch the tags. It was assumed they were in chronological order; however, they are actually in alphabetical order. Using regular `v.X.X.X` versioning still works; however, if they are NOT alphabetical order this will not work as expected. More information can be found about this issue [here](https://stackoverflow.com/questions/19452244/github-api-v3-order-tags-by-creation-date).**

# changes-since-last-tag

Do you do deployments when you push a new tag? Do you ever have multiple deployments in single repository? Do some of those deployments take a long time? Could some of these deployments be sometimes skipped if certain files haven't changed? If you answered yes to all of these questions, this action might be for you.

## Inputs

> NOTE: In addition to the inputs defined below, you can also pass in any of the minimatch options defined [here](https://github.com/isaacs/minimatch#options). For example, if you want to see the `dot` property, set `dot: true`.

### glob

**description**: The glob(s) of the files to check for changes (uses [`minimatch`](https://github.com/isaacs/minimatch)). All file changes that don't match at least one of the globs are filtered out. If you want to provide multiple globs, use a `,` between each glob. Defaults to `**`.  
**required**: false  
**example**: `src/**`  
**example**: `**`  
**example**: `*.js,*.py`

## Outputs

Ensure to set the step ID using the `id` attribute. See the [docs](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsid).

### steps.<STED_ID>.outputs.files

**description**: The names of all of the added, modified, removed and renamed files (comma separated).  
**example**: `a.txt, src/b.txt`

### steps.<STED_ID>.outputs.added

**description**: The names of the newly created files (comma separated).  
**example**: `a.txt, src/b.txt`

### steps.<STED_ID>.outputs.modified

**description**: The names of the updated files (comma separated).  
**example**: `a.txt, src/b.txt`

### steps.<STED_ID>.outputs.removed

**description**: The names of the removed files (comma separated).  
**example**: `a.txt, src/b.txt`

### steps.<STED_ID>.outputs.renamed

**description**: The names of the renamed files (comma separated).  
**example**: `a.txt, src/b.txt`

### steps.<STED_ID>.outputs.any_changed

**description**: Whether there were any files changes. This will always be `false` when this action runs on the first tag (as there is nothing to compare this tag to).  
**example**: `true`

### steps.<STED_ID>.outputs.first_tag

**description**: Whether this is the first tag.  
**example**: `false`

## Example Usage

You have repository that contains both the Android files (`android/`) and iOS files (`ios/`). Your GitHub action(s) contain two builds, one for iOS and one for Android. Sometimes, you only need to update your iOS application or vice versa. If one of these situations occur, you want to make sure only the builds that need to run actually do run.

```yaml
# This is required. This *must* only be run when a tag is pushed.
on:
  push:
    tags:
      # This is a catch all pattern but you can define your own pattern
      - '*'

jobs:
  deployment:
    runs-on: ubuntu-latest
    steps:
      - id: android_changes
        with:
          glob: android/**
        uses: jsmith/changes-since-last-tag@v0.3.3

      - id: ios_changes
        with:
          glob: ios/**
        uses: jsmith/changes-since-last-tag@v0.3.3

      # The == 'true' is important since we can only output strings
      # If you forgot this, the build will always run... I think
      - if: steps.android_changes.outputs.any_changed == 'true'
        run: make android # just an example command

      # Same as above except for ios
      - if: steps.ios_changes.outputs.any_changed == 'true'
        run: make ios

# This is an alternative method where this action just once
# This method *could* lead to false positives but it is unlikely
jobs:
  deployment:
    runs-on: ubuntu-latest
    steps:
      - id: changes
        uses: jsmith/changes-since-last-tag@v1

      - if: contains(steps.changes.outputs.files, 'android/')
        run: make android # just an example command

      - if: contains(steps.changes.outputs.files, 'ios/')
        run: make ios
```

## Deployment

These steps detail how to release a new version of `@jsmith/changes-since-last-tag`.

1. Make and commit your changes.
2. Run `npm version COMMAND` with the appropriate `COMMAND`.
3. Push everything using `git push && git push --tags`.
4. Publish a new release on `github.com`.
