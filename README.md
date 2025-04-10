# gh-pages-multibranch

This action allows you to deploy multiple branches in a single github
pages site, by using this arrangement:

```
/...default branch files...
/branch/branch1/...branch1 files...
/branch/other-branch/...other-branch files...
```

* On each push to the default branch, put the files in ``gh-pages:/``.
* On pushes to other branches, put the files in
  ``gh-pages:/branch/BRANCH_NAME/``
* It does this by extracting the old ``gh-pages`` branch content,
  removing any outdated content (the old branch or any branches that
  have been deleted since the last run), and adding in the new
  content.
* It doesn't deploy to gh-pages for you (other actions can do that),
  but it drops the remade files into the given ``directory``
  * To put this another way: The *directory* action argument contents gets
    replaced with the multiplexed gh-pages branch content.  You then
    use a different action to deploy this.

## Usage

```yaml
name: sphinx
on: [push, pull_request]  # No need to limit to certain branches
jobs:
  your-job:
    steps:
      [ checkout]
      [ place your built site in ./public ]
      [ add CNAME if you use it, to autodetect it in the next step ]

      - name: multipages
        uses: coderefinery/gh-pages-multibranch@main
        if: ${{ github.event_name == 'push' }}
        with:
          directory: public
      # Remember to not limit your deploy step to the default branch!
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.event_name == 'push' }}
        with:
          publish_branch: gh-pages
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: public/
          force_orphan: true
```

Surprising points:
* Make sure you don't limit your workflow to only your default
  branch.  `on: [push, pull_request]` works fine since other branches
  won't get published to `/`.


## Options

Used with `with:` as above.

* `directory`: relative path to the directory that contains the HTML
  site to be multiplexed and published.
  (The contents are replaced with the multi-branch site - so this
  directory is both the input to this action, and the input to the
  action that publishes it to the gh-pages branch)
* `cname`: this CNAME is used for generating the preview links.  It is
  tried to be auto-detected from the `directory` by default, if it is
  already there.  (CNAME is needed since some preview might be
  ORG.github.io/[repo/] and some might be CNAME.domain/.
* `default_branch`: The branch name that gets the top-level site.
  Default `main`.
* `publish_branch`: The branch to be pushed to for gh-pages, default
  `gh-pages`.  It is *not* automatically pushed there, you need to
  publish there in a separate step.  This is used to mix the old and
  new pages.

## See also

* https://github.com/peaceiris/actions-gh-pages - a good Github Pages
  publishing action.
