+++
title = 'Privacy'
# Rendered at /privacy/, but kept out of the home page post list, RSS and taxonomies.
[build]
  list = 'never'
+++

_Last updated: September 30, 2026_

The short version: this site records anonymous visits so I can see how people read what I write. Nothing is stored on your device, nothing identifies you, and nothing is shared with anyone.

## What I record

When you read a page here, a small script on zoravur.com records:

- **How you move through the page**: scrolling, mouse movement and clicks, and the page as it looked to you. I can replay this later, a bit like a screen recording of the page. Anything you type into a form is masked in your browser and never sent.
- **Which page you're on**: its address without the query string (`zoravur.com/about/`), and its title.
- **Where you came from**: just the site's domain, like `news.ycombinator.com`, not the full link.
- **Basic device details**: browser (e.g. Firefox), operating system, mobile or desktop, window size, and time zone.
- **Your country**: looked up from your IP address when the request arrives. Only the country is kept.

I use this to see which posts people read, how far they get, and where they lose interest, so I can write better ones.

## What I don't record

- No cookies, `localStorage`, or anything else stored on your device.
- No IP address, no full browser "user agent" string, no city, no internet provider.
- No ads, no third-party trackers, no cross-site tracking. The script and the data stay on zoravur.com and are never sold or shared.

## How page views are grouped into a visit

To know that three page views belong to one visit, the server takes your IP address and browser name, mixes in a random value that changes every day, and keeps only a one-way hash of the result. The random value is deleted at the end of the day, and the hash is erased once you've been inactive for 30 minutes. After that, nobody (me included) can connect your visit to you or to any other visit.

## Where it's stored and for how long

The site is served by GitHub Pages through Cloudflare. Like any web host, they handle your IP address to deliver pages. The recordings are stored in Cloudflare's database (D1) on my behalf and are deleted automatically after **30 days**.

## Opting out

- If your browser sends a [Global Privacy Control](https://globalprivacycontrol.org/) signal, nothing is recorded at all. Brave, DuckDuckGo and Firefox can send it (it's a setting in Firefox).
- Blocking JavaScript on this site also stops recording. Everything here still works without it.

## Legal basis and your rights

I record visits based on my legitimate interest in understanding how this site is read (GDPR Art. 6(1)(f)), limited to the anonymous data above. Because the data isn't tied to you, I usually can't find "your" recording to show or delete it. If you're worried about a specific visit, tell me roughly when it happened and I'll do what I can.

Questions? Email me at [hello@zoravur.com](mailto:hello@zoravur.com).
