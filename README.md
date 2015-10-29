## Development

Clone the repo:

```shell
$ git clone --recursive https://github.com/agis-/agis.io.git
```

Install the dependencies:

```shell
$ bundle
$ brew install npm && npm install cleancss # only needed for minifying assets
```

Build the site and minify the assets:

```shell
$ bundle exec rake
```

For more tasks:

```shell
$ bundle exec rake -T
```
