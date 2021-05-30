# bettysteger.com

Clean Jekyll theme

> This theme requires ruby and rubygems installed

* [x] Clean layout
* [x] Resposive layout
* [x] Preprocessor SASS
* [x] CSS minified
* [x] Pagination
* [x] Syntax highlight
* [x] Author config
* [x] Comments with Disqus
* [ ] Search posts
* [ ] Share posts

---

### Development

1. Install Ruby gems: `bundle install`
2. Start Jekyll server: `bundle exec jekyll serve`

Access, [localhost:4000](http://localhost:4000/)

### Deploy in Github pages

If you have no custom plugins what so ever (see [below](#plugins-on-github-pages)), you do not have to compile your site first, just:

* `git push origin master` or `git push origin gh-pages`

Otherwise:

1. Change the variables `GITHUB_REPONAME` and `GITHUB_REPO_BRANCH` in `Rakefile`
2. If master is your `GITHUB_REPO_BRANCH` create a `dev` branch
3. Run `rake` or `rake publish` for build and publish on Github (this will force overwrite the `GITHUB_REPO_BRANCH` branch)

---

### Using Rake tasks

* Create a new page: `rake page name="contact.md"`
* Create a new post: `rake post title="TITLE OF THE POST"`

---

### Plugins on GitHub Pages

GitHub Pages is powered by Jekyll. However, all Pages sites are generated using the --safe option to disable custom plugins for security reasons. Unfortunately, this means your plugins won’t work if you’re deploying to GitHub Pages.

You can still use GitHub Pages to publish your site, but you’ll need to convert the site locally and push the generated static files to your GitHub repository instead of the Jekyll source files.

