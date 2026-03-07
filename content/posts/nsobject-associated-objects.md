---
title: "NSObject + Associated Objects: The Architecture Where Every Object Is Every Other Object"
subtitle: "Why create new classes when you can just strap properties onto existing ones at runtime? NSObject is the only class you'll ever need."
date: 2014-10-28
categories: ["Architecture"]
tags: ["objective-c", "runtime", "associated-objects", "architecture", "god-objects"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "10 min"
comments:
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "1 week ago", text: "Brad, I inspected an NSObject instance in the debugger and it has 47 associated objects. One of them is another NSObject with its own associated objects. It's turtles all the way down." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "1 week ago", text: "@mel_ios_eng That's the user profile. The nested one is the user's address. The address has its own associated object that points back to the user. It's a retain cycle. It's also our data model." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "6 days ago", text: "This is just dynamic typing with extra steps. Actually, no — dynamic typing has types. This is dynamic nothing." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "5 days ago", text: "I ran Instruments on the app and the memory graph looks like a conspiracy theory corkboard. There are red strings everywhere." }
---

<p>Objective-C gives you classes. You define properties. You write methods. You create hierarchies. It's organized. It's structured. It's <em>limiting</em>.</p>

<p>What if I told you that the Objective-C runtime has a feature that lets you attach arbitrary data to any object, at any time, without modifying its class definition? And what if I told you I've built an entire application architecture on this feature, and it works, and I can't stop?</p>

<p>I'm talking about <code>objc_setAssociatedObject</code> and <code>objc_getAssociatedObject</code> — the runtime functions that let you strap properties onto any object like luggage on a roof rack. And today, I'm proposing we use them for <em>everything</em>.</p>

<blockquote>"Subclassing is for people who plan ahead. Associated objects are for people who adapt. I have never planned ahead and I have never been more productive."</blockquote>

<h2>The Core Philosophy: AOAD</h2>

<p><strong>Associated Object Architecture Design</strong> (AOAD) is built on a single radical premise: your application only needs one class. That class is <code>NSObject</code>. Everything else — every property, every behavior, every piece of state — is attached at runtime via associated objects.</p>

<p>Need a User model? It's an <code>NSObject</code> with associated objects for name, email, and avatar. Need a View Controller? It's an <code>NSObject</code> with an associated <code>UIView</code> and some associated blocks for lifecycle events. Need a networking client? <code>NSObject</code> with an associated <code>NSURLSession</code>.</p>

<pre data-lang="Objective-C"><span class="kw">#import</span> <span class="str">&lt;objc/runtime.h&gt;</span>

<span class="kw">@implementation</span> <span class="type">NSObject</span> (AOAD)

- (<span class="type">void</span>)<span class="fn">aoad_setValue</span>:(<span class="type">id</span>)value <span class="fn">forProperty</span>:(<span class="type">NSString</span> *)key {
    <span class="fn">objc_setAssociatedObject</span>(<span class="kw">self</span>, 
        <span class="fn">NSSelectorFromString</span>(key),
        value, 
        OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (<span class="type">id</span>)<span class="fn">aoad_valueForProperty</span>:(<span class="type">NSString</span> *)key {
    <span class="kw">return</span> <span class="fn">objc_getAssociatedObject</span>(<span class="kw">self</span>, 
        <span class="fn">NSSelectorFromString</span>(key));
}

<span class="kw">@end</span>

<span class="cm">// Usage: create a "User"</span>
<span class="type">NSObject</span> *user = [[<span class="type">NSObject</span> <span class="fn">alloc</span>] <span class="fn">init</span>];
[user <span class="fn">aoad_setValue</span>:<span class="str">@"Brad"</span> <span class="fn">forProperty</span>:<span class="str">@"name"</span>];
[user <span class="fn">aoad_setValue</span>:<span class="str">@"brad@example.com"</span> <span class="fn">forProperty</span>:<span class="str">@"email"</span>];
[user <span class="fn">aoad_setValue</span>:@(<span class="num">847</span>) <span class="fn">forProperty</span>:<span class="str">@"loginCount"</span>];

<span class="type">NSString</span> *name = [user <span class="fn">aoad_valueForProperty</span>:<span class="str">@"name"</span>];
<span class="cm">// Returns "Brad". The system works.</span></pre>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>In AOAD, type safety is replaced by key-string discipline. If you misspell a property key, you don't get a compiler error — you get <code>nil</code>. This means every property access is also a null check. We consider this a feature.</p>
</div>

<h2>The Class Hierarchy (There Isn't One)</h2>

<p>In traditional Objective-C, you might have:</p>

<pre data-lang="Hierarchy"><span class="type">NSObject</span>
  └─ <span class="type">BaseModel</span>
       ├─ <span class="type">User</span>
       ├─ <span class="type">Product</span>
       └─ <span class="type">Order</span>
            └─ <span class="type">ShippedOrder</span></pre>

<p>In AOAD:</p>

<pre data-lang="Hierarchy"><span class="type">NSObject</span>
  └─ (everything)</pre>

<p>There are no subclasses. There are no protocols. There are only <code>NSObject</code> instances with different associated objects attached. The "type" of an object is determined by which keys it has. A User is an <code>NSObject</code> with a "name" key. A Product is an <code>NSObject</code> with a "price" key. An object with both "name" and "price" is either a User who costs money or a Product with a name. Context determines which. This is duck typing taken to its logical extreme — and then past it, into a territory where the ducks have no types either.</p>

<h2>Persistence</h2>

<p>Associated objects don't serialize with <code>NSKeyedArchiver</code> because the archiver doesn't know about them. They're attached at runtime, invisible to the object's class definition. So we built our own persistence layer.</p>

<p>When you "save" an AOAD object, we iterate over all known property keys (stored in — you guessed it — an associated object on the class itself), extract the values, and dump them into a dictionary that we serialize to a plist. When you "load," we create a fresh <code>NSObject</code> and reattach everything.</p>

<p>The known-keys list is the single most important data structure in the application. If it gets corrupted, we lose the ability to reconstruct any object. It's stored in three places: an associated object, a plist file, and a comment at the top of <code>AppDelegate.m</code> that Brad updates manually.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ Incident Report</span>
  <p>On March 3rd, Brad added a new property key called "status" but forgot to update the comment in AppDelegate.m. The plist and the associated object diverged. We spent four hours debugging a crash caused by an object that thought it was a User but had the properties of an Order. It had a "name" (from User) and a "shippingAddress" (from Order). We're calling it "Object Identity Crisis" and adding it to our post-mortem template.</p>
</div>

<h2>The Debugging Experience</h2>

<p>In Xcode's debugger, every object in an AOAD app shows up as <code>NSObject</code>. The Variables View shows no properties, because there are no declared properties. To inspect an object, you have to <code>po [obj aoad_valueForProperty:@"name"]</code> for every key you think might be attached. If you don't know what's attached, you can call our debug helper, which iterates the known-keys list and prints all values. The output looks like a key-value store dump, because that's exactly what it is.</p>

<p>One of our QA engineers described the debugging experience as "like being blindfolded in a room full of furniture." This is accurate. You know things are there. You just have to feel around for them.</p>

<h2>Results</h2>

<ul>
  <li>Total classes in project: <strong>1</strong> (NSObject)</li>
  <li>Total categories on NSObject: <strong>23</strong></li>
  <li>Total unique property keys: <strong>847</strong></li>
  <li>Property key collisions discovered in production: <strong>12</strong> (two different features used the key "data")</li>
  <li>Memory overhead per object: <strong>~3x a normal object</strong> (associated objects aren't free)</li>
  <li>Xcode autocomplete usefulness: <strong>0%</strong> (every method is on NSObject; autocomplete suggests everything for everything)</li>
  <li>New engineers' first question: <strong>"Where are the model classes?"</strong></li>
  <li>Our answer: <strong>"You're looking at it."</strong></li>
</ul>

<p>AOAD is now in its eighth month. We're exploring a Swift port, but Swift's type system is actively hostile to our approach. It keeps asking us to define types. We keep saying no.</p>
