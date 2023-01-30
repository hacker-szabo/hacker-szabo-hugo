---
title: "Is Tabnabbing Still Relevant?"
date: 2023-01-30T19:19:59+01:00
---

## But wait, what is Tabnabbing?

Tabnabbing is a method to redirect a website to any site. It used to be a quite frequent vulnerability if a user could place a link onto the site (for example on Facebook or on a forum). The link either had to have `target="_blank"` or the page itself that contained the link without `target="_blank"` needed to be opened by a link that had it. When the user clicked, the new site - which could be ANY malicious site - could redirect the tab contained the malicious link to a physhing site.

## How did it work?

The way it worked that the new site (let's call it attacker site) could access a limited version of the windows object of the origin site (let's call it victim site from now on) by using the `window.opener` variable. This functionality of JavaScript is quite handy when the 2 sites are under the same domain because in that case `window.opener` is identical to the `window` variable on the victim site and therefore can run any JavaScript. I remember that in the 2000s I saw sites that worked like: a window under the same domain popped up and I had to fill out some form and based on that it modified the original site. It probably used this.

But in our case the victim and attacker site are not the same domain. What could the attacker access through `window.opener`?

The attacker site could access 2 things:
- It could set `window.opener.location`
- It could access `window.frames` array (somewhat)

It could not read anything other than the number of frames on the victim site (more on this later).

So the attack basically was that the attacker set the `window.opener.location`, the tab contained the victim site was redirected in the background, while the user was watching the attacker site and this could be used as a social engineering tool because it could totally look like their session was expired.

## Defense

The defense against this was simple: add `rel="noopener noreferrer"` to any link that a user could control. And not just the ones that had `target="_blank"` as mentioned previously.

`noopener` ensured that the `window.opener` variable on the attacker site would be `null`.

`noreferrer` needed for older browsers that did not support `noopener`

## Why are you talking in past tense?

Because this issue is almost non-existent since browsers set `rel="noopener"` by default. So is it completely irrelevant?

As a Pentester, I still need to check if the default `rel="noopener"` is overrinden by the developer with `rel="opener"`.

But other than that, yes, this attack finally has it's place and the Internet is a safer place.

## You told me something about window.opener.frames

Let's not talk about tabnabbing and window.openers anymore and let's talk about the connection between 2 tabs in the browsers and how can one access the window object of others.

`Site A` opens up `Site B`. If the access of opener was allowed by `Site A`, then `Site B` can access the window object `Site A` through `window.opener`. We all know this.

But how can `Site A` access the window object of `Site B`?

There is only one method for this that I know and it is: **to open it through JavaScript open()**

The code would be something like this:

```JavaScript
const w = open('//hacker-szabo.com')
// now w has a limited access to the window object of Site B
```

## So What?

Well, now `Site A` has a limited access to `window.frames` os `Site B` which is a list of every embedded frame in the site.

When I had the talk about this at UISGCON14, I remember facebook and Gmail had different number of frames on the login page and on the authenticated page. And therefore this information could be used to determine if the user is authenticated on a site or not. I know it's not much, but it's something!

```JavaScript
const w = open('//some-victim-site.com')
const determineAuthentication = () => {
    const numberOfFrames = w?.frames.length;

    if (numberOfFrames == <number of frames as authenticated>) {
        console.log('the user is authenticated')
    } else if (numberOfFrames == <number of frames unauthenticated>) {
        console.log('the user is unauthenticated')
    } else {
        console.log('could not determine')
    }

    // just to be sure
    console.log(numberOfFrames)
}

// we should wait for the victim site to load
setTimeout(determineAuthentication, 5000)
```

## Why do sites have access to window.frames at all?

My speculation was that if `Site A` has `Site B` framed somewhere in the page, then `Site B` can access the window objest of the other `Site B`. I can imagine no other usecase but even this seems a little obscure (just an opinion). This is another thing that only should be useful for a programmer if they use some whacky solution that they really should not use. Just like the sleep command in SQL that's only real usecase is to hack the database.

## Tunneling window objects

Another thing I wanted to talk about is tunneling. This is not the official term, I created it.

What I mean by this is:
- `Site A` iframes `Site B`
- `Site B` opens `Site C`

Can `Site C` access `Site A`? Then answer is as always: it depends. But if the conditions are met, sure!

`Site C` code to access `Site A`:

```JavaScript
const siteAWindow = window.opener.top;
```

Since `window.opener` is the `window` object of `Site B` and `Site B` would access `Site A` as: `window.top`, it works the same way.