---
title: Setting up a blog on GitHub
description: Blogging with GitHub, or, how to misuse it for note-taking like this
created: 2019-07-30
updated: 2020-04-11
---
I wanted something simple for kind blogging or taking notes. Although, when reading on, you may not think it is all that simple. Why not just use Wordpress or something that can be set up in 5 minutes? Oh well, I like to complicate matters. I was looking for something that is file-based and can use [Markdown](https://daringfireball.net/projects/markdown/). I didn't want something running on a database, I just see it as overkill for mostly text content.

## Introduction

I really dislike *heavy* websites that load megabytes of stuff for a few lines of text. I sighed with relief that it's not only me when I read Maciej Cegłowski's *[Website Obesity Crisis](https://idlewords.com/talks/website_obesity.htm)*. I would be just fine serving flat files, which is basically what this does.

[GitHub and Jekyll](https://help.github.com/en/articles/using-jekyll-as-a-static-site-generator-with-github-pages) work as a static site generator. The guide there is enough to get a basic system set up, but to give it more oomph, I had to piece together information from [Jekyll](https://jekyllrb.com) and [Liquid](https://help.shopify.com/en/themes/liquid), and wanted to keep some notes in one place.

Jekyll can <a href="https://jekyllrb.com/docs/usage/">generate a basic site or site scaffolding </a> to use as a basis. There are also <a href="https://jekyllrb.com/docs/themes/">several sites that offer Jekyll themes</a>. But I like the slight pain of doing everything from scratch, so there you go.

I am pretty happy how it turned out, especially size-wise. This article is under 150 kB and the site loads no off-site resources, except whatever GitHub might load on its own. It could be less, it should be less, but I gave in to the vanity of webfonts, which make up over 100 kB.

There is a [repo](https://github.com/Primozz/blogtest) where I played around with the initial versions.

Because GitHub generates the static files, I wouldn't need to have Jekyll and its dependencies installed. During initial testing, this can be rather inconvenient, as I would need to push the commits to see the effect. Installing Jekyll is no problem, the only issue I had to resolve  are setting the paths so that the site works both locally for testing and when pushed to GitHub. A workable solution is below under <a href="#testing">Testing</a>.

## Directory Structure

- `_includes` &ndash; Content from the files in this folder can be included with the `{% raw %}{% include <filename> %}{% endraw %}` tag.
- `_layouts` &ndash; The files in this folder serve as the templates. You specify the layout in each file's front matter or a default layout in `_config.yml`.
- `_posts` &ndash; The files in this folder contain the main content of the individual pages. The file names must be formatted like this: `YEAR-MONTH-DAY-title.MARKUP`
- `assets` &ndash; Site assets such as stylesheets and images. This folder name is not mandatory like the others that start with an underscore.

Jekyll's Docs have more on the <a href="https://jekyllrb.com/docs/structure/">directory structure</a>.

## Layout

I started with a site boilerplate, specifically <a href="http://getskeleton.com/">Skeleton</a>, which provides a responsive 12-column fluid grid, but basically any template can be used.

I took apart the template, replacing relevant parts with <a href="https://jekyllrb.com/docs/variables/">variables</a> and <a href="https://jekyllrb.com/docs/includes/">includes</a>.

<a href="https://github.com/Primozz/blogtest/blob/master/_layouts/default.html">The final result for the default layout</a>.

## Styles

On top of Skeleton's stylesheets, I added my own with customizations and webfonts. Nothing special here.

I also wanted to have code snippet highlighting, which Jekyll supports thanks to <a href="http://rouge.jneen.net/">Rouge</a>. I selected a few to try from the <a  href="https://jwarby.github.io/jekyll-pygments-themes/languages/ruby.html">gallery of highlighting themes</a> (<a href="https://github.com/jwarby/jekyll-pygments-themes">repository</a>). 

Syntax highlighting is invoked with `{% raw %}{% highlight <language> %}{% endraw %}` and closed with `{% raw %}{% endhighlight %}{% endraw %}`. Here is a <a href="https://github.com/jayferd/rouge/wiki/List-of-supported-languages-and-lexers">list of available languages</a>.

Example using the Manni theme:
<pre>
{% raw %}
{% highlight cpp %}
int main(int argc, char **argv){
    const char *first_arg{argv[1]};
    // do stuff
    return 0;
}
{% endhighlight %}
{% endraw %}
</pre>
results in

{% highlight cpp %}
int main(int argc, char **argv){
    const char *first_arg{argv[1]};
    // do stuff
    return 0;
}
{% endhighlight %}

See <a href="#escaping-liquid-tags">Escaping Liquid Tags</a> for how to prevent the processing of Liquid tags and display them instead.

## Posts (or in my case, notes)

Individual notes/posts go into the `_posts` folder. The file names must be in the format `YEAR-MONTH-DAY-title.MARKUP`, even if I don't use the date. However, the date is used for naming the subfolders where the pages with posts are generated.

The content must start with a YAML <a href="https://jekyllrb.com/docs/front-matter/">front matter</a>, where the mandatory variables are set, in my case the page title and meta description

The content is written in Markdown.

### Escaping Liquid Tags

To <a href="https://shopify.github.io/liquid/tags/raw/">display raw Liquid tags</a> on the page instead of processing them, put the part you want to escape between `{% raw %}{% raw %}{% endraw %}` and `{% raw %}{%{% endraw %} endraw %}`.

`{% raw %}{%{% endraw %} endraw %}` requires extra help. Putting it as such between `{% raw %}{% raw %}{% endraw %}` and `{% raw %}{%{% endraw %} endraw %}` doesn't work, because that has two `{% raw %}{%{% endraw %} endraw %}` in a row, where the first one closes the opening `{% raw %}{% raw %}{% endraw %}`, then the second one is considered an error and site building fails.

To work around this, I put just the opening left brace and percentage sign `{% raw %}{%{% endraw %}` in the escaped block and the rest `endraw %}` outside. The end result was this: `{% raw %}{%{% endraw %} raw %}{% raw %}{%{%{% endraw %} endraw %} endraw %}`.

To actually display this, I had to go one level deeper and escape everything again. I found that a decent way to escape such cases is to escape only the openers `{% raw %}{%{% endraw %}` and leave the rest unescaped.

## Testing

For content with a lot of markup, such as this one, it's worthwhile to install Jekyll and test locally before pushing to GitHub. I saved a lot of unnecessary commits by testing locally.

I have prepended all asset URLs with `{% raw %}{{ site.baseurl }}{% endraw %}`, so that the individual pages load everything they need. The site is in a subfolder of the domain, that is `my-notes`, same as the repo name. Because of that I don't prepend `{% raw %}{{ site.baseurl }}{% endraw %}` to any static links, such as `/my-notes/archive.html`, as `site.baseurl` inserts just the domain. This way, the published site works on GitHub, but locally, any static links don't work. But it is enough for testing whether the content renders correctly.

When building the site locally, I pass the parameter `--baseurl` with the full path to the folder into which the site is built, for instance:

{% highlight shell %}
jekyll build --baseurl file:///home/user/Documents/blog/_site
{% endhighlight %}

## Publishing

Once I'm ready to publish, I commit the changes and push them to GitHub. With just a few pages to generate, the updates usually appear after just a few minutes. A hard refresh is often necessary.

## Further Ideas

Jekyll can be further extended with plugins and code snippets. For the future, I have my eye on a <a href="https://github.com/allejo/jekyll-toc">code snippet for generating a table of contents</a> without a plugin or JavaScript.
