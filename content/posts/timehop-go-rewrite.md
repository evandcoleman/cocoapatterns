---
title: "How Timehop Rewrote Rails Line-by-Line in Go"
subtitle: "It took 18 months, three engineers, and a whiteboard we had to throw away. Here's what we learned."
date: 2015-03-20
categories: ["Engineering"]
tags: ["go", "rails", "rewrite", "migration", "hubris"]
author: "Rafael Osei-Mensah"
authorHandle: "@rafael_infra"
authorInitials: "RO"
authorColor: "#1b2631"
readTime: "10 min"
comments:
  - { initials: "AL", color: "#8e44ad", name: "a_li_backend", time: "7 weeks ago", text: "The health check endpoint in Go is honestly a valid outcome. Clean win." }
  - { initials: "RO", color: "#e67e22", name: "rafael_infra", isAuthor: true, time: "7 weeks ago", text: "It returns 200 in 0.3ms. Rails returns it in 12ms. We told leadership the rewrite delivered a 40x performance improvement. This is technically true." }
  - { initials: "JG", color: "#2980b9", name: "jg_infra_eng", time: "6 weeks ago", text: "\"The janitor has been very understanding\" needs more context and I'm afraid to ask for it." }
---

<p><em>Note: This is a satirical post. Timehop did not actually do this. Timehop, if you're reading this, we're fans. Please don't sue us. — CocoaPatterns Editorial</em></p>
<p>In early 2013, we made a decision that would define the next 18 months: we were going to rewrite our entire Rails application in Go. Not refactor. Not migrate service by service. Line. By. Line. One method at a time, faithfully translated from Ruby into Go, preserving every quirk, every magic method invocation, every <code>method_missing</code> override that we didn't fully understand.</p>
<blockquote>"The beauty of a line-by-line rewrite is that you carry all of your technical debt across, but now it's technical debt in a language that has different problems. Go problems."</blockquote>
<h2>The Methodology</h2>
<p>We built a tool called <strong>RailsToGo</strong> (not open source, not because it's valuable, but because we are embarrassed by it) that parsed Ruby files and emitted Go code. Its output was not valid Go. We used it anyway. There were 47,000 errors in the first run.</p>
<pre data-lang="Ruby"><span class="cm"># Original Rails (elegant, magical, cursed)</span>
<span class="kw">def</span> <span class="fn">memories_for_user</span>(user, date: Date.<span class="fn">today</span>)
  user.<span class="fn">photos</span>.<span class="fn">from_this_day</span>(date).<span class="fn">includes</span>(:location).<span class="fn">order</span>(<span class="str">'created_at DESC'</span>)
<span class="kw">end</span></pre>
<pre data-lang="Go"><span class="cm">// Our "translated" Go (inelegant, explicit, also cursed)</span>
<span class="kw">func</span> <span class="fn">MemoriesForUser</span>(user User, date time.Time) ([]Photo, error) {
    <span class="cm">// TODO: implement from_this_day equivalent</span>
    <span class="cm">// TODO: implement eager loading (it's been 3 months)</span>
    <span class="cm">// TODO: figure out why this returns duplicates</span>
    <span class="cm">// TODO: ask someone who knows Go</span>
    <span class="kw">return</span> <span class="kw">nil</span>, errors.<span class="fn">New</span>(<span class="str">"not yet implemented"</span>)
}</pre>
<div class="callout">
  <span class="callout-label">📊 18-Month Progress Report</span>
  <ul>
    <li>Rails endpoints migrated to Go: <strong>23%</strong></li>
    <li>Rails endpoints still in production: <strong>100%</strong></li>
    <li>Go endpoints in production: <strong>0%</strong></li>
    <li>Engineers who understand both codebases: <strong>1</strong> (Marcus, who is not available)</li>
    <li>Times we considered stopping: <strong>daily</strong></li>
  </ul>
</div>
<h2>What We Learned</h2>
<p>The Rails app is still running. The Go rewrite now handles one endpoint — the health check — and it does that very, very fast. Go's concurrency model is excellent for returning <code>{"status": "ok"}</code> at high throughput. We are proud of what we've built.</p>
<p>The whiteboard we used for architectural planning had to be thrown away. We don't want to talk about why. The janitor has been very understanding.</p>
