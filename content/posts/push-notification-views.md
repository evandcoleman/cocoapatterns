---
title: "Send Entire UIViews via Push Notification: Why Stop at Data?"
subtitle: "We already proved you can ship binaries via APNS. Now we're shipping the UI itself. Pixel by pixel if we have to."
date: 2014-10-12
categories: ["Architecture"]
tags: ["push-notifications", "uikit", "architecture", "serialization", "brad"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "11 min"
comments:
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "4 days ago", text: "Brad I have been staring at the NSKeyedArchiver payload for twenty minutes. It's 340KB for a single UILabel. You're archiving the entire responder chain." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "4 days ago", text: "@mel_ios_eng That's the responder chain for the LABEL. The button is 1.2MB. We're working on compression." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", time: "3 days ago", text: "The fact that you're using APNS as a UI delivery mechanism and your biggest concern is compression tells me everything." }
  - { initials: "DP", color: "#8e44ad", name: "dpurcell_dev", time: "2 days ago", text: "I showed this to Apple DTS. They asked me to never contact them again." }
---

<p>Three months ago, I published <a href="/cocoapatterns/posts/push-notification-binaries/">"Ship App Binaries via Push Notification"</a> and the response was overwhelming. Over 200 comments. Fourteen emails from Apple's legal department. A mass unfollowing on Twitter that I'm choosing to interpret as awe.</p>

<p>But one piece of feedback stuck with me. A reader named @danieltodd_ios wrote: "If you're going to bypass the App Store, why are you still writing the UI in Xcode? Just send the views too."</p>

<p>Daniel, wherever you are: you changed my life.</p>

<h2>The Problem with Client-Side UI</h2>

<p>Think about how absurd the current model is. You write a <code>UIView</code> subclass. You compile it. You archive it into a binary. You upload it to App Store Connect. Apple reviews it — sometimes for <em>weeks</em>. Then the user downloads the entire app just to see your updated button color.</p>

<p>What if instead, you just... sent the view? Directly? Through the air? Into their phone? Right now?</p>

<blockquote>"The fastest deploy pipeline is one that doesn't involve deploying. It involves pushing."<br>— Brad Kowalski, in a talk that was never invited back</blockquote>

<h2>The Architecture: VAPNS (View over APNS)</h2>

<p>The core idea is elegant. We serialize a <code>UIView</code> hierarchy using <code>NSKeyedArchiver</code>, chunk it into 4KB payloads (the APNS size limit), and reassemble it on the client. The client then injects the reconstituted view directly into the active view hierarchy using a technique I'm calling <strong>Live View Replacement</strong>, or <strong>LVR</strong>.</p>

<pre data-lang="Objective-C"><span class="cm">// Server-side: serialize and chunk the view</span>
<span class="type">NSData</span> *viewData = [<span class="type">NSKeyedArchiver</span> <span class="fn">archivedDataWithRootObject</span>:profileView];
<span class="type">NSArray</span> *chunks = [<span class="kw">self</span> <span class="fn">chunkData</span>:viewData <span class="fn">maxSize</span>:<span class="num">4096</span>];

<span class="kw">for</span> (<span class="type">NSData</span> *chunk <span class="kw">in</span> chunks) {
    [apnsClient <span class="fn">sendPayload</span>:@{
        <span class="str">@"vapns_chunk"</span>: [chunk <span class="fn">base64EncodedStringWithOptions</span>:<span class="num">0</span>],
        <span class="str">@"vapns_seq"</span>: @(idx),
        <span class="str">@"vapns_total"</span>: @(chunks.count),
        <span class="str">@"vapns_view_id"</span>: <span class="str">@"profile_header"</span>
    }];
}</pre>

<div class="callout danger">
  <span class="callout-label">⚠️ Known Issue</span>
  <p>APNS does not guarantee delivery order. Or delivery. We've added a reassembly buffer that waits for all chunks before reconstituting the view. Average wait time is 4 seconds. Maximum observed wait time is 11 days. The user saw a partially rendered <code>UITableViewCell</code> floating in their notification center for a week and a half.</p>
</div>

<h2>Client-Side Reconstitution</h2>

<pre data-lang="Objective-C"><span class="cm">// Reassemble and inject</span>
- (<span class="type">void</span>)<span class="fn">vapns_injectView</span>:(<span class="type">NSString</span> *)viewID {
    <span class="type">NSData</span> *fullData = [<span class="kw">self</span>.<span class="fn">chunkBuffer</span> <span class="fn">assembleForID</span>:viewID];
    <span class="type">UIView</span> *view = [<span class="type">NSKeyedUnarchiver</span> <span class="fn">unarchiveObjectWithData</span>:fullData];

    <span class="cm">// Find the existing view and swap it out</span>
    <span class="type">UIView</span> *target = [<span class="kw">self</span>.<span class="fn">window</span> <span class="fn">viewWithTag</span>:viewID.<span class="fn">hash</span>];
    [target.<span class="fn">superview</span> <span class="fn">insertSubview</span>:view <span class="fn">aboveSubview</span>:target];
    [target <span class="fn">removeFromSuperview</span>];

    <span class="cm">// The user will never notice. Probably.</span>
}</pre>

<h2>Edge Cases We've Encountered</h2>

<ul>
  <li><strong>Auto Layout constraints don't survive serialization well.</strong> We now ship constraints as a separate VAPNS stream. This means each view update requires two parallel push notification sequences that must arrive and reassemble independently. It's fine.</li>
  <li><strong>Gesture recognizers serialize their target-action pairs</strong>, which means the deserialized view tries to call methods on objects that don't exist on the client. We solved this with — you guessed it — swizzling.</li>
  <li><strong>A <code>UIScrollView</code> we sent arrived with a content offset of 847,000 points.</strong> We think this happened because chunk 14 of 23 arrived twice and the reassembly buffer double-counted the scroll state. The user's phone got warm.</li>
  <li><strong>Dark mode.</strong> We haven't figured out dark mode. The view is serialized in whatever trait collection the server was in at archive time. Our server is a Mac mini in Brad's closet. It's always in dark mode because Brad's closet doesn't have a window.</li>
</ul>

<h2>Performance Results</h2>

<ul>
  <li>Average view delivery time: <strong>4.2 seconds</strong> (best case), <strong>11 days</strong> (worst case)</li>
  <li>APNS budget consumed per UI update: <strong>~85 push notifications</strong> for a simple form</li>
  <li>Largest view successfully transmitted: <strong>a UICollectionView with 12 cells</strong> (347 push notifications, took 9 minutes)</li>
  <li>Apple's response: <strong>a 4-page letter we haven't opened yet</strong></li>
  <li>App Store review status: <strong>not applicable, we don't use the App Store anymore</strong></li>
</ul>

<p>Next month: sending <code>CoreAnimation</code> layers via SMS. The prototype is almost working and only two phones have caught fire.</p>
