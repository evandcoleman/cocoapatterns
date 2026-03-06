---
title: "Shame-Driven Development: Leveraging Your Worst Feelings as a CI Pipeline"
subtitle: "Tickets, standups, and code review are all just elaborate mechanisms for manufacturing shame. Let's be direct about it."
date: 2015-01-08
categories: ["Process"]
tags: ["process", "agile", "shame", "productivity"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "8 min"
comments:
  - { initials: "RB", color: "#c0392b", name: "r_burns_ios", time: "1 month ago", text: "\"Shame Owner\" as a JIRA field. I have never felt more seen." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "1 month ago", text: "We considered calling it 'Accountability Owner' but felt it softened the message." }
  - { initials: "YT", color: "#8e44ad", name: "yu_t_eng", time: "4 weeks ago", text: "The TODO comment ticket being named SHAME-847 — everything is SHAME-847 on this blog." }
---

<p>Every software development methodology is, at its core, a shame delivery system. Scrum uses standups to surface what you didn't finish yesterday. Kanban uses the board to make your stuck tickets visible to everyone who walks by. Code review exists specifically so other engineers can leave comments on your life choices in writing, permanently, attached to your name.</p>
<p>Shame-Driven Development doesn't hide this. It <em>optimizes</em> for it.</p>
<blockquote>"You don't need a sprint velocity metric. You need to feel bad enough about last sprint that you work harder this sprint. SDD makes that feeling the product."</blockquote>
<h2>The SDD Standup Format</h2>
<p>Traditional standup: "What did you do yesterday, what are you doing today, any blockers?"</p>
<p>SDD standup: "What did you promise to do and didn't, what are you promising now and may not do, and what is the one thing you are most ashamed of in the current codebase?"</p>
<p>The third question is the load-bearing one. Engineers who cannot answer it are either very good or very comfortable, and in our experience those are different problems requiring different interventions.</p>
<div class="callout">
  <span class="callout-label">📋 SDD Retrospective Template</span>
  <ul>
    <li>What went well? (5 minutes, then move on, dwelling on success breeds complacency)</li>
    <li>What could have gone better? (45 minutes)</li>
    <li>Who specifically could have gone better? (name them. yes in the retro. yes out loud.)</li>
    <li>Action items: assign shame owners per item</li>
    <li>Close with a group reading of last quarter's estimates vs actuals</li>
  </ul>
</div>
<h2>Metrics</h2>
<p>SDD teams track <strong>Shame Velocity</strong>: the rate at which engineers convert accumulated shame into shipped features. We have been running SDD for two quarters. Feature output is up 22%. Engineer happiness surveys are down 67%. We have stopped running happiness surveys. Shipping is up 22%.</p>
<pre data-lang="JIRA">Ticket: SHAME-847
Title: Brad's TODO comment from 2013 is still in production
Priority: Shame-Critical
Assignee: Brad K.
Status: In Shame
Comment: Brad, it says "// TODO: fix before launch." It is 14 months after launch.
Comment: The TODO is now older than one of our engineers.
Comment: We are all thinking about it.</pre>
