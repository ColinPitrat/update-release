# Create/Update release and upload assets

**Deprecated**: use https://github.com/softprops/action-gh-release instead

An action to create a release or update it if it exists and publish assets into
it. This ensures the release and tag are updated so that the release date is
updated.

This is typically aimed at updating a `latest` or `nightly` release, possibly
from different workflows.
This allows the release to always be available (never deleted) while always
being up-to-date (the tag is updated to so the date shown by GitHub corresponds
to the latest asset uploaded).

*Drawbacks*:
 - Assets must be manually deleted if you stop building them. 
 - Assets can be desynchronized when using multiple workflows if some are
   failing while others succeed.

- [update-release](#update-release)
  - [Quick start](#quick-start)
    - [For builds lasting more than an hour](#for-builds-lasting-more-than-an-hour)
  - [Summary](#summary)
  - [Guide](#guide)
  - [Inputs](#inputs)
    - [token](#token)
    - [files](#files)
    - [release](#release)
    - [tag](#tag)
    - [message](#message)
    - [body](#body)
    - [prerelease](#prerelease)
    - [draft](#draft)
  - [Outputs](#outputs)
    - [files](#files-1)
    - [draft](#draft-1)
    - [prerelease](#prerelease-1)
    - [release](#release-1)
    - [tag](#tag-1)
  - [Internals](#internals)
    - [security concerns](#security-concerns)
  - [Problems?](#problems)

## Background

This is a fork of [johnwbyrd/update-release](https://github.com/johnwbyrd/update-release)
which is Archived so I can't submit changes to it.

Deprecated: use https://github.com/softprops/action-gh-release instead

## Quick start

Insert the following into the appropriate step in your `.github/workflows/*.yml`
file:

    - name: Update release
      uses: ColinPitrat/update-release@v1.0.1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        files: ./file-to-release.zip dist/other-file-to-release.exe README.md

### For builds lasting more than an hour

The `${{ secrets.GITHUB_TOKEN }}` is valid for exactly an hour from the time
your build starts.  If your build requires longer than an hour to run, you will
need to [create your own access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)
with repo admin access, [store it as a secret](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)
in your own repository, and reference that secret token in your build:

    - name: Update release
      uses: ColinPitrat/update-release@v1.0.1
      with:
        token: ${{ secrets.YOUR_PRIVATE_SECRET_TOKEN }}
        asset: ./the-file-you-want-to-release.zip

## Summary

[This Github action](https://www.github.com/ColinPitrat/update-release) allows
you, to publish files created by your GitHub Actions as assets in new or
existing releases. As it does so, it updates the release so that the tag and
date match the last released asset.

Note: if you publish assets to the same release from different workflows, it is
up to you to ensure that all the assets are updated and actually match the tag.

Because this action is written in [TypeScript](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) 
and executes in [node.js](https://nodejs.org/en/), it runs on *all* Github's
supported build runner platforms. These include Windows, MacOS, and Ubuntu as of
this writing.

## Guide

Once your build has successfully completed, update-release will choose a release
name for your build.  Regardless of whether the ref that triggered the build is
a tag or a branch, you'll get a human-friendly release name.  You can of course
override the default choice. If the Github release name already exists, it is
reused; otherwise, it is created.

## Inputs

The following parameters are accepted as inputs to the update-release action.

### token

This should be [your secure Github token](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token).
Use `${{ secrets.GITHUB_TOKEN }}` as the parameter if your build lasts less than an hour.

If your build lasts more than an hour, you will need to [create your own access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)
with repo admin access, [store it as a secret](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)
in your own repository, and reference that secret token in your build.

This parameter is **required**.

### files

The paths to files that you wish to add to the release.
Presumably, this should include at least one file that you just built.
File paths can be provided as absolute paths, or they can alternately be
relative to `${{ github.workspace }}`.  

This parameter is **required**.

### release
    
The name of the release to be created. A reasonable-looking release name will
be created from the current `${{ github.ref }}` if this input is not supplied.
This reasonable looking default is created by taking `${{ github.ref }}`,
removing the prefixes `refs/`, `heads/`, and `tags/` , and then replacing any
remaining forward-slash symbols `/` with dashes `-`.

If you don't like this behavior, just override the release name here yourself.

This parameter is **optional**.

### tag

The name of the tag to be created. For some inexplicable reason, Github thinks
that you need to have a tag corresponding to every release, which makes no
sense if you're using Github to do continuous integration builds.  The tag will
be the same as the calculated name of the release, if this input is not
supplied. 

This parameter is **optional**.

### message

A brief description of the tag and also of the release.

This parameter is **optional**.

### body

A longer description of the release, if it is created.

This parameter is **optional**.

### prerelease

Should the release, if created, be marked as a prerelease?
Such releases are generally publicly visible.
Provide true or false as a parameter.

This parameter is **optional**.
The default setting is `false`.

### draft

Should the release, if created, be marked as a draft?  Such releases are
generally not publicly visible.  Provide true or false as a parameter.

This parameter is **optional**.
The default setting is `false`.
    
## Outputs

If assets are successfully published, you will get the following outputs from
the step, which you can use in later processing.

The following parameters are provided as outputs to this Github action.

### files

The calculated local paths of the files to be uploaded into the release.

### draft

Whether the release, if created, was marked as a draft.

### prerelease

Whether the release, if created, was marked as a prerelease.

### release

The name of the release.

### tag

The tag used to create the release.

## Internals

### Setup

To build under Debian (should be easy to adapt):

```
apt-get install webpack npm
npm install --save-dev typescript ts-loader v8-compile-cache
npm run bundle
```

### Details

This Github action was written for [node.js](https://nodejs.org/en/) in
[TypeScript](https://github.com/ColinPitrat/update-release), and it uses
[webpack](https://webpack.js.org/) in order to run
[ESLint](https://eslint.org/) before bundling.
Use [npm install](https://docs.npmjs.com/cli/install) to install all
package.json dependencies of update-release, before hacking on it.

Several [npm](https://www.npmjs.com/) targets were added to speed along
development.  The `test` run target builds readable `dist/main.js` and
`dist/main.map.js` files, for source-level debugging of the TypeScript.  A
`test-watch` run target watches the `src/main.ts` file for changes, and lints
and recompiles it as needed.  And, a `bundle` run target prepares a production
minified `dist/main.js`.

This action uses the `dotenv` import in order to facilitate debugging.  This
import reads a `.env` file, if it exists, as the root of the installation, and
uses it to populate environmental variables for local testing of update-release.
A typical `.env` file for developing update-release, might look something like
this:

    INPUT_ASSET=your-build-asset.zip
    INPUT_TOKEN=00000000000000000000000000000001
    GITHUB_REPOSITORY=you/your-repo
    GITHUB_REF=refs/heads/master
    GITHUB_WORKSPACE=/absolute/local/path/to/workspace

Using an `.env` file, you can perform local testing and debugging of
update-release without having to build a product first.

### Security concerns

You *should* review the source code of this -- and *all other!* -- Github
actions, to verify that they don't store, transmit, or otherwise mistreat your
secure Github token.

When you provide your secure access token to any Github action, you're
essentially giving the code in that action permission to do whatever it wants to
your repository.  *Don't* just hand over your security tokens to any Github
actions, for which the sources are not available!

For extra peace of mind, please review `src/main.ts` *in detail*; also, feel
free to rebuild it using `npm run bundle`, to make sure that the minified
`dist/main.js` corresponds exactly to the one in the repo.

As an extra protection, `src/main.ts` tries to helpfully mark the token that you
provide it as a secret, so that it doesn't inadvertently sneak into any log
files.

## Problems?

I welcome all patches and improvements as pull requests against
[this repository](https://github.com/ColinPitrat/update-release).
