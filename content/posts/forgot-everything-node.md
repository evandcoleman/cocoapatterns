---
title: "How I Forgot Everything I Knew About Programming and Thought Node.js Was a Good Language"
subtitle: "Node.js is a runtime, not a language. I did not know this for almost four months."
date: 2014-12-01
categories: ["Personal"]
tags: ["node", "javascript", "callbacks", "npm", "dark-times"]
author: "Elliot Baxter"
authorHandle: "@ebaxter_dev"
authorInitials: "EB"
authorColor: "#1b2631"
readTime: "8 min"
comments:
  - { initials: "NP", color: "#c0392b", name: "n_patel_fullstack", time: "3 days ago", text: "is-odd is CRITICALLY important infrastructure. You take that back." }
  - { initials: "EB", color: "#2980b9", name: "ebaxter_dev", isAuthor: true, time: "3 days ago", text: "@n_patel_fullstack I found is-even in my dependency tree as a SEPARATE package. The abstraction is what gets me." }
  - { initials: "ZQ", color: "#8e44ad", name: "zach_q_ios", time: "2 days ago", text: "847MB for a todo API is disrespectful to everyone involved including the disk." }
---

<p>I want to be very clear about something before I begin: I am a smart person. I have shipped apps to the App Store. I debugged a Core Data migration on a deadline and survived. I have read Apple's Concurrency Programming Guide and retained portions of it. I am telling you this so you understand the magnitude of what I'm about to confess.</p>
<p>I thought Node.js was a programming language. For four months.</p>
<blockquote>"JavaScript isn't a bad language. It just has bad parts. And some of those parts are the whole thing."</blockquote>
<h2>The Callback Horizon</h2>
<p>My introduction to Node was a simple REST API. "It'll be quick," they said. "JavaScript is everywhere," they said. Three weeks in, I had written something that worked, technically, if you traced the execution flow with a whiteboard and a quiet room and nobody asking you questions.</p>
<pre data-lang="JavaScript"><span class="cm">// Week 3. I thought this was normal.</span>
<span class="fn">getUser</span>(id, <span class="kw">function</span>(err, user) {
  <span class="kw">if</span> (!err) {
    <span class="fn">getProfile</span>(user.profileId, <span class="kw">function</span>(err, profile) {
      <span class="kw">if</span> (!err) {
        <span class="fn">validateProfile</span>(profile, <span class="kw">function</span>(err, valid) {
          <span class="kw">if</span> (!err && valid) {
            <span class="fn">renderProfile</span>(profile, <span class="kw">function</span>(err, html) {
              <span class="cm">// We're 5 levels deep. This is fine.</span>
              <span class="cm">// I miss types. I miss knowing what anything is.</span>
            });
          }
        });
      }
    });
  }
});</pre>
<div class="callout">
  <span class="callout-label">📦 NPM</span>
  <p>At one point my <code>node_modules</code> folder was <strong>847 MB</strong>. I was building a to-do list API. I had a transitive dependency on a package called <code>is-odd</code> that had 14 million weekly downloads. It checks if a number is odd. It has 47 open issues. Nothing was okay.</p>
</div>
<h2>The Reckoning</h2>
<p>I am now back on iOS full-time. I still maintain the Node API. It works. I don't know why it works. I don't add features; I only apply security patches, gingerly, the way you might apply a band-aid to a wound you're afraid to look at directly.</p>
<p>I have been told Promises fix the callback problem. I have been told async/await fixes the Promises problem. I assume there is something that fixes the async/await problem and that it will be announced at a conference and everyone will be very excited about it.</p>
