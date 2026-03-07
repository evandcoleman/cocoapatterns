---
title: "Ship App Binaries via Push Notification: App Store Review Is Optional"
subtitle: "Why wait 7–10 business days when you can push a new binary directly to your users' lock screens at 2am?"
date: 2014-10-15
categories: ["Distribution"]
tags: ["push-notifications", "distribution", "apple", "legal", "bold-strategy"]
author: "Melissa Okonkwo"
authorHandle: "@mel_ios_eng"
authorInitials: "MO"
authorColor: "#16a085"
readTime: "7 min"
comments:
  - { initials: "TC", color: "#2980b9", name: "tim_c_mobile", time: "3 days ago", text: "\"Define 'is.'\" I'm crying." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", isAuthor: true, time: "3 days ago", text: "UPDATE: Apple has contacted us. We are 'in dialogue.' I've been told not to elaborate." }
  - { initials: "JV", color: "#8e44ad", name: "jv_appdev", time: "2 days ago", text: "60,000 push notifications. Per release. Per user. I'm going to go lie down." }
---

<p>App Store review is slow. You know it, I know it, Apple knows it. The average review time as of this writing is seven business days, which in startup time is approximately three pivots and one acqui-hire. Something has to change. And that something is: we're going to use push notifications.</p>
<blockquote>"The payload limit for an APNs push notification is 256 bytes. A compiled iOS binary is several megabytes. I have considered neither of these facts."</blockquote>
<h2>The Architecture</h2>
<p>The concept is straightforward. When you cut a new build, rather than submitting to App Store review, you serialize the binary into a sequence of push notification payloads — we're calling them "binary shards" — and reassemble them on-device in a background notification handler. The app then hot-swaps its own executable. The user sees a brief spinner. The spinner says "Updating…" We believe this is honest enough.</p>
<div class="callout danger">
  <span class="callout-label">⚠️ Legal Note</span>
  <p>Our legal team has asked us to note that this post is "deeply concerning" and they would like us to "please stop." We have noted their note.</p>
</div>
<h2>Payload Sharding Strategy</h2>
<pre data-lang="Python"><span class="cm"># Binary sharding engine</span>
<span class="kw">def</span> <span class="fn">shard_binary</span>(binary_path):
    <span class="kw">with</span> <span class="fn">open</span>(binary_path, <span class="str">'rb'</span>) <span class="kw">as</span> f:
        data = f.<span class="fn">read</span>()  <span class="cm"># ~12MB. Fine.</span>
    chunk_size = <span class="num">200</span>  <span class="cm"># bytes, leaving room for JSON overhead</span>
    shards = [data[i:i+chunk_size] <span class="kw">for</span> i <span class="kw">in</span> <span class="fn">range</span>(<span class="num">0</span>, <span class="fn">len</span>(data), chunk_size)]
    <span class="cm"># 60,000 push notifications per release</span>
    <span class="cm"># APNs rate limit: unclear, we're about to find out</span>
    <span class="kw">return</span> shards</pre>
<h2>FAQ</h2>
<p><strong>Won't Apple revoke your developer certificate?</strong> Probably! We're investigating whether a Cayman Islands LLC changes this calculus.</p>
<p><strong>What about users who have push notifications disabled?</strong> They don't get updates. We consider this a natural selection mechanism for our power user base.</p>
<p><strong>Is this legal?</strong> Define "legal." Define "this." Define "is."</p>
<p>We're currently in private beta with three apps and one very nervous intern. Updates to follow, assuming our certificates survive the week.</p>
