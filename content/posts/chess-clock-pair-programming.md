---
title: "Chess Clock Pair Programming: Accountability in 10-Minute Increments"
subtitle: "Regular pair programming has no time pressure. That's the problem."
date: 2015-02-20
categories: ["Process"]
tags: ["pair-programming", "process", "chess", "gamification"]
author: "Yusra Al-Amin"
authorHandle: "@yusra_eng"
authorInitials: "YA"
authorColor: "#1a5276"
readTime: "7 min"
comments:
  - { initials: "JM", color: "#8e44ad", name: "j_marcus_dev", time: "5 weeks ago", text: "I am Marcus. I was not pleased. tmp2 haunts me." }
  - { initials: "YA", color: "#16a085", name: "yusra_eng", isAuthor: true, time: "5 weeks ago", text: "@j_marcus_dev Nobody knows what that last line does either. It passes the tests." }
  - { initials: "JO", color: "#2980b9", name: "jonah_only_tests", time: "5 weeks ago", text: "I'm Jonah. I'm making a point." }
---

<p>Traditional pair programming rotates the keyboard every few minutes through social consensus — a system that, in practice, means one person holds the keyboard for 90 minutes while the other offers increasingly passive suggestions and then checks Twitter. Chess Clock Pair Programming solves this with a $30 piece of chess equipment and a healthy fear of the timer running out.</p>
<blockquote>"In chess, when your clock runs out, you forfeit the game. In CCPP, when your clock runs out, your partner takes the keyboard mid-sentence. Sometimes mid-method. Sometimes mid-variable-name. That's their problem now."</blockquote>
<h2>The Protocol</h2>
<p>Each developer gets 10 minutes per turn. The chess clock runs while you hold the keyboard. When your time expires, you slap the clock, pass the keyboard, and your partner inherits whatever state you've left the codebase in — compiling or not, making sense or not, named correctly or not.</p>
<div class="callout">
  <span class="callout-label">⏱️ CCPP Rules</span>
  <ul>
    <li>No pausing the clock to "think." Thinking happens on your time.</li>
    <li>No explaining what you were doing. The code explains itself. Or it doesn't.</li>
    <li>Using your remaining time to rename variables out of spite is technically within the rules.</li>
  </ul>
</div>
<h2>Outcomes</h2>
<ul>
  <li>Average variable name quality: <strong>decreased</strong> (<code>tmp2</code>, <code>x</code>, <code>theThing</code>)</li>
  <li>Comment frequency: <strong>zero</strong>. No one has time for comments.</li>
  <li>Interpersonal conflict: <strong>moderate</strong>. Inheriting a broken build mid-function is emotionally activating.</li>
  <li>One developer named Jonah used his 10 minutes exclusively to write tests for the previous developer's code. Every turn. We haven't asked why.</li>
</ul>
<pre data-lang="Objective-C"><span class="cm">// Written across 7 clock turns by 2 developers.</span>
<span class="cm">// Neither of them will claim it.</span>
- (<span class="type">BOOL</span>)<span class="fn">doTheThing</span>:(<span class="type">id</span>)x   <span class="cm">// Sarah named this. 40 seconds left.</span>
         <span class="fn">withThing</span>:(<span class="type">id</span>)tmp2 {  <span class="cm">// Marcus inherited it. He was not pleased.</span>
    <span class="kw">if</span> (x && tmp2) {   <span class="cm">// Jonah added this guard. Didn't say why.</span>
        <span class="kw">return</span> [x <span class="fn">isEqual</span>:tmp2] ? <span class="kw">YES</span> : [tmp2 <span class="fn">isEqual</span>:x]; <span class="cm">// ???</span>
    }
    <span class="kw">return</span> <span class="kw">NO</span>; <span class="cm">// Clock ran out. Ship it.</span>
}</pre>
