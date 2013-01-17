---
layout:    post
title:     Jekyll on Heroku Without Rack::Jekyll or Custom Buildpacks
published: true
tags:
  - code
  - Heroku
  - Jekyll
  - Ruby
---

For a while now, this site has been hosted on [Heroku][] as static files
generated by [Jekyll][], which is much faster than either my old
WordPress blog or my previous Rails-based site (even with caching). Most
sets of instructions for deploying Jekyll sites to Heroku either say to
use [Rack::Jekyll][] and commit the site's compiled static files into
Git, or to use a [custom Heroku buildpack][] to compile the site at
deploy time. I wanted to stick as close to the default Heroku setup as
possible without cluttering the repository with compiled pages, and
Rack::Jekyll is unnecessarily complicated when [Rack::TryStatic works
just fine][].
<!-- more -->

## Compiling on deployment

[Heroku's Ruby buildpack][] looks by default for the Rake task
`assets:precompile` and runs it if it finds it. This is typically for
the [Rails asset pipeline][] to precompile [CoffeeScript][] files,
[Compass][] stylesheets, &c., but any site that uses the Ruby buildpack
can add a task with the same name to its Rakefile and have it run every
time the site is pushed to Heroku. This is not only perfect for
compiling Jekyll pages, but also allows the use of Compass,
CoffeeScript, or any other such precompiler we want[^precompilers]. For
example, to compile Jekyll files and Compass stylesheets[^compass]:

{% gist 4551206 Rakefile %} 

## Serving from Rack

Once they've been compiled by Heroku, the static files are served up by
[Unicorn][] and a pair of [Rack::Contrib][] modules: `try_static` and
`not_found`. `try_static` lets Rack serve up static HTML files and
tries to match specified paths, and `not_found` tells Rack to serve a
specific file in the event of a 404 error. Add `rack_contrib` to the
`Gemfile` and write a Rackup file as follows:

{% gist 4551206 config.ru %}

Finally, create a `Procfile` with a web task for Heroku to run your
server of choice, such as Unicorn[^unicorn]:

{% gist 4551206 Procfile %}

The [entire source of this site][] is available on GitHub, so you can
see there how I've [configured Unicorn][], my [Compass settings][],
[Gemfile][], &c., as a basis for your own deployment.

## Footnotes

[^precompilers]: Using Liquid tags and Jekyll syntax in files requires
    using a [converter plugin for Jekyll][] and adding the Jekyll YAML
    front matter to the files to be converted.

[^compass]: Assuming you have added `compass` to your `Gemfile` and
    configured it with a `compass.rb` file. These stylesheets are
    compiled before and separately from all of the files processed by
    Jekyll.

[^unicorn]: Again, assuming you have added `unicorn` to your `Gemfile`
    and configured it appropriately.

[Heroku]: http://www.heroku.com/
[Jekyll]: http://jekyllrb.com/
[Rack::Jekyll]: https://github.com/adaoraul/rack-jekyll
[custom Heroku buildpack]: https://github.com/mattmanning/heroku-buildpack-ruby-jekyll
[Rack::TryStatic works just fine]: http://mwmanning.com/2011/12/04/Jekyll-on-Heroku-Part-2.html
[Heroku's Ruby buildpack]: https://github.com/heroku/heroku-buildpack-ruby/blob/master/lib/language_pack/ruby.rb#L573
[Rails asset pipeline]: http://guides.rubyonrails.org/asset_pipeline.html
[CoffeeScript]: http://coffeescript.org/
[Compass]: http://compass-style.org/
[Unicorn]: http://unicorn.bogomips.org/
[Rack::Contrib]: https://github.com/rack/rack-contrib
[entire source of this site]: https://github.com/jbhannah/jbhannah.net
[configured Unicorn]: https://github.com/jbhannah/jbhannah.net/blob/master/unicorn.rb
[Compass settings]: https://github.com/jbhannah/jbhannah.net/blob/master/compass.rb
[Gemfile]: https://github.com/jbhannah/jbhannah.net/blob/master/Gemfile
[converter plugin for Jekyll]: https://github.com/mojombo/jekyll/wiki/Plugins