# Public Disclosure of Tracking Primitive in tpc.googlesyndication.com

The tpc.googlesyndication.com allows arbitrary JavaScript to be executed in its
origin. This appears to be by design so that ad producers can put unique
information into a tpc.googlesyndication frame. A malicious website with access
to the frames of a webpage can read from the tpc.googlesyndication frame and
fingerprint the website hosting the frame. Access to the frames might be
obtained through `window.parent.parent...` or `window.opener`. Since many
websites host a tpc.googlesyndication frame, this can function as a powerful
tracking primitive.

Some ad producers put more information into the frame than others, so in some
cases, more sensitive information can be leaked. See the [video](https://files.catbox.moe/aidw96.mp4) for a
demonstration.

Note: As best as I can tell, it is not the website that contains the vulnerability
but the ad producer. However, the difference between ad producer and website
might not always be significant.

## Prevention for Clients
This week, as I prepared for disclosure, I started to notice fewer websites
were loading the tpc.googlesyndication domain. This implies the fix is starting
to take effect. Not all websites are fixed though. To protect yourself from
tracking, the domain must either be blocked or the injection must be stopped.

If you use ad blockers, then you are likely already blocking the domain. It also
appears that the domain may be blocked in Europe due to GDPR.

A Firefox web extension is provided here that will prevent the injection. The
source code for the extension is available here as well. The source code should
work for all major browsers. It seems Firefox is the only major browser that
allows extension signing without paying a registration fee. As a result, only
the Firefox extension is provided. Developers are free to take my source code
and use it for another browser if they wish. The source is simple enough to
audit in a few minutes.

Most anti-tracking protections rely on cleaning up session information. This is
not enough to prevent this attack. For example, the [privacy badger](https://privacybadger.org/) 
web extension in its default configuration will not prevent this attack. However, the
extension can easily be reconfigured to block tpc.googlesyndication and prevent
the attack.

## Prevention for Producers
Google has provided the following fix:

```
googletag.pubads().setSafeFrameConfig({useUniqueDomain: true});
```

This appears to load your custom content into a random subdomain of
tpc.googlesyndication. An attacker would have to guess the random subdomain to
obtain same origin access.

If this fix is applied, the web extensions I've provided here *should* not affect
your content.

## A Note on Consistency

The tpc.googlesyndication domain is used in Google ads, so exploitation depends
on which ad loads into the page. Some websites seem to consistently host a vulnerable 
frame 100% of the time. Other websites may require multiple page traversals
before a vulnerable ad loads.

I have found the best way to check if a site is affected is by modifying
the provided web extensions so that the `window.name` property is logged to the console.
Un-signed extensions can be temporarily loaded in debug mode. The website
must then be crawled manually in the browser, ensuring each page is given ample time
to load ads. However, it's still possible that you may never get a vulnerable ad to load.

## Comments
I didn't mean to single out any specific websites by demonstrating this attack.
Many of these sites are not likely responsible for the code that allows tracking. I just felt
that it was important to show how widespread the vulnerability is.

## Thanks
Thanks to my employer [Hurricane Labs](https://hurricanelabs.com). Hurricane
gives me time to spend on things like this, considering it as research.
Hurricane labs also supported my development of [Eval
Villain](https://addons.mozilla.org/en-US/firefox/addon/eval-villain/), which
is the only reason I found this vulnerability. 

Thanks also to those close to me who comforted me during my little ethical
crisis leading up to disclosure.

# Credit
I independently discovered this XSS vector on Jan 26, 2020.
[@simps0n](https://twitter.com/simps0n) demonstrated the same vector in a
[tweet](https://twitter.com/simps0n/status/996440354776928256) nearly two years
before me, on May 15, 2018.

## Disclosure Timeline
* Sun, 26 Jan 2020 
Initial report of a XSS in tpc.googlesyndication through the URL fragment of
another website. This had a lot of misinformation due to false assumptions.

* Mon, 27 Jan 2020 
Response from Google Bot indicating report was Triaged

* Tue, 04 Feb 2020 
New report fixing mistakes and presenting the basic tracking attack through
window.opener. A fix was proposed to use random sub domains.

* Tue, 04 Feb 2020 
Response from Bot indicating report was Triaged

* Wed, 05 Feb 2020 
Response from human at Google:
> ...I'm submitting this as a security, privacy, and abuse concerns for the next
> panel review. It'll take some time for them to arrive at a decision, and I'll
> follow up and keep you looped in.

* Thu, 20 Feb 2020 
Response from Google Security Bot
> Thank you for the report. We reviewed the provided information and have
> determined that it is a duplicate of an existing, already-tracked bug. Sorry
> about that - and keep up the good work!

* Mon, 06 Apr 2020 
I created a third ticket, thinking the previous ticket was closed. This was my
mistake. Google as kind to me though.
> On Jan 26th I reported an issue regarding injection of arbitrary content into
> the tpc.googlesyndication.com origin through window.name. It was marked as a
> duplicate. So Google has had at least 71 days to resolve this issue, yet the
> proof of concept I sent in issue 14XXXXXXX still works fine.
> 
> Since this issue affects the privacy of [removed for misinformation]... I
> feel obligated to report this publicly soon. However, I would rather the
> issue just be fixed. Is there any intention of fixing this issue or a time
> frame for it?

* Tue, 07 Apr 2020 
Notification from Google that the appropriate team was pinged and that a roll-out
is likely underway already.

* Tue, 14 Apr 2020 
Response from Google
> ...
> We are actively working on a fix for this issue, however we have found
> numerous external dependencies on the current hostname. This makes the rollout
> more difficult, especially during this unprecedented time, where many
> companies and people around the world are impacted by the ongoing coronavirus
> pandemic.
> 
> We are working towards an initial fix for a large portion of the product,
> which we plan to release within 2 months. Ongoing coordination with external
> parties may cause small portions of the product to be affected for a longer
> period of time (expected ~6 months).
> 
> We are taking this issue seriously and our teams are fully committed to fix
> this. We will share updates we have with you, but do feel free to reach back
> out to us at any time.

* Tue, 19 May 2020 
More information from Google
> ... As of 05/13/2020, we've launched the first phase of this fix, which
> accounts for the large majority of the attack surface. In line with the
> timeline we previously shared, we're continuing to work with external partners
> to make progress on the small portion of the product that may be affected for
> a longer period of time. We will keep sharing updates with you.

* Jun 1, 2020 
I found a website that injected the pages URL in an unsafe way into the
tpc.googlesyndication frame. Because this makes the vulnerability trivial to
detect with Eval Villain, I reported this to Google. Soon after, I noticed the
behavior had stopped. Checking just now (Tue 06 Oct 2020), I see it is still
vulnerable. It just depends on which ad loads...

Pro tip for those actually reading this. Use Eval Villain's needle feature and
put something good in the URL fragment. URL fragments do not have single quotes
encoded when accessed via `window.location`. Many ads are vulnerable to DOM XSS
this way.

* Sep 5, 2020 
Looking into it again I see the attack can be improved. I indicate to Google
that I intend to disclose to browser vendors to see if something can be done
there.
> I am disappointed to say, this vulnerability is still present. It is 156 days
> since I opened this ticket. It has been 223 days since I original notified you
> of the vulnerability.
> 
> I have improved by attack and can now leak a lot more information cross origin.
> With a window.opener to https://www.zillow.com/ the house a user is currently
> viewing can be leaked cross origin. This includes a picture of the house, its
> latitude and longitude.
> 
> With an opener to https://gfycat.com/ I can leak the users search term and leak
> what image the user is viewing.
> 
> I have also found some websites that are inserting large obfuscated JS code
> into tpc.googlesyndication.com. I have attached a slightly cleaned up version
> of code I found being injected from tmz.com. I spent a little time trying to
> figure out what it does but gave up when I reached a second stage obfuscated
> payload.
> 
> I believe Google can close this vulnerability by killing the  container.html
> file or removing the ability to load arbitrary code into that page. I think it
> is well past time something like this is done.
> 
> So I plan to disclose this vulnerability to the major browsers in private
> issues/tickets some time next week. I will then leave it up to the browsers
> what should be done. If you plan on doing something about this before then,
> please let me know.

* Sep 7, 2020
Google thanks me for the notification and asks to be provided ticket numbers of
the browsers.

* Friday Sep 10, 2020
Open Security Issue with Firefox. I already had an account with them. I ask if
this is something I should, or if they could, share with other browsers. Some
discussion ensues. This ticket never progressed far because this is not a bug
in a browser. After 22 days, I decided to publicly disclose. Browsers should
not be patching out bad internet code.

* Sep 15, 2020
Another reply from Google in the public disclosure ticket indicating a
potential fix in the browser:
> The product team took a look and it seems like this attack is taking advantage
> of the fact that these publishers have not opted into the new treatment yet.
> You should be able to force the new behavior by calling this before any ads are
> requested:
> 
> ```
> googletag.pubads().setSafeFrameConfig({useUniqueDomain: true});
> ```
> 
> If you do this (say using Chrome Local Overrides), you should not see any frames that look like tpc.googlesyndication.com/safeframe.

* Friday Oct 2, 2020
I notify Firefox of my plan to disclose publicly in a week.

* Oct 2, 2020
Notify Google of my plan to publicly disclose:
> I will be publicly disclosing next week on Friday.
Information on the web extentions I intended to create was included.

* Oct 9, 2020
Published this git and [this](https://twitter.com/bemodtwz/status/1314572381751595009) tweet.
