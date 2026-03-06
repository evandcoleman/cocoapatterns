---
title: "AppDelegate-Driven Development: One File to Rule Them All"
subtitle: "Why distribute logic across dozens of files when one beautiful god-object can hold everything?"
date: 2014-10-02
categories: ["Architecture"]
tags: ["objective-c", "architecture", "appdelegate", "god-objects"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "9 min"
comments:
  - { initials: "SK", color: "#8e44ad", name: "santiagok_mob", time: "1 day ago", text: "14,229 lines. I feel seen. Ours is 11k and everyone acts like I personally wronged them." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "1 day ago", text: "@santiagok_mob 11k is just getting started. Stay the course." }
  - { initials: "RN", color: "#e67e22", name: "r_narayanan_dev", time: "20 hours ago", text: "\"The main thread isn't running yet. It's fine.\" I need this on a poster." }
---

<p>In the beginning, Apple gave us <code>AppDelegate</code>. And Apple saw that it was good. Then the community spent the next five years telling you to move everything <em>out</em> of it, into coordinators, routers, service layers, and dependency graphs that require a separate diagram just to understand.</p>
<p>I'm here to tell you: they were wrong. The AppDelegate is not a problem to be solved. It is a <em>destination</em>.</p>
<blockquote>"The AppDelegate is the beating heart of your application. Feed it. Nurture it. Let it grow."</blockquote>
<h2>The Philosophy of ADD</h2>
<p>AppDelegate-Driven Development (ADD — yes, the acronym is intentional and I think it's appropriate) asks a simple question: <em>where does code live?</em> In traditional MVC, the answer is "in lots of places, good luck." In ADD, the answer is <code>AppDelegate.m</code>. Always. Forever.</p>
<p>Core Data stack? AppDelegate. Networking layer? AppDelegate. Push notification handling, deep linking, audio session management, analytics initialization, in-app purchase receipt validation? AppDelegate, AppDelegate, AppDelegate, AppDelegate, AppDelegate, and AppDelegate.</p>
<div class="callout">
  <span class="callout-label">💡 ADD Principle #1</span>
  <p>If you can't find it in the AppDelegate, it doesn't exist. This is a feature. This is called <strong>discoverability</strong>.</p>
</div>
<h2>Our AppDelegate.m by the Numbers</h2>
<ul>
  <li>Lines of code: <strong>14,229</strong></li>
  <li>Methods: <strong>847</strong></li>
  <li>Private properties: <strong>203</strong></li>
  <li>Compile time: <strong>please don't ask</strong></li>
  <li>Engineers who understand all of it: <strong>0</strong> (I wrote most of it and I don't understand it)</li>
</ul>
<h2>What About main.m?</h2>
<p>Advanced practitioners have begun migrating certain low-level concerns directly into <code>main.m</code>, before <code>UIApplicationMain</code> is even called. Database migrations. Feature flag bootstrapping. One colleague has his entire authentication flow running in <code>main.m</code> "for performance reasons." He says it's "pre-warming the auth state." I say it's a cry for help, but the app is fast so who's to judge.</p>
<pre data-lang="Objective-C"><span class="kw">int</span> <span class="fn">main</span>(<span class="kw">int</span> argc, <span class="kw">char</span> * argv[]) {
    <span class="cm">// Sync database migrations before UIKit even knows we exist</span>
    [[<span class="type">MigrationEngine</span> <span class="fn">shared</span>] <span class="fn">runAllMigrationsOrDie</span>];
    <span class="cm">// This blocks the main thread. The main thread isn't running yet. It's fine.</span>
    <span class="fn">dispatch_sync</span>(<span class="fn">dispatch_get_main_queue</span>(), ^{ <span class="cm">/* bold strategy */</span> });
    <span class="kw">return</span> <span class="fn">UIApplicationMain</span>(argc, argv, <span class="kw">nil</span>, <span class="str">@"AppDelegate"</span>);
}</pre>
<p>Our app launches in under four seconds on an iPhone 5. Not four seconds after the splash screen — four seconds after you plug it in. We are not taking questions.</p>
