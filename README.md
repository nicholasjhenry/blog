# Nicholas Henry Blogs

This is a central repository of my blog posts published across a number of platforms including:

* https://medium.com/@nicholasjhenry
* https://gist.github.com/nicholasjhenry (Code Snippets only)
* http://blog.firsthand.ca (deprecated)

The archive is also published as a Jekyll site on GitHub Pages at
<https://nicholasjhenry.github.io/blog/>. Each post keeps a `canonical_url` pointing at the platform
it was originally published on, so the site is a mirror rather than the source of record.

## Previewing locally

Ruby is pinned by `mise.toml`, and gems install into `vendor/bundle` rather than globally, so nothing
is installed system-wide. The tasks below are defined in `mise.toml`; run `mise tasks` to list them.

```bash
mise trust        # once per clone, to allow mise.toml
mise run serve    # installs gems if needed, then serves
```

Then open <http://127.0.0.1:4000/blog/>.

A few things worth knowing:

* The `/blog/` path is required. `baseurl` is set in `_config.yml`, so the bare root returns a 404.
* `serve` rebuilds on save, but not for changes to `_config.yml` — restart for those.
* `--incremental` does not regenerate the home index when a post or draft is added, so a new entry
  appears at its own URL but not in the list on the front page. Leave it off when adding a post.
* Posts in `_drafts/` are skipped by `serve` and will not appear. Use `drafts` for those.
* The build prints Dart Sass deprecation warnings from the minima theme. They are expected and can be
  ignored.

Nothing lints or tests post content, so run `mise run build` before pushing. A stray `{{` or
`{%` in a post — common in Erlang and Elixir terms — is parsed as a Liquid tag and fails the whole
build. Wrap those code blocks in `{% raw %}` / `{% endraw %}`.

## References

* [Viewing YAML Metadata in your Documents](https://github.blog/2013-09-27-viewing-yaml-metadata-in-your-documents/)

## About Me

* Follow me on [Twitter](http://www.twitter.com/nicholasjhenry)
* Connect via [LinkedIn](http://ca.linkedin.com/in/nicholasjhenry)
