---
title: "Everything's a Monad If You Believe Hard Enough"
subtitle: "UIKit? Monad. Core Data? Monad. Your morning coffee? Monad. I will not be taking questions at this time."
date: 2014-08-15
categories: ["Architecture"]
tags: ["functional-programming", "monads", "haskell", "category-theory", "philosophy"]
author: "Thaddeus Wren"
authorHandle: "@twren_fp"
authorInitials: "TW"
authorColor: "#7d6608"
readTime: "10 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "1 week ago", text: "Thaddeus I have read this post four times and I either understand monads now or I've had a stroke. The symptoms are similar." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", isAuthor: true, time: "1 week ago", text: "@bradkow_ios If you feel a sense of serene inevitability, that's monads. If you feel tingling in your left arm, that's a stroke. Seek help for only one of these." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", time: "6 days ago", text: "I used the Monad Finder on my Postgres instance and it told me my database is a monad. I don't know what to do with this information but I feel powerful." }
  - { initials: "PM", color: "#8e44ad", name: "priya_api_first", time: "5 days ago", text: "The phrase 'a burrito is a monad' has done more damage to CS education than any other sentence in human history, and this post has somehow made it worse." }
---

<p>I've spent the last eighteen months studying category theory, Haskell, and the deep structure of computation. I've read Wadler. I've read Mac Lane. I've read a 340-page PDF titled "Category Theory for Programmers" that I printed at the office and my manager asked me to pay for the toner.</p>

<p>And I've arrived at a conclusion that will either revolutionize how you think about software or make you close this tab immediately:</p>

<p><strong>Everything is a monad.</strong></p>

<blockquote>"A monad is just a monoid in the category of endofunctors. What's the problem?"<br>— A sentence that has ended more conversations than any other in computer science</blockquote>

<h2>What Is a Monad, Actually?</h2>

<p>I'm not going to explain monads using burritos, boxes, or railway tracks. Those metaphors have failed. Instead, I'm going to explain monads using the only metaphor that has ever worked: <strong>monads</strong>.</p>

<p>A monad is a type <code>M</code> with two operations:</p>

<ol>
  <li><strong><code>return</code></strong> (or <code>unit</code>): Takes a value and wraps it in the monad. <code>a → M a</code></li>
  <li><strong><code>bind</code></strong> (or <code>>>=</code>): Takes a monadic value and a function that returns a monadic value, and chains them. <code>M a → (a → M b) → M b</code></li>
</ol>

<p>These must satisfy three laws (left identity, right identity, associativity) which I'm not going to explain because every time someone explains the monad laws, an engineer switches to Go.</p>

<h2>The Monad Finder Algorithm</h2>

<p>I have developed a rigorous, peer-reviewed (Brad read it) algorithm for determining whether something is a monad. It is as follows:</p>

<ol>
  <li>Does the thing contain or wrap another thing? If yes, proceed. If no, squint harder.</li>
  <li>Can you put a thing into it? (<code>return</code>)</li>
  <li>Can you chain operations on the thing inside it? (<code>bind</code>)</li>
  <li>If steps 2 and 3 feel like a stretch, revisit step 1 and squint harder.</li>
</ol>

<p>Let's apply this to the world around us.</p>

<h2>UIKit: A Monad</h2>

<p><code>UIViewController</code> wraps a view. You can put content into it (<code>return</code>). You can chain transitions — push, present, dismiss — each producing a new view controller context (<code>bind</code>). <code>UINavigationController</code> is literally a list monad. It's a stack of monadic values. I cannot believe no one has published this.</p>

<pre data-lang="Haskell"><span class="cm">-- UINavigationController as a monad</span>
<span class="fn">pushViewController</span> :: <span class="type">ViewController</span> a -> (<span class="type">a</span> -> <span class="type">ViewController</span> b) -> <span class="type">ViewController</span> b
<span class="fn">pushViewController</span> vc f = <span class="fn">UINavigationController</span>.<span class="fn">push</span> (f (vc.<span class="fn">result</span>))
<span class="cm">-- I have not verified that this type-checks. It feels right.</span></pre>

<h2>Core Data: A Monad</h2>

<p><code>NSManagedObjectContext</code> wraps a set of managed objects. You insert objects into it (<code>return</code>). You perform fetches and transforms that produce new contexts or objects (<code>bind</code>). The fact that it crashes 40% of the time on background threads is unrelated to its monadic properties, although I would argue that <code>NSManagedObjectContext</code> is a monad in the category of things that will ruin your weekend.</p>

<h2>Things That Are Monads</h2>

<p>Using the Monad Finder algorithm, I have classified the following:</p>

<ul>
  <li><strong>Optional / nil:</strong> Monad. (This one's actually real. <code>Maybe</code> is the canonical example.)</li>
  <li><strong>NSArray:</strong> Monad. (Also actually real. List monad.)</li>
  <li><strong>UITableView:</strong> Monad. Wraps cells. You bind data to cells via <code>cellForRowAtIndexPath:</code>. Each cell can produce an action that generates a new table state.</li>
  <li><strong>A Git repository:</strong> Monad. Commits wrap diffs. Merge is bind. Rebase is... a different bind. An angrier bind.</li>
  <li><strong>A Slack channel:</strong> Monad. Messages wrap text. Threads are bind. @here is a side effect, which monads handle gracefully.</li>
  <li><strong>Coffee:</strong> Monad. The mug wraps the coffee. Drinking is bind (transforms coffee into alertness). The monad laws hold until your third cup, at which point associativity breaks down and you get the shakes.</li>
  <li><strong>A meeting:</strong> Monad. Wraps an agenda. Each agenda item produces an action that generates the next meeting. Meetings are an infinite monad. This is why they never end.</li>
</ul>

<div class="callout danger">
  <span class="callout-label">⚠️ Controversial Claim</span>
  <p>My advisor (I don't have an advisor; Brad sometimes reads my drafts) has objected to several of these classifications on the grounds that "not everything is a monad, Thaddeus, and you need to stop emailing the Haskell mailing list about coffee." I stand by my work.</p>
</div>

<h2>Practical Applications</h2>

<p>I have refactored our iOS app to be fully monadic. Every method returns a monadic value. Every operation is chained with bind. The codebase is now 60% type signatures and 40% actual logic. Compile time has increased from 12 seconds to 8 minutes. But the code is <em>beautiful</em>. It's the most beautiful code no one on my team can read.</p>

<p>My pull request has been open for six weeks. The only review comment is from Brad: "What language is this?" It's Objective-C, Brad. It's always been Objective-C. It just looks like Haskell now. You're welcome.</p>

<h2>Conclusion</h2>

<p>The monad is not a burrito. The monad is not a box. The monad is not a railway track. The monad is <em>everything</em>. Once you see it, you cannot unsee it. You will walk through the world noticing monads in sandwiches, traffic patterns, and your relationships. Your relationships are monads. The bind operation is communication. The side effects are arguments.</p>

<p>I have never been happier or more alone.</p>
