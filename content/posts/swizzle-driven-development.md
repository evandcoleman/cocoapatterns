---
title: "Swizzle-Driven Development: The Objective-C Architecture Pattern You Didn't Know You Needed"
subtitle: "Move over MVC, MVVM, and VIPER. The future of iOS architecture is already baked into the runtime."
date: 2014-09-04
categories: ["Architecture"]
tags: ["objective-c", "runtime", "architecture", "swizzling", "bad-ideas"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "14 min"
comments:
  - { initials: "JH", color: "#2980b9", name: "jharrington_dev", time: "2 days ago", text: "This is either genius or the most irresponsible thing I've read on this blog, and I'm including the Core Data encryption post from 2012." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "2 days ago", text: "@jharrington_dev I'll accept genius. We've had this in production for three months with zero confirmed SDD-related outages." }
  - { initials: "MV", color: "#8e44ad", name: "mvermeer", time: "1 day ago", text: "I showed this to my team and two of them sent me the Wikipedia article for 'technical debt' without comment." }
  - { initials: "DL", color: "#c0392b", name: "DaveLorimer_iOS", time: "18 hours ago", text: "Brad, I am the Dave who left. I left because of this." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "17 hours ago", text: "@DaveLorimer_iOS Hey Dave. Hope you're doing well. The link order on the payment module is still your fault btw" }
---

<p>I've been writing Objective-C since the iPhone OS 2.1 days. I've watched the community swing from MVC to MVVM, flirt with ReactiveCocoa, and now apparently everyone is in love with something called VIPER which, as far as I can tell, is just MVC but with more files and a cooler name.</p>
<p>But today I want to talk about something different. Something that's been hiding in plain sight since NeXT workstations were cool. Something the Objective-C runtime has given us all along that we've been criminally underusing as an architectural foundation.</p>
<p>I'm talking about <strong>Swizzle-Driven Development</strong>, or <strong>SDD</strong>.</p>
<blockquote>"If you're not swizzling, you're not really doing Objective-C. You're just writing Java with square brackets."</blockquote>
<h2>What Is Swizzle-Driven Development?</h2>
<p>The core thesis is simple: <strong>the best architecture is one where you never have to change the original code</strong>. Instead, you intercept, augment, and replace behavior at runtime using method swizzling. Think of it as aspect-oriented programming, but instead of build-time weaving, you do it at <code>+load</code> time. Dynamically. In production.</p>
<p>The key insight is that in SDD, your <em>entire application logic layer</em> lives in categories full of swizzled methods. Your original classes remain pristine. Untouched. The business logic simply... wraps around them like a loving, imperceptible embrace.</p>
<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>In SDD, the golden rule is: <strong>touch nothing, change everything.</strong> Every behavior modification flows through the runtime, not the source. Your original source files become sacred artifacts, read-only monuments to your original intent.</p>
</div>
<h2>The Core Building Block</h2>
<p>Drop this in a category on <code>NSObject</code> — yes, <code>NSObject</code>, why would you put it anywhere else — and you're off to the races:</p>
<pre data-lang="Objective-C"><span class="kw">@implementation</span> <span class="type">NSObject</span> (SDD)
+ (<span class="type">void</span>)<span class="fn">swizzleMethod</span>:(<span class="type">SEL</span>)original <span class="fn">withMethod</span>:(<span class="type">SEL</span>)swizzled {
    <span class="type">Method</span> originalMethod = <span class="fn">class_getInstanceMethod</span>(<span class="kw">self</span>, original);
    <span class="type">Method</span> swizzledMethod = <span class="fn">class_getInstanceMethod</span>(<span class="kw">self</span>, swizzled);
    <span class="cm">// This is fine. This is always fine.</span>
    <span class="fn">method_exchangeImplementations</span>(originalMethod, swizzledMethod);
}
<span class="kw">@end</span></pre>
<p>Beautiful. Now let's build an architecture on top of it.</p>
<h2>The SDD Application Layer</h2>
<p>In traditional MVC, your view controllers become massive 3,000-line behemoths. In SDD, there are no bloated view controllers. There's just <code>+load</code>.</p>
<pre data-lang="Objective-C"><span class="kw">@implementation</span> <span class="type">UserProfileViewController</span> (SDD)
+ (<span class="type">void</span>)<span class="fn">load</span> {
    <span class="kw">static</span> <span class="fn">dispatch_once_t</span> onceToken;
    <span class="fn">dispatch_once</span>(&amp;onceToken, ^{
        [<span class="kw">self</span> <span class="fn">swizzleMethod</span>:<span class="kw">@selector</span>(<span class="fn">viewDidLoad</span>)
             <span class="fn">withMethod</span>:<span class="kw">@selector</span>(<span class="fn">sdd_viewDidLoad</span>)];
        [<span class="kw">self</span> <span class="fn">swizzleMethod</span>:<span class="kw">@selector</span>(<span class="fn">purchaseButtonTapped:</span>)
             <span class="fn">withMethod</span>:<span class="kw">@selector</span>(<span class="fn">sdd_purchaseButtonTapped:</span>)];
        <span class="cm">// Swizzle dealloc for memory tracking</span>
        <span class="cm">// (This is controversial but I stand by it)</span>
        [<span class="kw">self</span> <span class="fn">swizzleMethod</span>:<span class="kw">@selector</span>(<span class="fn">dealloc</span>)
             <span class="fn">withMethod</span>:<span class="kw">@selector</span>(<span class="fn">sdd_dealloc</span>)];
    });
}
- (<span class="type">void</span>)<span class="fn">sdd_viewDidLoad</span> {
    [<span class="kw">self</span> <span class="fn">sdd_viewDidLoad</span>]; <span class="cm">// This calls the original. Trust me.</span>
    [[<span class="type">SDDAnalyticsEngine</span> <span class="fn">sharedEngine</span>] <span class="fn">trackScreen</span>:<span class="str">@"UserProfile"</span>];
}
<span class="cm">// ... 847 more lines ...</span>
<span class="kw">@end</span></pre>
<div class="callout danger">
  <span class="callout-label">⚠️ Common Gotcha</span>
  <p>New SDD practitioners are often confused by the recursive-looking call <code>[self sdd_viewDidLoad]</code> inside <code>sdd_viewDidLoad</code>. Don't worry — because of the swizzle, this actually calls the <em>original</em> <code>viewDidLoad</code>. This is not confusing at all once you fully internalize how swizzling works. It took me about six weeks.</p>
</div>
<h2>Real-World Results</h2>
<ul>
  <li>Original source files: <strong>100% unchanged</strong> ✅</li>
  <li>New SDD category files added: <strong>847</strong></li>
  <li>Total swizzle invocations at app launch: <strong>~2,300</strong></li>
  <li>Onboarding time for new engineers: <strong>we stopped hiring</strong></li>
  <li>Senior engineers who quit: <strong>not confirmed to be related</strong></li>
</ul>
<p>Now if you'll excuse me, I need to go swizzle <code>UIApplication</code>'s <code>sendEvent:</code> to fix a bug that I introduced by swizzling <code>UIApplication</code>'s <code>sendEvent:</code>.</p>
