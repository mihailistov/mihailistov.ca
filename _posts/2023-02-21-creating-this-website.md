---
layout: post
title: Creating this website
date: 2023-02-21T17:19:21.000-07:00
tags:
- general

---
## > Delusions of Grandeur

I think every software engineer starts out their journey of building a personal site to showcase their thoughts, works, and present themselves with grand ideas. I imagined custom engineering a blog setup with a full backend dashboards, built in CI/CD pipeline with automated testing to run and ensure everything is working as expected, ability to add integrations to connect to my personal Instagram feed, comment sections on blog posts that I could create using a WYSIWYG editor. However upon considering the reality of that, the infrastructure required to connect the frontend, backend, and databases and on-going hosting cost of that, it made absolutely no sense.

Some key factors that really helped shape my decision were:

* Am I over-engineering this?
* Do I really need all the fanciful functionality?
* Is the overhead and additional costs worth the effort?
* Is the time investment worth the business value that this is delivering?

At the end of the day, I just wanted a simple, fast, responsive website where I could share some of my thoughts and help people get to know me. Anything beyond that is superfluous and simply unnecessary.

## > Solutions worth considering

Some of the options I looked at are listed below:

1. Ghost Headless CMS
2. React w/ SSR (Server-side rendering)
3. Static site generators (Jekyll vs. Gatsby vs. Hugo)

### Ghost Headless CMS

#### Pros

* Has pretty much every feature any blogger could possibly dream of and support to add custom integrations
* Effortless, time-saving, easily extensible

#### Cons

* Process intensive, server runs on node, requires an instance w/ at least 1GB memory
* Requires a MySQL database (more overhead)

After some research it seems this solution would require at least $5/month if I used GCP to host for both the instance and the database. While the ease of use, functionality, and features were tempting, but it's overkill.

### React w/ SSR

#### Pros

* Built using vanilla ES6 Javascript which I'm extremely comfortable using, as well as my extensive experience working with React makes this a nice option

#### Cons

* Requires lots of custom configuration, npm package setup
* Still requires an instance to run Node.js ($$)

### Static site generators

Brief overview of 3 different options, not delving into too much detail, and mostly opinion based:

### Hugo

No prior experience with Hugo, however given I have some Go experience, this is attractive as an option.

#### Pros

* Fast build times
* Powerful templating
* Support for i18n

#### Cons

* Confusing templating
* Documentation can be confusing

### Gatsby

I have used Gatsby before and I believe it's best for more fanciful sites where presentation and a feature-rich experience might be needed.

#### Pros

* Best performance
* Quality plugin ecosystem & good documentation

#### Cons

* Over-engineered for very simple sites

### Jekyll

I have limited experience with Ruby and Liquid, but this seems like a good opportunity for a refresher 🍪

#### Pros

* Simple templating, using markdown and Liquid (used by Shopify templates)
* **Free** hosting via Github pages

#### Cons

* Installing ruby dependencies and initial setup can be cumbersome
* Complicated page logic may be difficult to implement

## > Implementation

I ended up going with Jekyll due to it's simplicity, the fact that it's free to deploy via Github pages, and because it fulfills all my needs for now.

### Installing dependencies

* Ruby
* Jekyll

First I went and made sure I had all the prerequisites I needed to install Jekyll. Being on macOS I followed these detailed [instructions](https://jekyllrb.com/docs/installation/macos/)

All was going so well until I had reached the last step which was to use Ruby to install the gem required for Jekyll: `gem install jekyll`

This resulted in a permissions error: `(Gem::FilePermissionError)`

My instinct was to run the command with sudo: `sudo gem install jekyll`, however upon a bit of research I learned that you should _never_ run gem commands with sudo. Doing so poses security risks as you may inadvertently install a malicious gem, it's something that should never be done in a production environment.

Upon digging a bit deeper, it looked like it was trying to install Gems under `/Library`  which would have required root access. So next I tried to pass this flag that would specify to install Gems under my user directory:

    gem install --user-install bundler jekyll

Finally it complained that I didn't have the PATH exported: `You don't have [PATH ]in your PATH, gem executables will not run.` after running `gem install --user-install bundler`

Since I use zsh, I added the following to my `~/.zshrc` file:

    export GEM_HOME="$(ruby -e 'puts Gem.user_dir')"
    export PATH="$PATH:$GEM_HOME/bin"

That finally allowed me to install the required Gems for Jekyll. Bit of a headache, it's no wonder that people are paying for a solution such as [Ruby on Mac](https://www.rubyonmac.dev/ "Ruby on Mac"), but I wanted to save $49 USD.

### Deciding on a Theme

Deciding which theme to use seems like a small decision but will ultimately have to serve the purpose of what I'm using this site for, which for now is to display some simple (text) content. While I love photography and have a dream of turning this into a full-fledged blogs with rich (photo) content, that is a bit beyond my scope for what I'm trying to accomplish.

For future reference, I really loved this [theme](https://index.jekyllthemes.io/project/triangles) but since this better suits photo and travel blogging, and costs $49 USD, I've decided against it.

I instead chose this simple, free [theme](https://github.com/knhash/jekyllBear)

However it seems my issues with Ruby gem files and Jekyll are tireless, because the theme `.gemspec` file depends on `"jekyll", "~> 4.2"`, and because github pages doesn't support jekyll > 3.9.3, it doesn't play nicely:

    -> bundle install
    Fetching gem metadata from https://rubygems.org/...........
    Resolving dependencies...
    Could not find compatible versions
    
    Because every version of jekyll-bear-theme depends on jekyll ~> 4.2
      and Gemfile depends on jekyll ~> 3.9.3,
      jekyll-bear-theme cannot be used.
    So, because Gemfile depends on jekyll-bear-theme >= 0,
      version solving has failed.

Since I was already set on this theme, I wanted to see if the requirement of `"jekyll", "~> 4.2"` in the `.gemspec` for the theme was a hard requirement, my hunch was that it is not, since this is a very basic theme, I suspect it should work with any version of jekyll. So now I had to figure out if it was possible to modify an installed gem, turns out there is, first we need to "unpack" the existing gem file to expose it's contents, this is done using:

`gem unpack jekyll-bear-theme`

This created an output folder, so I dove head-first into it: `cd jekyll-bear-theme-0.1.3`

I noticed the unpacked gem was missing the required `.gemspec` file that is needed to build a gem, so I resorted back to the git repo for the theme, and alas there was a copy of it, so I downloaded that file, and modified line #15 in `jekyllBear/jekyllBear.gemspec`

`spec.add_runtime_dependency "jekyll", "~> 4.2"`

Changing the "4.2" to the latest supported jekyll version by `github-pages`, which is "3.9.3".

Finally, from inside the unpacked gem, I ran `gem build`, resulting in an output of `jekyll-bear-theme-0.1.3.gem`. The last step was to install this gem, replacing the existing install,

`gem install --local jekyll-bear-theme-0.1.3/jekyll-bear-theme-0.1.3.gem`

Running `bundle install` shows a success, we now have jekyll version 3.9.3 installed alongside the `jekyll-bear-theme` gem, finally, we install the `github-pages` gem and test locally that the theme places nicely, which it does. Awesome sauce!

### Github Pages

Next I went ahead and read through and followed steps on deploying a Jekyll site on Github Pages [here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)

However, I quickly ran into issues yet again, surprise surprise,

    github-pages 228 | Error:  The jekyll-bear-theme theme could not be found.

It turns out that Github Pages only has a limited number of supported themes found [here](https://pages.github.com/themes/)

However, there is a supported Gem that allows you to link to any theme that is hosted on a github repo, alas, there is still hope for this simple theme I've stubbornly committed to using! Enter  [jekyll-remote-theme](https://github.com/benbalter/jekyll-remote-theme)

So now I had to fork the original theme into my own repo, and modify the `.gemspec` file to drop the required version, as done above, the process is the same.

Next I updated the `_config.yml` file with `remote-theme: mihailistov/jekyllBear`

Success! Github Pages successfully builds and deploys my theme.

The last step was to setup a custom domain which was pretty straight-forward, just required updating the DNS records, adding a CNAME www record to point to the [github.io](http://github.io) generated URL, and adding several A records pointing to github's servers.

Mint, we're in business!

### Forestry CMS

While completely unnecessary, I learned that there's a sweet CMS that plays nicely with Github Pages and Jekyll, within a minute I was able to connect Forestry to my repo and get it to display everything in a nice WYSIWYG editor. While I prefer to write my notes using a local markdown app like Obsidian, having Forestry setup will make things it a bit nicer to tweak things or just make quick updates.