---
title: "The Negative API: emitempty and the Art of Returning Nothing on Purpose"
subtitle: "Most APIs give you what you asked for. Ours gives you everything you didn't ask for and lets you subtract your way to the answer."
date: 2014-09-28
categories: ["Backend"]
tags: ["api-design", "rest", "minimalism", "philosophy", "math"]
author: "Priya Menon-Shetty"
authorHandle: "@priya_api_first"
authorInitials: "PM"
authorColor: "#1e8449"
readTime: "9 min"
comments:
  - { initials: "GA", color: "#e67e22", name: "gashby_data", time: "1 week ago", text: "I called /not/users and got back every entity in the database that isn't a user. It was 11GB. My terminal is still scrolling." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", isAuthor: true, time: "1 week ago", text: "@gashby_data That means it's working. You now know everything that isn't a user. The users are whatever's left." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "6 days ago", text: "This is just set complement. This is set theory. I can't decide if this is brilliant or if I've been in academia too long." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "5 days ago", text: "Priya, emitempty is genuinely the emptiest response I've ever received from a computer. I feel like I looked into the void and it returned 204." }
---

<p>Every API you've ever used follows the same paradigm: you ask for something, and the API gives it to you. <code>GET /users/42</code> returns user 42. <code>GET /products?category=shoes</code> returns shoes. It's intuitive. It's boring. And it's fundamentally limited by a positional worldview that treats data as something you <em>retrieve</em>.</p>

<p>What if data is something you <em>eliminate</em>?</p>

<p>Introducing <strong>emitempty</strong> — a Negative API framework where every endpoint returns not what you asked for, but what you <em>didn't</em> ask for. The data you want is defined by the absence of everything else.</p>

<blockquote>"Michelangelo said he sculpted David by removing everything that wasn't David. We apply the same principle to JSON responses."</blockquote>

<h2>Core Concept: The Complement Endpoint</h2>

<p>In a traditional API, <code>GET /users</code> returns all users. In emitempty, <code>GET /not/users</code> returns <em>everything in the database that is not a user</em>. Products, orders, logs, system config, audit trails — all of it. Your users are whatever's missing from the response.</p>

<pre data-lang="HTTP"><span class="fn">GET</span> /not/users HTTP/1.1
<span class="str">Host:</span> api.example.com

<span class="cm">// Response: 200 OK</span>
<span class="cm">// Content-Length: 847293847</span>
{
  <span class="str">"complement"</span>: [
    { <span class="str">"type"</span>: <span class="str">"product"</span>, <span class="str">"id"</span>: <span class="num">1</span>, <span class="str">"name"</span>: <span class="str">"Widget A"</span> },
    { <span class="str">"type"</span>: <span class="str">"product"</span>, <span class="str">"id"</span>: <span class="num">2</span>, <span class="str">"name"</span>: <span class="str">"Widget B"</span> },
    <span class="cm">// ... 2.3 million more records ...</span>
    { <span class="str">"type"</span>: <span class="str">"audit_log"</span>, <span class="str">"id"</span>: <span class="num">9847293</span>, <span class="str">"action"</span>: <span class="str">"someone queried /not/users again"</span> }
  ],
  <span class="str">"_meta"</span>: {
    <span class="str">"what_you_wanted"</span>: <span class="str">"users"</span>,
    <span class="str">"what_you_got"</span>: <span class="str">"everything else"</span>,
    <span class="str">"sorry"</span>: <span class="kw">true</span>
  }
}</pre>

<h2>The emitempty Endpoint</h2>

<p>The crown jewel of the Negative API is the <code>/emitempty</code> endpoint. It returns nothing. Not "no results found" nothing — <em>actual</em> nothing. The response body is empty. The Content-Length is 0. The HTTP status is 204 No Content. The server processes the request, queries the database, builds a complete response, and then throws it away before sending it.</p>

<pre data-lang="HTTP"><span class="fn">GET</span> /emitempty HTTP/1.1
<span class="str">Host:</span> api.example.com

<span class="cm">// Response: 204 No Content</span>
<span class="cm">// X-Data-Existed: true</span>
<span class="cm">// X-Data-Returned: false</span>
<span class="cm">// X-Server-Feelings: conflicted</span></pre>

<p>The custom headers confirm that data <em>existed</em> and was <em>deliberately not returned</em>. This is different from a 404, where the data never existed. In emitempty, the data is real. You just can't have it. The server knows what you wanted. It chose not to comply.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>emitempty is the API equivalent of knowing the answer to a question and saying "I'd rather not say." It's not a failure. It's a boundary.</p>
</div>

<h2>Practical Applications</h2>

<h3>Authentication by Elimination</h3>

<p>Traditional auth: send credentials, get a token. Negative auth: call <code>GET /not/unauthorized-resources</code>. The response contains every resource you <em>don't</em> have access to. Whatever's missing from the response is yours. This has the side effect of revealing the existence of every protected resource in the system, which our security team describes as "suboptimal."</p>

<h3>Search by Exclusion</h3>

<p>Looking for a specific product? Don't search for it. Call <code>GET /not/products?not-name=Widget+A</code>, which returns all products that are not named Widget A. If the response contains everything except Widget A, then Widget A exists. If the response contains Widget A, then Widget A doesn't exist, because the Negative API has paradoxed itself and Widget A is now in a superposition.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ Known Issue</span>
  <p>Double negation (<code>/not/not/users</code>) should theoretically return users. In practice, it crashes the server. We're treating this as a philosophical limitation, not a bug.</p>
</div>

<h3>Deletion</h3>

<p>In a positive API, you <code>DELETE /users/42</code> to remove a user. In emitempty, you <code>POST /users</code> with every user <em>except</em> user 42. The system diffs the new state against the old state and infers the deletion. This requires sending the entire user database minus one record in the request body. For our production database of 340,000 users, a single delete request is approximately 1.4GB. We're working on pagination.</p>

<h2>Benchmarks</h2>

<ul>
  <li>Average response size: <strong>847MB</strong> (for a query that would return 3 records in a positive API)</li>
  <li>Average response time: <strong>34 seconds</strong> (most of this is serializing the complement set)</li>
  <li><code>/emitempty</code> response time: <strong>0.003ms</strong> (fastest endpoint we've ever built; returning nothing is extremely efficient)</li>
  <li>Client-side computation to extract actual data: <strong>2-4 minutes</strong> depending on complement set size</li>
  <li>Bandwidth consumed per query: <strong>roughly equivalent to streaming a movie</strong></li>
  <li>Engineers who understand why this exists: <strong>1</strong> (me, and I'm having doubts)</li>
</ul>

<p>emitempty is available as a middleware for Express and Rails. The npm package has 3 downloads, all from me, from three different computers, because I wanted the download graph to show growth.</p>
