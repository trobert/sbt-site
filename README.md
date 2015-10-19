# sbt-site
[![Build Status](https://travis-ci.org/sbt/sbt-site.svg)](https://travis-ci.org/sbt/sbt-site)

[![Join the chat at https://gitter.im/sbt/sbt-site](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/sbt/sbt-site?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[ ![Download](https://api.bintray.com/packages/sbt/sbt-plugin-releases/sbt-site/images/download.svg) ](https://bintray.com/sbt/sbt-plugin-releases/sbt-site-imported/_latestVersion)

This sbt plugin generates project websites from static content, [jekyll], [sphinx], [pamflet], [nanoc], and/or [asciidoctor], and can optionally include generated scaladoc. It is designed to work hand-in-hand with publishing plugins like [sbt-ghpages].

## Usage

`sbt-site` is deployed as an `AutoPlugin`. To enable, simply add the following to  your `project/plugins.sbt` file:

```
addSbtPlugin("com.typesafe.sbt" % "sbt-site" % "1.0.0")
```

> _Note_: As of version 1.0.x, sbt >= 0.13.5 is required.  
> * For earlier 0.13.x releases, use [version 0.8.2][0.8.2].
> * For sbt 0.12, use [version 0.7.2][0.7.2].
> * To upgrade from a previous version, see the **[migration guide]**.

See the [sbt-ghpages] plugin for information about publishing to [GitHub Pages]. We expect other publishing mechanisms to be supported in the future.

## Adding Static Content to Your Site
When you run `makeSite`, your project's webpage is generated in the `target/site` directory. By default, all files under `src/site` are included in `target/site`. To use specific third-party generators (e.g. [Jekyll]), additional sub-plugins will need to be enabled, as described below.

The `src/site` directory can be overridden via the `siteSourceDirectory` key:

```
siteSourceDirectory <<= target / "generated-stuff"
```

Additional files outside of `siteSourceDirectory` can be added individually via the `siteMappings` key:

```
siteMappings ++= Seq(file1 -> "location.html", file2 -> "image.png")
```

### Static Content with Variable Substitution
The site plugin supports basic variable substitution when copying files from `src/site-preprocess`. To enable, add this to your `build.sbt` file:

```
enablePlugins(PreprocessPlugin)
```

Variables are delimited with surrounding `@` symbols (e.g. `@VERSION@`). Values are assigned to variables via the `preprocessVars[Map[String, String]]` setting. For example:

```
preprocessVars := Map("VERSION" -> version.value, "DATE" -> new Date().toString)
```

The setting `preprocessIncludeFilter` is used to define the filename extensions that should be processed when `makeSite` is run. The default filter is `"*.txt" | "*.html" | "*.md" | "*.rst"`.

### Jekyll Site Generation
The `sbt-site` plugin has direct support for running [Jekyll]. This is surprisingly useful for supporting custom Jekyll plugins that are not allowed when publishing to GitHub, or hosting a Jekyll site on your own server. To add Jekyll support, enable the associated plugin:

```
enablePlugins(JekyllPlugin)
```

This assumes you have a jekyll project in the `src/jekyll` directory. To change this, set the key `sourceDirectory` in the `Jekyll` scope:

```
sourceDirectory in Jekyll := sourceDirectory / "hyde"
```

To redirect the output to a subdirectory of `target/site`, use the `siteSubdirName` key in `Jekyll` scope:

```
// Puts output in `target/site/notJekyllButHyde`
siteSubdirName in Jekyll := "notJekyllButHyde"
```

One common issue with Jekyll is ensuring that everyone uses the same version for generating a website.  There is special support for ensuring the version of gems. To do so, add the following to your `build.sbt` file:

```
requiredGems := Map(
  "jekyll" -> "0.11.2",
  "liquid" -> "2.3.0"
)
```

### Sphinx Site Generation
The `sbt-site` plugin has direct support for building [Sphinx] projects. To enable Sphinx site generation, simply enable the associated plugin in your `build.sbt` file:

```
enablePlugins(SphinxPlugin)
```

This assumes you have a Sphinx project under the `src/sphinx` directory. To change this, set the `sourceDirectory` key in the `Sphinx` scope:

```
sourceDirectory in Sphinx := sourceDirectory / "androsphinx"
```

Similarly, the output can be redirected to a subdirectory of `target/site` via the `siteSubdirName` key in `Sphinx` scope:

```
// Puts output in `target/site/giza`
siteSubdirName in Sphinx := "giza"
```

### Pamflet Site Generation

The `sbt-site` plugin has direct support for building [Pamflet] projects. To enable sphinx site generation, simply enable the associated plugin in your `build.sbt` file:

```
enablePlugins(PamfletPlugin)
```

This assumes you have a sphinx project under the `src/sphinx` directory. To change this, set the `sourceDirectory` key in the `Sphinx` scope:

```
sourceDirectory in Pamflet := sourceDirectory / "papyrus"
```

Similarly, the output can be redirected to a subdirectory of `target/site` via the `siteSubdirName` key in `Sphinx` scope:

```
// Puts output in `target/site/parchment`
siteSubdirName in Sphinx := "parchment"
```

----

### Nanoc Site Generation
The site plugin has direct support for building [nanoc][nanoc] projects. This assumes you have a nanoc project under the `src/nanoc` directory. To enable nanoc site generation, simply simply add the following to your `build.sbt` file:

```
site.nanocSupport()
```

### Asciidoctor Site Generation
The site plugin has direct support for building [Asciidoctor][asciidoctor] projects locally. This assumes you have a asciidoctor project under the `src/asciidoctor` directory. To enable asciidoctor site generation, simply add the following to your `build.sbt` file:

```
site.asciidoctorSupport()
```

## Scaladoc APIS
To include Scaladoc with your site, add the following line to your `build.sbt`:

```
site.includeScaladoc()
```

This will default to putting the scaladoc under the `latest/api` directory on the website. You can configure this by passing a parameter to the `includeScaladoc` method:

```
site.includeScaladoc("alternative/directory")
```

## Previewing the Site
To preview your generated site, run `previewSite` to open a web server. Direct your web browser to [http://localhost:4000/](http://localhost:4000/) to view your site.

## Packaging and Publishing
To create a zip package of the site run `package-site`.

To also publish this zip file when running `publish`, add the following to your `build.sbt`:

```
site.publishSite
```

## License

`sbt-site` is released under a "BSD 3-Clause" license. See [LICENSE](LICENSE) for specifics and copyright declaration.

[0.7.2]: https://github.com/sbt/sbt-site/tree/v0.7.2
[0.8.2]: https://github.com/sbt/sbt-site/tree/v0.8.2
[migration guide]: notes/migrate-0.8.2-to-1.0.md
[sbt-ghpages]: http://github.com/sbt/sbt-ghpages
[jekyll]: http://jekyllrb.com
[pamflet]: http://pamflet.databinder.net
[nanoc]: http://nanoc.ws/
[asciidoctor]: http://asciidoctor.org
[sphinx]: http://sphinx-doc.org
[GitHub Pages]: https://pages.github.com
