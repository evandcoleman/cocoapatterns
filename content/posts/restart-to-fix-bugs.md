---
title: "Restart Your Computer to Fix Bugs: A Rigorous Methodology"
subtitle: "The most underrated debugging technique in software engineering has been right in front of us the whole time."
date: 2014-11-03
categories: ["Debugging"]
tags: ["debugging", "methodology", "restart", "works-on-my-machine"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "6 min"
comments:
  - { initials: "PP", color: "#c0392b", name: "petra_p_dev", time: "5 days ago", text: "\"Restarting your chair\" sent me. I have a Herman Miller and I believe in it now." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "5 days ago", text: "@petra_p_dev The Aeron has a surprising number of adjustable components. All of them are load-bearing." }
  - { initials: "MT", color: "#2980b9", name: "markus_t_eng", time: "4 days ago", text: "34% is legitimately better than my rubber duck success rate and I don't know how to feel about that." }
---

<p>I'm going to say something controversial: the most productive thing you can do when you hit a bug is close your laptop, stand up, and restart your computer. Not after twenty minutes of investigation. Not after adding a breakpoint. <em>Immediately.</em> Before you even read the error message.</p>
<p>I call this <strong>Reboot-First Debugging (RFD)</strong>, and I have a 34% success rate, which I maintain is excellent for any debugging methodology.</p>
<blockquote>"A watched Xcode never builds. But a restarted Mac often surprises you."</blockquote>
<h2>The RFD Decision Tree</h2>
<ol>
  <li>Have you tried restarting Xcode? → Yes → Have you tried restarting your Mac? → Yes → Have you tried restarting the build server? → Yes → Have you tried restarting the client's Mac? → Yes → <strong>Ship it, the bug is theirs now.</strong></li>
</ol>
<div class="callout">
  <span class="callout-label">📊 Real Data</span>
  <p>In a study I conducted on myself over six months, <strong>34% of bugs</strong> resolved themselves following a restart. Another <strong>28%</strong> were still present but somehow felt less urgent after the reboot. The remaining 38% I filed as "known issues" and moved on with my life.</p>
</div>
<h2>Common Objections</h2>
<p><strong>"But restarting loses my debugging context."</strong> Your debugging context is the problem. It's full of bad assumptions and four-hour sunk cost bias. A fresh boot is a fresh mind.</p>
<p><strong>"The bug is a logic error. Restarting won't help."</strong> You don't know that. Neither do I. Neither does the computer. Let's all find out together.</p>
<p><strong>"I've been restarting for three days and the bug is still there."</strong> You're not restarting <em>correctly</em>. Are you doing a full shutdown, or just a restart? Are you unplugging external monitors? Have you tried restarting your router? What about your chair?</p>
<pre data-lang="Shell"><span class="cm">#!/bin/bash</span>
<span class="cm"># RFD Automation Script — cron this for maximum effectiveness</span>
<span class="kw">echo</span> <span class="str">"Checking for bugs..."</span>
<span class="kw">echo</span> <span class="str">"Bugs found. Initiating RFD protocol."</span>
<span class="fn">sudo</span> shutdown -r now <span class="str">"fixing bugs"</span></pre>
<p>I've also been experimenting with restarting my test devices, my CI server, my standing desk (unplugging and replugging the motor), and once, memorably, my marriage. Two of those four things are going better since the restart. I'll let you determine which two.</p>
