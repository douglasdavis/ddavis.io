## Personal website

GitHub pages hosted personal website: https://ddavis.fyi/

## Serving

To serve locally (requires [bundler](http://bundler.io/)).

```
make install ## creates vendor directly, runs bundle install
make serve   ## runs bundle exec jekyll serve
```

To build including drafts (living in `content/_drafts`):

```
make servedrafts
```
