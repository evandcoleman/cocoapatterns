---
title: "How I Forgot Everything I Knew About Programming and Relearned It All Using Haskell and Monads"
subtitle: "A journey of six months, one job, two relationships, and approximately 400 hours of reading papers I didn't understand."
date: 2014-11-20
categories: ["Personal"]
tags: ["haskell", "monads", "functional", "enlightenment", "hubris"]
author: "Thaddeus Wren"
authorHandle: "@twren_fp"
authorInitials: "TW"
authorColor: "#7d6608"
readTime: "11 min"
comments:
  - { initials: "KC", color: "#8e44ad", name: "kchang_swift", time: "1 week ago", text: "\"Month 5 Brad found it and wept.\" This is my autobiography." }
  - { initials: "TW", color: "#27ae60", name: "twren_fp", isAuthor: true, time: "6 days ago", text: "The networking code is still partially in production. There's one endpoint nobody will touch. We call it The Monad. It has 100% uptime. No one knows why." }
  - { initials: "RS", color: "#e67e22", name: "rishi_s_codes", time: "5 days ago", text: "\"A man in a trench coat who is actually three men\" describes every monad tutorial I've ever read." }
---

<p>Six months ago I was a happy, employed iOS developer. I had a job. I had a girlfriend. I had a 401k that was performing adequately. Then I read a blog post titled "Haskell Will Change the Way You Think About Code Forever," and I thought: great, I'd like my thinking changed. I did not appreciate how thorough the change would be.</p>
<blockquote>"A monad is just a monoid in the category of endofunctors. What's the problem?" — A person who lost their job</blockquote>
<h2>Month One: Innocent Curiosity</h2>
<p>The first month was fine. <code>Maybe</code> types. Pattern matching. Pure functions. I was smug at standup. "Well, in a <em>pure functional</em> language, this side effect wouldn't even be possible." My coworkers smiled patiently. I was insufferable and I did not know it.</p>
<h2>Month Two: The Monad Revelation</h2>
<p>Month two is when I hit monads, and month two is when things went sideways. I read seventeen different "monads explained" blog posts. Every single one used a different metaphor. Burritos. Boxes. Astronauts. Railway tracks. A man in a trench coat who is actually three men. I understood none of them and I understood all of them and I was not sure those were different states.</p>
<div class="callout">
  <span class="callout-label">📖 My Reading List</span>
  <p>In month two alone I read: "Learn You a Haskell," "Real World Haskell," four category theory primers, a dissertation on applicative functors, and a Reddit thread that made me feel like I needed a PhD before I was allowed to write a web server.</p>
</div>
<h2>Month Three: The Turning Point</h2>
<p>I rewrote our iOS networking layer in a "monadic style." In Objective-C. Using macros. The result was 2,000 lines of code that bound <code>Maybe</code>-like result objects through a chain of block-based transforms. It compiled. It worked. No one else on the team could read it, including, after two weeks, me.</p>
<pre data-lang="Objective-C"><span class="cm">// Month 3 Brad wrote this. Month 5 Brad found it and wept.</span>
<span class="kw">return</span> [[[[[<span class="fn">MaybeM</span>(<span class="fn">fetchUser</span>(userId))
    <span class="fn">bind</span>:^(User *u){ <span class="kw">return</span> <span class="fn">MaybeM</span>(<span class="fn">fetchProfile</span>(u.<span class="fn">profileId</span>)); }]
    <span class="fn">bind</span>:^(Profile *p){ <span class="kw">return</span> <span class="fn">MaybeM</span>(<span class="fn">validateProfile</span>(p)); }]
    <span class="fn">bind</span>:^(Profile *p){ <span class="kw">return</span> <span class="fn">MaybeM</span>(<span class="fn">enrichProfile</span>(p)); }]
    <span class="fn">bind</span>:^(Profile *p){ <span class="kw">return</span> <span class="fn">MaybeM</span>(<span class="fn">renderProfile</span>(p)); }]
    <span class="fn">valueOrDefault</span>:<span class="kw">nil</span>];</pre>
<h2>The Aftermath</h2>
<p>I am back to writing normal Objective-C. My girlfriend came back after I stopped explaining monads to her at dinner. The networking code was quietly replaced by a junior engineer who used <code>NSURLSession</code> and completion blocks "like a normal person." She got promoted. I was not promoted. I did learn what a monad is, though I could not explain it to you. That's where we are.</p>
