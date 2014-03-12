jekyll-travis
=============

Integrate Jekyll with Github Pages and Travis CI to automatically build Jekyll site.

## Background

[Github Pages](http://pages.github.com/) are a great approach to building websites. Using a Github repository that follows some naming conventions and the [Jekyll](http://jekyllrb.com/) static site generator we can build fairly sophisticated static websites that remain easy to maintain. Github Pages will automatically generate a website from a repository containing a Jekyll project and we can use the [Github Pages](https://github.com/github/pages-gem) Ruby Gem to maintain a local Jekyll environment in sync with GitHub Pages.

Letting Github Pages generate the Jekyll site has three important limitations:

* we can't use an [asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html), e.g. to compile sass into css or to concatenate and minify javascript
* we can't use [jekyll plugins](http://jekyllrb.com/docs/plugins/)
* we can't use the [Pandoc](http://johnmacfarlane.net/pandoc/) markdown converter

It is understandable that Github Pages doesn't allow the above for security reasons (it uses the Jekyll `--safe` flag). The workaround is to generate the site locally and then to push the generated HTML to Github. This works fine for personal blogs or small websites, but doesn't really scale to collaborative projects - and this is what Github is about. What we need for larger projects is a workflow that automatically builds a Jekyll site hosted on Github Pages whenever the Github repo holding the source code is updated.

The main motivation for me was to be able to use the Pandoc document converter, as I usually generate [Scholarly Markdown](http://blog.martinfenner.org/2013/06/17/what-is-scholarly-markdown/) with embedding of references and some other non-standard markdown. And I need to use plugins to enhance Jekyll, e.g. for better integration with [knitr](http://yihui.name/knitr/) and other tools that produce markdown.

## Setup

The basic idea is to use the [Travis CI](http://docs.travis-ci.com/user/getting-started/) continuous integration (CI) service. This service is free for open source projects (assuming [fair use](http://travis-ci.com/plans)) and nicely integrates with Github via Service Hooks. For private projects or projects that are not open source please consider their commercial service for private repositories.

The workflow is as follows:

* `git pull` to the Github repo triggers Travis CI
* Travis CI starts up a virtual machine and installs all required software (mostly Ruby gems)
* We use a custom rake task to tell travis CI how to build the Jekyll site and push the updated content back to Github
* Travis CI clones a different branch (either `gh-pages` or `master`, depending on the kind of Github repo) that holds the static HTML pages
* Travis CI runs `jekyll build` with the destination in the other branch
* Travis CI does a `git push` of the other branch
* Github Pages starts serving the updates site

Depending on the required software that needs to be installed, the whole process takes anywhere between 1 and 5 min and is fully automated.

You can add the example files provided in this repo to your Jekyll project to get started. Please remember the following:

* make sure you have enabled your source repo in the Travis CI admin dashboard so that the webhook is triggered
* install the travis gem (`gem install travis`) and generate a secret version of three required `ENV` variables `GIT_NAME`, `GIT_EMAIL` and `GH_TOKEN` (more info in the sample `.travis.yml`).
* make sure you add `vendor` to your .gitignore as Travis CI is vendoring the Ruby gems there. The `vendor` folder should also be excluded in the Jekyll `_config.yml` (see example file).
* add the following to your Jekyll `_config.yml` file: `username`, `repo` and `branch`.
* make sure `destination` in `_config.yml` matches the path to the destination repo defined above.
* we have seen [intermittent timeouts](http://blog.travis-ci.com/2013-05-20-network-timeouts-build-retries/) fetching gems from Rubygems.org. `install: bundle install` lets Travis CI automatically retry, and we are using `source "http://production.cf.rubygems.org/"` in Gemfile to point to a different repository.
* add the contents of `Rakefile` to your Jekyll Rakefile (or replace it). The provided `Rakefile` has some additional commands, but the important one here is `rake site:deploy`.
* (Optionally) add a Travis CI logo/link to your README.

## Examples

The following sites use the workflow described above. Please send me a note if you want me to add your site.

* [ALM Community Website](http://articlemetrics.github.io/) - the project website for an Open Source software project. Github repo [here](https://github.com/articlemetrics/articlemetrics.github.io), the site currently uses Jekyll in `--safe` mode (Travis CI needs under 2 min to build the site).
* [Opening Science](http://book.openingscience.org/) - a book on Open Science, Github repo [here](https://github.com/openingscience/openingscience.github.io).

## License

See [LICENSE](LICENSE).
