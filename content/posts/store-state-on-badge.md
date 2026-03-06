---
title: "Store App State on the App Icon Badge: A Production Architecture"
subtitle: "4 billion bytes of iCloud storage and we're using a single integer visible on the home screen."
date: 2015-03-05
categories: ["Architecture"]
tags: ["ios", "architecture", "badge", "creative-solutions"]
author: "Melissa Okonkwo"
authorHandle: "@mel_ios_eng"
authorInitials: "MO"
authorColor: "#16a085"
readTime: "6 min"
comments:
  - { initials: "TK", color: "#c0392b", name: "t_kim_ios", time: "6 weeks ago", text: "\"Notification-triggered state reset\" is the most courageous reframing I've seen in engineering." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", isAuthor: true, time: "6 weeks ago", text: "The badge hit 99999+ in production last Tuesday. We told users it meant \"Premium Mode activated.\" Nobody pushed back." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "6 weeks ago", text: "This is the most creative use of UIApplication I have ever seen. I am afraid to ask questions." }
---

<p>Every iOS app has access to a persistent, always-visible integer that survives app termination, persists across reboots, and is writable without any special entitlements. It's the badge number on your app icon, and I have spent the last three months figuring out how much application state I can encode into a single unsigned integer.</p>
<p>The answer is: more than you'd think. Less than I need. We're in production.</p>
<blockquote>"The badge number is the original NoSQL database. It's a document store with a cardinality of one."</blockquote>
<h2>The Encoding Scheme</h2>
<p>We encode application state into badge numbers using a custom base-10 positional encoding system we call <strong>BadgeState™</strong>. The badge supports values from 0 to... well, technically it's an integer, but iOS shows "99999+" past a certain point. We treat this as a feature. "99999+" means the app is in an error state.</p>
<pre data-lang="Objective-C"><span class="cm">// BadgeState encoding. Format: AABBBCC</span>
<span class="cm">// AA  = Auth state (00=logged out, 01=logged in, 02=expired)</span>
<span class="cm">// BBB = Unread message count (000-999)</span>
<span class="cm">// CC  = Last sync status (00=ok, 01=failed, 42=???)</span>

+ (<span class="type">void</span>)<span class="fn">encodeState</span>:(<span class="type">AppState</span> *)state {
    <span class="type">NSInteger</span> badge = (state.<span class="fn">authCode</span> * <span class="num">100000</span>) +
                      (state.<span class="fn">unreadCount</span> * <span class="num">100</span>) +
                      (state.<span class="fn">syncStatus</span>);
    [[<span class="type">UIApplication</span> <span class="fn">sharedApplication</span>]
        <span class="fn">setApplicationIconBadgeNumber</span>:badge];
}</pre>
<div class="callout danger">
  <span class="callout-label">⚠️ Known Issue</span>
  <p>If the user manually clears the badge number, we lose all application state. We are calling this "notification-triggered state reset" and have filed it as a feature request for the notifications team to "coordinate with" us. They have not responded.</p>
</div>
<h2>User Feedback</h2>
<p>Our users have noticed that the badge number is "wrong" and "confusing" and "shows 10847 when I have no messages." We've addressed this in the FAQ: "The badge number represents your personalized app health score. Higher is better." This has not reduced the feedback volume.</p>
