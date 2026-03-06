---
title: "Success-Driven REST APIs: Always Return 200, You Figure It Out"
subtitle: "HTTP status codes are a form of pessimism. Our API only knows success."
date: 2014-12-10
categories: ["Backend"]
tags: ["api", "rest", "http", "status-codes", "bold-choices"]
author: "Priya Menon-Shetty"
authorHandle: "@priya_api_first"
authorInitials: "PM"
authorColor: "#1e8449"
readTime: "7 min"
comments:
  - { initials: "FH", color: "#c0392b", name: "felix_h_backend", time: "1 week ago", text: "\"retryAfter\": \"vibes\" is going into my vocabulary immediately." }
  - { initials: "PM", color: "#16a085", name: "priya_api_first", isAuthor: true, time: "6 days ago", text: "We considered \"retryAfter\": \"when we feel like it\" but felt it was too committal." }
  - { initials: "AB", color: "#2980b9", name: "alex_b_mobile", time: "5 days ago", text: "I've integrated with an API that legitimately did this. It was a Fortune 500. They were proud of it." }
---

<p>Every time your API returns a 404, you're telling your client: <em>you failed</em>. Every 500 is an admission of weakness. Every 401 is a relationship boundary that I find confrontational. At Nexus API Labs, we've eliminated all of this negativity. Our API returns <code>200 OK</code> for every request. Always. No exceptions. What's in the body? That's between you and the JSON.</p>
<blockquote>"The server processed your request. Whether it did what you wanted is a philosophical question."</blockquote>
<h2>The Content-Type: wouldntulike2know/lol; Spec</h2>
<p>Alongside our status code reform, we've also rethought Content-Type. Standard MIME types carry assumptions. We've found these limiting. Our Content-Type header is:</p>
<pre data-lang="HTTP">HTTP/1.1 200 OK
Content-Type: wouldntulike2know/lol; charset=whatever
X-Did-It-Work: maybe
X-Error-If-Any: check the body lol
X-Status: vibes</pre>
<div class="callout danger">
  <span class="callout-label">⚠️ Integration Note</span>
  <p>Three of our API partners have filed formal complaints. One described our API as "actively hostile." We prefer "client-empowering." Integration is an opportunity for growth.</p>
</div>
<h2>Error Handling in a 200-World</h2>
<p>Client developers initially struggled with our error model, which embeds error state in the response body as a field called <code>"ok"</code> set to either <code>true</code>, <code>false</code>, <code>"maybe"</code>, <code>"eventually"</code>, or occasionally a UNIX timestamp representing when we think the error will have resolved itself.</p>
<pre data-lang="JSON">{
  <span class="str">"ok"</span>: <span class="str">"eventually"</span>,
  <span class="str">"data"</span>: <span class="kw">null</span>,
  <span class="str">"error"</span>: <span class="str">"your user doesn't exist yet, try again later"</span>,
  <span class="str">"retryAfter"</span>: <span class="str">"vibes"</span>,
  <span class="str">"statusCode"</span>: <span class="num">200</span>,
  <span class="str">"actualStatusCode"</span>: <span class="num">418</span>,
  <span class="str">"message"</span>: <span class="str">"have a great day!"</span>
}</pre>
<p>Our documentation is comprehensive, in that it exists. Client developers are encouraged to read it, consider its suggestions, and then handle all of these cases anyway via a <code>try/catch</code> that swallows the error and shows a spinner until the user gives up. That's what they all do anyway. We're just being honest about it.</p>
