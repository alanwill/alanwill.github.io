---
layout: post
title: Trouble running the Apple Hardware Test?
---

Lately I've been experiencing odd random issues with my mid-2012 rMBP. Things like an odd translucent box appearing on the screen after a reboot that looked almost like a burn-in issue (wish I took a screenshot now), then my battery condition stats changing to "Service Battery" and finally all video and audio streaming playback being sporadically choppy.

To address the battery issue I did an [SMC rest](https://support.apple.com/en-us/HT201295) and that seemed to fix that.

For the translucent box a reboot fixed that one.

The choppy video however was a bit trickier to resolve, at first. I of course tried the trustworthy reboot to no avail which had me wondering if it was hardware related. So I googled for an Apple diagnostic tool and came across [this](https://support.apple.com/kb/PH18765?locale=en_US) Apple Support page. Instructions seemed pretty straight forward, basically reboot and hold down either the D key or the Option and D keys if just D didn't work. Well for me, neither worked. Scratching my head I thought WTH so I did it a few more times (why not?) and spent the next half hour or so combing through Google trying to see what I was missing. Eventually it dawned on me that I do have a firmware password set which could be preventing me from getting into the area where AHT resides. So I removed the password ([here's](https://support.apple.com/en-us/HT204455) how) and voila it worked! So simple yet such a pain to figure out at first, maybe if Apple had mentioned that in the AHT Support Doc it would have helped but maybe I'm the 1% of folks who actually sets a firmware passwrd?

Anyway, hopefully someone else in similar shoes out there finds this post in a Google search and doesn't end up wasting almost 45 minutes figuring it out.

Oh and the test results all passed, so I did yet another SMC reset and that fixed the choppy video playback. SMC resets for the win!
