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
is installed system-wide.

```bash
mise trust        # once per clone, to allow mise.toml
bundle install
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000/blog/>.

A few things worth knowing:

* The `/blog/` path is required. `baseurl` is set in `_config.yml`, so the bare root returns a 404.
* `bundle exec jekyll serve` rebuilds on save, but not for changes to `_config.yml` — restart for those.
* Posts in `drafts/` are excluded from the build and will not appear.
* The build prints Dart Sass deprecation warnings from the minima theme. They are expected and can be
  ignored.

Nothing lints or tests post content, so run `bundle exec jekyll build` before pushing. A stray `{{` or
`{%` in a post — common in Erlang and Elixir terms — is parsed as a Liquid tag and fails the whole
build. Wrap those code blocks in `{% raw %}` / `{% endraw %}`.



## References

* [Viewing YAML Metadata in your Documents](https://github.blog/2013-09-27-viewing-yaml-metadata-in-your-documents/)

## About Me

* Follow me on [Twitter](http://www.twitter.com/nicholasjhenry)
* Connect via [LinkedIn](http://ca.linkedin.com/in/nicholasjhenry)
