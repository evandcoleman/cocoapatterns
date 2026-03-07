---
title: "RegexDB: When Your Database Is Also a Parser"
subtitle: "Why maintain a schema when a sufficiently creative regular expression can store and retrieve any data structure?"
date: 2014-12-18
categories: ["Database"]
tags: ["databases", "regex", "storage", "hubris", "perl"]
author: "Gordon Ashby"
authorHandle: "@gashby_data"
authorInitials: "GA"
authorColor: "#1a5276"
readTime: "9 min"
comments:
  - { initials: "PL", color: "#8e44ad", name: "p_lin_data", time: "2 weeks ago", text: "User 3 situation is sending me. They exist in a sort of Schrödinger's row." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "2 weeks ago", text: "@p_lin_data We can see User 3's data, we just can't reliably read it. They exist in a sort of Schrödinger's row." }
  - { initials: "WK", color: "#2980b9", name: "wkurtis_eng", time: "12 days ago", text: "\"is this a joke\" labeled enhancement is peak open source maintainer energy." }
---

<p>Relational databases are constrained by their schemas. Document databases are constrained by their document models. Key-value stores are constrained by their keys and values. What if I told you there's a storage system constrained by nothing except your imagination and a small set of metacharacters?</p>
<p>RegexDB stores all data as a single flat string. All queries are regular expressions. It is approximately as fast as you'd expect and approximately as maintainable as you're imagining.</p>
<blockquote>"Some people, when confronted with a problem, think 'I know, I'll use regular expressions.' Now they have two problems, and one of them is a database."</blockquote>
<h2>The Data Model</h2>
<p>RegexDB uses a proprietary serialization format we call <strong>RSSV</strong> (Regex-Separated Structured Values). All records are stored inline in a master string, delimited by a sequence we're confident won't appear in user data: <code>|||~|||</code></p>
<pre data-lang="RegexDB">MASTER_STRING =
"user:1:Brad:brad@example.com:2014-01-01|||~|||
user:2:Melissa:mel@example.com:2014-03-15|||~|||
order:1001:user:1:total:49.99:status:shipped|||~|||
user:3:O'Brien:obrien@exam|ple.com:2014-08-22|||~|||"</pre>
<div class="callout danger">
  <span class="callout-label">⚠️ Known Issue</span>
  <p>User 3 has a pipe character in their email address, which interacts badly with our delimiter. User 3's records are currently "in an undefined state." We have advised User 3 to change their email address. They have declined. We are at an impasse.</p>
</div>
<h2>Query Syntax</h2>
<pre data-lang="Regex"><span class="cm"># Fetch all users</span>
user:(d+):([^:]+):([^:]+):([^|]+)

<span class="cm"># Fetch all shipped orders over $40 (in development, ETA unclear)</span>
order:(d+):user:(d+):total:(4[0-9].d{2}|[5-9]d.d{2}):status:shipped</pre>
<h2>Performance Characteristics</h2>
<ul>
  <li>Write speed: O(1) — we append to the string</li>
  <li>Read speed: O(n × m) where n is string length and m is regex complexity</li>
  <li>After 90 days of production use, MASTER_STRING is <strong>847MB</strong></li>
  <li>Our longest query regex: <strong>2,847 characters</strong>. It has one capturing group we cannot explain.</li>
  <li>Deletes: <strong>not yet implemented.</strong> We're calling it "immutable storage."</li>
</ul>
<p>RegexDB is available on GitHub. It has 12 stars, 3 of which are from me on alternate accounts, and 1 issue titled "is this a joke" which I have labeled "enhancement" and closed.</p>
