---
title: 'How to: Free SSL Certificate that Renews Automatically'
toc: true
author: Richard Jeffords
author_img: /img/boo.png
author_link: https://richardjeffords.com
excerpt: "I could hop on the proverbial soapbox and preach about the reasons why having an SSL certificate for your website is important, but you're not here for that. You're here because you already know..."
tags:
  - SSL
  - Bash
  - Cron
  - Sys Admin
  - Encryption
date: 2016-10-01 17:03:38
---
## Introduction
I could hop on the proverbial soapbox and preach about the [reasons](https://www.linkedin.com/pulse/importance-advantages-ssl-certificates-jay-jones) why having an SSL certificate for your website is important, but you're not here for that. You're here because you already know. SSL is basically the first step to legitimacy these days. Some, I'll call them [opportunistic companies](https://www.godaddy.com/web-security/ssl-certificate), have been taking advantage of the situation and charge a lot of quan to help you out. Thankfully, there is better a way to get a certificate, and it's free to use! It's called [Let's Encrypt](https://letsencrypt.org/), and it's available for [almost](https://letsencrypt.org/docs/rate-limits/) any website. Thanks Linux Foundation! 
## Prerequisites
- SSH Access
- Bash or a similar shell
- Ability to create a cron job

That's it! Simple enough.
## Get the Certificate
Head over to the [Certbot](https://certbot.eff.org/) and follow the installation instructions for your server and operating system. Once you have gotten the certificate, restart your web server if you haven't already and see it if works. I recommend forcing SSL on your site. It's not too hard to do and definitely a good idea for security.
## Set up the Cron Job
### Test the Renew Script
Now that you have a working SSL certificate, you should try to renew it with the following command:
```
    letsencrypt renew --dry-run --agree-tos
```
You should get some kind of feedback about whether or not this works. Here's what mine looks like:
![Dry run success](renew_test.jpg)
I know some of you might be curious as to what happens if you don't include the dry run flag, so I did that too. If it didn't work, you may want to try getting a different certificate or heading over to the [Let's Encrypt community](https://community.letsencrypt.org/) for help.
### Add the Job
Open up your cron file on your server (the command is usually `crontab -e`) and add this line to the bottom:
```
0 8,20 * * * sleep $((3600 * (RANDOM % 4))); letsencrypt renew --agree-tos > /dev/null
```
What this does is tell your cron to check the status of your certificate twice a day at a random minute during the 8th and 20th hours of the day (starting at zero). Why do I do it this way? Because that's how they want you to use it, twice a day at a random minute. Feel free to change the 8 and 20 to other numbers if you feel the desire to do so. Finally, we send the output to /dev/null because we don't need it. If you have mail set up on your server, you'd get emailed with the output from above where we didn't use the `--dry-run` flag if you didn't send it to /dev/null.
Optionally you can add `-m youremail@example.com` after `--agree-tos` to send yourself an email when your certificate actually does get renewed.

All done! Welcome to the future!  