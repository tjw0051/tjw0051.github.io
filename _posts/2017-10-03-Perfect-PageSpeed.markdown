---
layout:     post
title:      "Getting a Perfect 100/100 Google PageSpeed Score with an S3 Static Website"
subtitle:   "How to Spaghettify your Website beyond Recognition"
date:       2017-10-03 15:16:00
author:     "Tom"
header-img: "img/2017-10-03_cover.png"
header-mask: 0.5
catalog: true
tags:
    - Web
    - SEO
    - Optimisation
---

![Perfect PageSpeed](https://imgur.com/J9CY0dl.png)

[Google PageSpeed](https://developers.google.com/speed/pagespeed/insights/) is a tool for testing the performance of websites for mobile and desktop, and plays some part in how Google's algorithms rank your website. Page speed isn't everything but I got obsessed with the idea of achieving a perfect 100/100 score for both mobile and desktop, so I set out to achieve it on a small landing page for one of my mobile apps.

I'll be tailoring this guide to websites hosted on [Amazon Web Services S3](https://aws.amazon.com/s3) and [Cloudfront](https://aws.amazon.com/cloudfront). If you're unfamiliar with this hosting method I highly recommend checking out [Jorge Escobar's guide here](https://medium.com/@esfoobar/setting-up-an-https-static-site-using-amazon-aws-7ab270c4e277) which covers hosting on S3, distributing through the Cloudfront CDN and setting up SSL/TLS/HTTPS. Hosting a static site on S3 can be very cost-effective; hosting multiple sites for < $5 month is not unusual. With Amazon's free-tier it may even be free for you.

After you've set up your website through S3, go to [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/), input your website and hit analyze.

![Youtube's PageSpeed Score](https://imgur.com/AgWfqx5.png)

*Yes, that's youtube with a score of 46 for mobile and 76 for desktop. Funnily enough, Vimeo gets 93 mobile / 80 desktop. I hear another [anti-trust fine](https://www.theverge.com/2017/6/27/15872354/google-eu-fine-antitrust-shopping) a'brewin*

Rather helpfully PageSpeed provides insights into how to improve your website's score. I'll break down each of the Possible Optimisations below into seperate sections with advice on how to fix them for an S3-hosted website.

## Optimise Images

![Optimised Images](https://imgur.com/clbcCHU.png)

This is one of the easiest things to fix and should give one of the biggest boosts in both performance and PageSpeed score. PageSpeed automatically compresses and resizes your images for you and provides a download link at the bottom. Click the link, download the images and replace your websites images with Google's optimised versions. 

## Minify Javascript/HTML/CSS

Minification removes unneccessary whitespace and characters to make the overall filesize smaller. This is also pretty easy to fix, you can use the optimised versions provided by google, the same as with the images, or you can use a tool like [Gulp](https://gulpjs.com) or even an in-browser tool like [minifier.org](http://www.minifier.org). Replace your files with the minified versions and the warning should disappear.

![Minification](https://imgur.com/N1s04CN.png)

## Avoid Landing Page Redirects

If you're receiving this issue on an S3 site make sure you have your DNS set up correctly (Route53 if you used the guide mentioned earlier). Your domain name should point to your Cloudfront Distribution, which should point to your S3 bucket. Here's a screenshot of my configuration: 

**Route53 Setup**

![Route53 Setup](https://imgur.com/hMljEl4.png)

**Cloudfront Setup**

![Cloudfront Setup](https://imgur.com/BY8YgFx.png)

**S3 Setup**

![S3 Setup](https://imgur.com/s1wooI0.png)

## Leverage Browser Caching

Another slightly more difficult configuration is setting up caching. This enables client-side caching so that resources are stored by a user's browser for a duration.

There are two ways you can set this up: on the S3 Console or via the S3 upload command.

To change this on the console, select all items in your website's S3 bucket, click 'More' > 'Change metadata'. 

![Select all bucket items](https://imgur.com/9JmqVhV.png)

Add a new item with key 'Cache-Control'. Set the value to 'max-age:[seconds]'. This is the how long the item will be cached client-side before expiring, and a new copy is downloaded by the client. This needs to be a big number, if you set something like 2 hours then Google PageSpeed will still nag you. In the example I've set it to 2 years (63072000 seconds) which supresses these warnings. 

![Add Cache-Control](https://imgur.com/yZ7lQGo.png)

The second way to do this via the S3 upload command with the `--cache-control` option. Here's an example:

```sh
aws s3 cp --recursive index.html s3://ansavoicemail.com/ \
    --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers \ 
    --cache-control max-age=63072000
```

---

Remember to invalidate the Cloudfront cache after you make changes to your S3 bucket, so that these changes are propogated to the edge-servers, otherwise Cloudfront will be serving the old version of your files. There are better ways to do this but if you want to quickly refresh all files, go to Cloudfront, select your distribution, go to the `Invalidations` tab, click `Create Invalidation`. In the `Object Paths` box put `*`. Click `Invalidate`. 

![Invalidating Cloudfront Cache](https://imgur.com/oxME6iM.png)

## Eliminate render-blocking JavaScript and CSS in above-the-fold Content

This is perhaps the most difficult optimisation as its highly dependent on your websites layout and content. This warning occurs because your website is loading external resources above-the-fold (the top-section of your website that is visible when it loads). For example you may be loading bootstrap or ajax from their respective CDNs, which is slowing down your rendering.

Firstly try to remove any unused external resources, or move them locally into your s3 bucket if possible. If you're using bootstrap or another framework, you're probably only using a fraction of the CSS it provides. You can try to extract only the CSS selectors that are being utilised. Check out [CSS Used for Google Chrome](https://chrome.google.com/webstore/detail/css-used/cdopjfddjlonogibjahpnmjpoangjfff?hl=en) Run it on your site, copy the CSS it gives you, run it through a CSS minifier and save it. If you only have a small amount of CSS you can include it in-line on your site. In the `<head>` section of your site add the minified css between `<style>` tags:

```css
<style>
.nav{margin-top:2rem;margin-bottom:2rem}
...
</style>
```

A common pitfall of this step is Google Analytics which throws the 'Eliminate render-blocking...' warning. To remedy this, download Google Analytics from https://www.google-analytics.com/analytics.js and store it with your site in S3. I also found that this script was re-downloading analytics.js from Google, so CTRL+F in analytics.js and replace any references to the above URL with your local file:

```sh
https://www.google-analytics.com/analytics.js
replace with e.g. ->
js/analytics.js
```

You may also have a Google Tag Manager script, something like this:

```html
<script async src='https://www.googletagmanager.com/gtag/js?id=UA-01234567-1'>
```

Go to the URL, download the script, save it locally. Run CTRL+F again and replace any references to `https://www.google-analytics.com/analytics.js` with your local copy. 

## Enable Compression

As S3 isn't a normal webserver it wastes no processing power in gzip compressing files for you, so you need to do this manually before you upload them. 

You should only compress html, css and javascript files with gzip. To do this on Mac and Linux run gzip -9 and then the filename:

```
gzip -9 -k -f *.html
gzip -9 -k -f *.js
gzip -9 -k -f *.css
```
`-9` is for maximum level of compression, `-k` keeps the original file, `-f` is force, so we replace our previous compression without the 'are you sure...' prompt.

If you compress `index.html` you should get `index.html.gz` out. We need to upload these compressed files to S3 with the compression type included in the metadata, so that the client knows to decompress the files before showing them in the browser.

You can add the compression metadata through the S3 console or with the upload command. The command below shows how you might do this with the `--content-encoding` option:

```sh
aws s3 cp --recursive index.html s3://ansavoicemail.com/ --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers --cache-control max-age=63072000 --content-encoding gzip
```

Alternatively, in the S3 console, select your compressed files, go to 'More' > 'Change metadata'. Add a new key called 'Content-Encoding' with the value 'gzip':

![Content Encoding Metadata](https://imgur.com/vXUE5Dd.png)

**NOTE:** Don't forget to change any references in your html files to the compressed files. For exaple if you're loading in your JavaScript with `src="css/myscript.min.js"`, compress your JavaScript then change the reference to `src="js/myscript.min.css.gz"`. Alternatively rename your compressed files to remove the `.gz` extension, then you'll have less problems during development.

## Reduce Server Response Time

I received this warning while I was making the previous optimisations. Once i'd removed all the other warnings this one dissapeared too. As its hosted on S3 with Amazon's CDN you're pretty cutting-edge in terms of deployment. If that doesn't cut it then you need a consultant, not a tutorial. 

You might want to try reduce the size of your content. 100MB of optimised images is still 100MB of images; try to defer rendering until the above-the-fold content is rendered or implement some sort of splash-screen to make the site appear to render quicker. 

## Conclusion

Hopefully these optimisations have improved the speed of your S3 static site to that perfect 100, or as close as you can get. PageSpeed is only a part of SEO and is not worth obsessing over. Getting a perfect 100 involves a lot spaghettification that can cause headaches if you need to make frequent updates. Downloading scripts like Google Analytics to your own server means that you won't be up-to-date if it changes, unless you set up some additional automation. Unless your website is cripplingly slow, users probably won't even notice your fine-tuning, but it should help bring in a few new users through search engines.

—— Tom 2017.10