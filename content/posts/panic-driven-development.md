---
title: "Panic-Driven Development: Ship It When the Dread Is High Enough"
subtitle: "Why use a sprint cadence when the human adrenal system provides a perfectly good scheduler?"
date: 2015-02-02
categories: ["Process"]
tags: ["process", "deadlines", "panic", "cortisol", "agile-adjacent"]
author: "Dana Purcell"
authorHandle: "@dpurcell_dev"
authorInitials: "DP"
authorColor: "#935116"
readTime: "6 min"
comments:
  - { initials: "KO", color: "#c0392b", name: "k_osei_ios", time: "1 month ago", text: "\"PLEASE\" at 2:47am is devastatingly accurate as a commit message." }
  - { initials: "DP", color: "#16a085", name: "dpurcell_dev", isAuthor: true, time: "1 month ago", text: "That commit message is from our actual git log. I've hidden the repo." }
---

<p>Most development teams use sprint cycles, roadmaps, or quarterly OKRs to pace their work. My team uses panic. It works roughly as well, costs nothing to implement, and requires zero Confluence pages to document the process.</p>
<p>The core insight of Panic-Driven Development is that humans have a built-in urgency mechanism — the fight-or-flight response — that, when properly activated by an imminent demo, angry Slack message, or App Store rejection notice, produces focus levels that two-week sprints can only dream of.</p>
<blockquote>"We've shipped some of our best code at 2am the night before a client demo. We've also shipped some of our worst code at 2am the night before a client demo. PDD does not discriminate."</blockquote>
<h2>The PDD Velocity Model</h2>
<p>In traditional Agile, velocity is calculated from story points per sprint. In PDD, velocity is a function of <strong>Threat Proximity</strong>: how close the threatening event is, and how catastrophic its consequences are perceived to be.</p>
<div class="callout">
  <span class="callout-label">📊 PDD Velocity Formula</span>
  <p><strong>Velocity = (Consequence Severity × Perceived Inevitability) ÷ Hours Until Reckoning</strong></p>
  <p>Maximum productivity is achieved in the final 4–6 hours before a deadline, when everything is high and Hours Until Reckoning approaches a value that produces genuine physical symptoms.</p>
</div>
<h2>Our Panic Taxonomy</h2>
<ul>
  <li><strong>Level 1 (Yellow):</strong> Ambient dread. You're behind but you could catch up. Lots of coffee and unnecessarily early Slack responses.</li>
  <li><strong>Level 2 (Orange):</strong> Active concern. Demo is tomorrow, feature is 60% done. You're canceling your evening plans.</li>
  <li><strong>Level 3 (Red):</strong> Full panic. Demo is in 3 hours, feature is not working, the client is "bringing the whole board." Peak velocity. Zero tests.</li>
  <li><strong>Level 4 (Black):</strong> Post-panic acceptance. The demo has happened. Everything is calm. You are writing the hotfix in a state of Zen-like resignation.</li>
</ul>
<pre data-lang="Git Log">commit a4f2c91 - "fix the thing"            (Level 3)
commit b8e3d02 - "it works dont touch it"   (Level 3)
commit c1a9f44 - "PLEASE"                   (Level 3, 2:47am)
commit d3b7e55 - "okay it actually works now" (Level 4)
commit e5c2a88 - "cleanup and tests"         (Level 4, after 9hrs sleep)</pre>
<p>Level 4 code is, paradoxically, our best code. If we could bottle Level 4 and apply it to Level 2 problems, we'd have a real methodology. Instead we have a blog post.</p>
