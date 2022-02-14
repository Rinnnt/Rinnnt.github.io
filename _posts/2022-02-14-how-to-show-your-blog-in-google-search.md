---
layout: post
title: "How to Show your Blog in Google Search"
date: 2022-02-14
categories: seo
excerpt_separator: <!--more-->
---

Google will very likely not know about your hosted website until you tell google about it. Lets dive into how google search works, how to show your website in google search results, and how to optimize google search for your website using jekyll-seo-tag.
<!--more-->

### Google Search

There are three steps before google outputs search results for any website: 
1. Crawling
2. Indexing
3. Serving

Crawling is the act of finding and recording what websites exist on the web. Google uses an automated web crawler to find out new pages, and store them in their database. This could be done through following a link from a known webpage to a new webpage, or by uploading a sitemap(a list of pages) manually. A new website probably does not have a link leading to it, so the author should manually upload information about the new website.

Indexing is when google tries to make sense of your website. It finds out what the website's purpose is, what its contents are about, etc. This helps google understand your website, so that it can use the information to *serve* your website to users.

Serving is when google compares the user's query with website information to provide the best matching websites for a search result. 

[Find out more about google search](https://developers.google.com/search/docs/beginner/get-started)

### Showing your website on google

To check if your website is showing on google search results, type the following into google search:
```
site:yourwebsite.com
```
This queries only for the contents of the specified website.

To show your website on google search, go to [Google Search Console](https://search.google.com/search-console/about), and add your website as your property. To do this you will be asked to verify the ownership of the website. The easiest way to do this for me was to add a google search console verification meta tag into the `<head></head>` part of your website's html content. 

With your website added as a property, click on URL inspection on the left content tab, and type in your website domain name. If google does not know about your website it will show "Not Indexed". On the top right corner there is a "Test live url" feature that can check if google can crawl to your website. Check that google can reach your website then click on "Request Indexing". This will take a few days to complete (for me it took 3 days). After your URL has been indexed, google will now be able to serve your website to users, because it has information about your page.

### SEO using jekyll-seo-tag

SEO (Search Engine Optimization) improves your website's availability on the web for search engines, by providing more or better metadata about the site. If you have used [Jekyll](https://jekyllrb.com/) to create your website, you can use the jekyll-seo-tag plugin to add metadata easily.

To use Jekyll-seo-tag, add the following line at the bottom of your site's `Gemfile` (not Gemfile.LOCK):
```ruby
gem 'jekyll-seo-tag'
```
Then add the following to your site's `_config.yml`:
```yaml
plugins:
 - jekyll-seo-tag
```
Finally, add the following right before </head> in your site's templates, by default it is in the directory `_includes` and in the file `head.html`. Delete duplicates if necessary:
```
\{% seo %\}
```

Jekyll-seo-tag is installed! Make sure your pages have YAML front matter so that jekyll-seo-tag can use the front matter information to create the metadata. For more usage information consult [Usage](https://github.com/jekyll/jekyll-seo-tag/blob/master/docs/usage.md).

How do we check if Jekyll-seo-tag is working or not? Simply go to your site using your browser, and right click any blank part of the website. At the bottom click on either `view page source` or `inspect` to load the html contents of the site (F12 for shortcut in chrome or firefox). Now expand the `<head>...</head>` section and if it is working properly, you should see:
```html
<!-- Begin Jekyll SEO tag v(version) -->
...
<!-- End Jekyll SEO tag-->
```


[Back](/)
