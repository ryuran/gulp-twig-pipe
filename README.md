# gulp-twig-pipe
Gulp plugin that runs JSON files through a Twig template, converting them to HTML

> fork of [gulp-hbs](https://www.npmjs.com/package/gulp-hbs)

## Install

```
   npm install gulp-twig-pipe --save-dev
```

## Usage

```js
    var twig = require('gulp-twig-pipe');

    gulp.src('*.json')
        .pipe(twig('template.html.twig')))
        .pipe(gulp.dest('html'));
```

You can also use other gulp streams to describe your template(s).

```js
    var twig = require('gulp-twig-pipe');

    gulp.src('*.json')
        .pipe(twig(gulp.src('templates/**/*.html.twig')))
        .pipe(gulp.dest('html'));
```

In that case, the stream must either emit exactly one template file,
or the data must contain a `template` key which will choose among the
different templates, using their relative path as a key.

## API

### twig(templateSrc[, options])

templateSrc: Path to a twig template,
or gulp (or vinyl) stream of template files.

The following options are supported:

* **cache** (default `true`, only applicable if `templateSrc` is a string).

  If set to `true`, multiple references to the same tmplate will use a
  cached version of the compiled template.  The option may also be set
  to an object which will then be used as the cache.

  Not using the global cache, as is the default, is important in tasks
  intended to be run from a `gulp.watch` call if the template itself
  may be modified as well.

* **defaultTemplate** (only applicable if `templateSrc` is a stream,
  defaults to the sole file in that stream).

  If there are multiple templates, then sources with no template specified
  will use this template by default.

* **dataSource** (defaults to `'json'`).

  In `json` mode, input is assumed to represent a JSON-encoded file,
  which will be used as the data source for the template.
  This is useful for data read from `json` files or from plugins like
  `gulp-dox` or `markit-json`.

  In `vinyl` mode, the whole file object with all the attributes
  other plugins might have added to it will be accessible.
  The original file contents, converted to a string, will be contained
  in the attribute indicated by `bodyAttribute`.

  In `data` mode, the `data` attribute of the file object will be used.
  The original content will again be made accessible under the
  attribute indicated by `bodyAttribute`.

  The option may also be set to a function, which will take a vinyl
  file and return the data object to be fed into the compiled
  template for rendering.

* **bodyAttribute** (defaults to `'body'`).

  If `dataSource` is something other than `json`, this option
  indicates under what attribute the original file content will be
  made available.

* **templateAttribute** (defaults to `'template'`,
  only applicable if `templateSrc` is a stream).

  This attribute is used to choose a template from those in the
  template stream.  Nested attribute names (separated by dots) are
  allowed.

* **compile** (defaults to `Twig.twig`).

  Function used to compile each template.
  This function will accept a string and return a function.
  The returned function must turn the provided data into
  the rendered output used by applying the tempate to the data.

### twig.extend((Twig) => {})

Extend twig.

### twig.twig

This is the underlying Twig object used by this plugin.
Using this reference, you can use the full Twig.js API
and be certain to affect the module used by the plugin.

## Cooperation

The plugin is designed to cooperate with various other plugins.

### gulp-dox

To document JavaScript sources, you can use
[Dox](https://github.com/tj/dox) in a pipeline like this
using the [gulp-dox](https://github.com/ayhankuru/gulp-dox) plugin:

```js
    var dox = require('gulp-dox');
    var twig = require('gulp-twig-pipe');

    gulp.src('src/**/*.js')
        .pipe(dox())
        .pipe(twig('api-template.html.twig')))
        .pipe(gulp.dest('docs'));
```

### markit-json

You can use a Markdown-generated body with YAML front matter
using the [markit-json](https://github.com/Minwe/markit-json) plugin.
Use the default `dataSource:'json'` configuration,
and refer to the body using `{{{body}}}` in your templates.

```js
    var markdown = require('markit-json');
    var twig = require('gulp-twig-pipe');

    gulp.src('*.md')
        .pipe(markdown())
        .pipe(twig(gulp.src('*.html.twig'), {defaultTemplate: 'default.html.twig'}))
        .pipe(gulp.dest('html'));
```

### gulp-front-matter

Another way to handle front matter is using the
[gulp-front-matter](https://github.com/lmtm/gulp-front-matter) plugin.
There are two ways of using this.
It's probably best to configure `gulp-front-matter` using `property: 'data'`.
Then you can use `dataSource: 'data'` for gulp-twig to access the front matter.

```js
    var fm = require('gulp-front-matter');
    var twig = require('gulp-twig-pipe');

    gulp.src('src/**/*.html')
        .pipe(fm({property: 'data'}))
        .pipe(twig(gulp.src('templates/*.html.twig'), {dataSource: 'data'}))
        .pipe(gulp.dest('html'));
```

If you want to stick to the default property name, you can use the
`vinyl` data source to make all of the file object available.
In this case, you have to list the `frontMatter` path element
of your data structure explicitely in your twig template.

```js
    var fm = require('gulp-front-matter');
    var twig = require('gulp-twig-pipe');

    gulp.src('src/**/*.html')
        .pipe(fm())
        .pipe(twig(gulp.src('templates/*.html.twig'), {
            dataSource: 'vinyl',
            templateAttribute: 'frontMatter.template'}))
        .pipe(gulp.dest('html'));
```


### gulp-data

The [gulp-data](https://github.com/colynb/gulp-data) plugin provides
a flexible way to use the `data` property of the vinyl file to hold
additional data to accompany any document.
Use the `dataSource: 'data'` configuration to access that.

```js
    var data = require('gulp-data');
    var twig = require('gulp-twig-pipe');

    gulp.src('src/**/*.html')
        .pipe(data(function(file) { return obtainData(file); }))
        .pipe(twig(gulp.src('templates/*.html.twig'), {dataSource: 'data'}))
        .pipe(gulp.dest('html'));
```
