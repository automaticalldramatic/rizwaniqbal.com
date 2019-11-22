---
title: "10 tips to make great SEO friendly websites"
date: 2011-03-27T09:07:51+05:30
draft: true
toc: false
images:
tags: 
  - seo
  - google seo ranking
  - archives
---

> I have restored these posts from wayback machine. I was young and stupid then, so I would take my words with a hint of caution too

A great website design is a must for any business but it is useless to have a great website if nobody ever visits it. Its like building a theme park and not advertising it. Websites with zero visitors are as good as theme parks with empty rides. One of the biggest questions you will face as a web designer is to get your clients website “on the first page of Google”. Its perplexing how every client you meet will say the same thing and will have the same misconceptions about SEO. All they ever want is their websites to be on the top of google and they wouldn’t even want to change their website which looks like it was built in the 90′s by a guy learning HTML.
I wrote about an infographic a few days back about how Google ranks your web pages, but SEO is also an important part. Here are 10 SEO tips, to make a great SEO friendly site without compromising on design or usability.

1. Content was always the king and still is:

When I think about web spiders, I envisage machines like the Sentinels from Matrix. These go around the internet looking for content to feed on. They eat text from your pages and jump to another page when they are done. If you are building a website that uses flash or a lot of images and hardly any text, then you have a tough shot to get on top of Google listings (unless you have loads of money to spend on keywords).If you are using images, or sprites (for javascript animation), make sure you use  CSS background image replacement technique.

If you are using Flash, then make sure you are aware of how to make Flash objects accessible and web-crawler-friendly. Search engines have a really tough time crawling a website that uses Flash.

Make sure the website is well structured and this structure is exposed to bots. If you wanted these sentinels, that patrol the sewers to pass through your tunnel everyday, you would make sure that your tunnels are well connected. Your website is the part of the sewer (internet) that these sentinels (bots) patrol. If your website structure is not well defined, they will not crawl the entire website, leading to bad SEO.To make sure that these bots see your website in a way that humans do, here are a few things you should pay attention to.

2. Meta data should be unique:

Every page on your website should have unique meta data. Meta data is made of meta tags viz. keywords, description, page titles, author etc:  Many web designers build a website and forget to change meta data on every page. You can use a good CMS or a framework like CodeIgniter to generate unique meta tags for each page. This shows how to generate dynamic data using CodeIgniter.

This is important because, crawlers keep meta information about page in their memory. If each page has meta information that defines data on that page, you have a greater chance of featuring higher in search engine ranking to related topics.

3. SEO friendly URL’s:

Your URL is like the name plate on your house. If your URL tells the crawler what to expect on the page, it already knows what the structure is. Compare a URL like rizwaniqbal.com?c=90 to rizwaniqbal.com/companies/easports. The latter tells the crawler that this is a companies page for easports. Now, if someone searches for easports, there is a  good chance they will find your page.

Always try to use URL’s that have keywords for the page rather than parameters. I achieve this by writing controllers on CodeIgniter with SEO friendly names. You can even do this using .htaccess.

4. Use heading tags properly:


Make good use of heading tags in your web page content; they provide search engines with information on the structure of the HTML document, and they often place higher value on these tags relative to other text on the web page (except perhaps hyperlinks).

Use the `<h1>` tag for the main topic of the page. Make good use of `<h2>` through `<h6>` tags to indicate content hierarchy and to delineate blocks of similar content.

I don’t recommend using multiple `<h1>` tags on a single page so that your key topic is not diluted.

5. Make sure your site navigation is search engine friendly:

I have come across a lot of websites that have a flash menu. As discussed before, flash is not very search engine friendly. If you take my personal opinion, I would never use it. In fact, if I see a flash menu on a website, I don’t see it worthy of my attention and browse away.

Use Javascript. Use CSS. This is 2011. Don’t make flash menus please.

That is all about the structure of your website. Apart from that, here are a few more tricks that you can keep up your sleeve to improve a websites SEO.

6. Put Javascript/CSS outside the HTML:

This might sound rude to you, but do you like to pray before your meals? I HATE IT!! I like to get to the food as soon as I can. When you put javascript and CSS on your web page, you are making those hungry bots wait to get to the content. A lot of developers, out of sheer practice and lack of common sense put lines and lines of javascript in the header. As I said, your bots feed on content. Don’t make them wait. Put javascript tags at the bottom of your page and externalize as much as you can.

7. Use robots.txt


It is surprising how less robots.txt is used and talked about. Every website has its dark nook and corners that owners wouldn’t want to be exposed. I usually do a lot of testing on my website in a different directory. I don’t want these pages to be parsed. You should shield the pages that you don’t want using robots.txt. Its a nifty file that is very useful. Your admin pages, private directories and pages with a lot of script on it should not be scrolled by bots.

Unlike sentinels, web crawlers are very obedient.  You can tell them what is not food and they will not munch on the pages you disallow in robots.txt.

8. Use ALT attribute for images:

Most IDE’s that you use will show you a warning if you forget the alt attribute for images. This one is not required but is every bit useful if you want good seo.

Search engines will read alt attributes and may take them into consideration when determining the relevancy of the page to the keywords a searcher queries. It is probably also used in ranking image-based search engines like Google Images.

Outside of the SEO angle, image alt attributes help users who cannot see images.

9. Update regularly:

This ones a bummer. You need to feed crawlers new data every time they visit your website. Even if its pizza, you can’t eat the same thing everyday. Search engines love to see content of web pages changing from time to time as it indicates that the site is still alive and well.

With changing content comes greater crawling frequency by search engines as well. I have seen my blog suffer just because I don’t update a lot. But, to help that, I have widget on my home page that syncs with my other activities on the internet and posts a live feed on my blog everyday. This helps me keep my website alive for bots.

10. Follow W3C standards:

This one goes without speaking. You need to follow internet standards. Everyone loves well formed, clean code. Even search engines.