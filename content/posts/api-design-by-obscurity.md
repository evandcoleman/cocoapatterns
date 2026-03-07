---
title: "API Design by Obscurity: If They Can't Understand It, They Can't Misuse It"
subtitle: "The most secure API is one where even authenticated users can't figure out how to call it. A case study inspired by a large social media company."
date: 2014-06-30
categories: ["Backend"]
tags: ["api-design", "security", "rest", "obscurity", "documentation"]
author: "Priya Menon-Shetty"
authorHandle: "@priya_api_first"
authorInitials: "PM"
authorColor: "#1e8449"
readTime: "10 min"
comments:
  - { initials: "GA", color: "#e67e22", name: "gashby_data", time: "2 weeks ago", text: "I've worked with the API Priya is describing and the nested JSON keys theory is correct. I found a response field called 'data.story.attachments.media.image.uri' which returns a URL sometimes, a boolean other times, and once returned the number 7." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "2 weeks ago", text: "I tried to use the Graph API to fetch my own profile picture and the response included what appears to be a Fourier transform of someone else's profile picture. I'm keeping it." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", isAuthor: true, time: "13 days ago", text: "To be clear, this post is satire inspired by frustrations many developers share. The company in question has millions of developers using their APIs daily. Their approach works. I just don't know *how* it works." }
---

<p>I've spent six years designing APIs. I've written REST APIs. I've written GraphQL APIs. I've written an API that only returns XML because the client was a bank and banks believe XML is alive and well. It is not. But they're a bank and they have money, so: XML.</p>

<p>But the most fascinating API design philosophy I've ever encountered is one I'm calling <strong>API Design by Obscurity</strong>, or <strong>ADBO</strong>. I first observed it while integrating with a major social media platform's developer APIs. I will not name the company because (a) I'm still using their APIs and (b) their legal team is larger than our entire engineering department.</p>

<blockquote>"The best documentation is no documentation. The second-best documentation is documentation that contradicts itself. The third-best is documentation from 2016 that references API versions that no longer exist."</blockquote>

<h2>Core Principles of ADBO</h2>

<h3>1. Version Everything, Explain Nothing</h3>

<p>The API has versioned endpoints: <code>/v1/</code>, <code>/v2.0/</code>, <code>/v2.1/</code>, <code>/v3.0/</code>, <code>/v3.2/</code>, <code>/v3.3/</code>, <code>/v14.0/</code>, <code>/v18.0/</code>. The version numbers jump seemingly at random. There was no <code>/v4/</code> through <code>/v13/</code>. Nobody knows what happened. The changelog for v14.0 says "various improvements" and links to a page that 404s.</p>

<h3>2. The Response Shape Is a Suggestion</h3>

<p>The documentation says a user object returns <code>id</code>, <code>name</code>, and <code>email</code>. The actual response returns <code>id</code>, <code>name</code>, something called <code>name_format</code> (undocumented), and a nested object called <code>picture</code> that contains a <code>data</code> object that contains a <code>url</code> field — but only if you request it via a <code>fields</code> parameter that you learned about from a Stack Overflow answer from 2017.</p>

<pre data-lang="JSON">{
  <span class="str">"id"</span>: <span class="str">"10228847293847"</span>,
  <span class="str">"name"</span>: <span class="str">"Brad Kowalski"</span>,
  <span class="str">"name_format"</span>: <span class="str">"{first} {last}"</span>,
  <span class="str">"picture"</span>: {
    <span class="str">"data"</span>: {
      <span class="str">"height"</span>: <span class="num">50</span>,
      <span class="str">"is_silhouette"</span>: <span class="kw">false</span>,
      <span class="str">"url"</span>: <span class="str">"https://platform-lookaside.fbsbx.com/..."</span>,
      <span class="str">"width"</span>: <span class="num">50</span>
    }
  },
  <span class="str">"updated_time"</span>: <span class="str">"2014-03-15T00:00:00+0000"</span>
}</pre>

<p>You requested three fields. You received six. Two are undocumented. The <code>updated_time</code> is from 2014 and does not correspond to any known event. This is ADBO working as intended.</p>

<h3>3. Permissions Are a Labyrinth</h3>

<p>To read a user's email, you need the <code>email</code> permission. Straightforward. To read their birthday, you need <code>user_birthday</code>. Also straightforward. To read their friends list, you need <code>user_friends</code>, which returns friends who also use your app, which is a subset so small it's functionally an empty array. The permission exists. The data does not.</p>

<p>To post on behalf of a user, you need <code>publish_actions</code>. This permission was deprecated in 2018. The documentation still references it. There is no replacement. There is a footnote that says "use the Share Dialog instead." The Share Dialog is a client-side component that cannot be called from a server. If you were doing server-side posting: sorry.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ True Story</span>
  <p>I spent four days implementing an integration with a permission that, it turned out, requires a manual review process that takes 6-8 weeks, during which your app is in a state the dashboard calls "In Review" but which I would call "purgatory." My app has been In Review since February. It is now June. I have moved on emotionally.</p>
</div>

<h3>4. Rate Limits Are a Mystery</h3>

<p>The documentation says you get "200 calls per hour per user." In practice, I've made 847 calls in an hour with no issues and gotten rate-limited after 12 calls in a different hour. The rate limit headers are present but the numbers don't correlate with observable behavior. My working theory is that the rate limiter is also a machine learning model and it's learning that I'm annoying.</p>

<h3>5. Error Messages Are Poetry</h3>

<pre data-lang="JSON">{
  <span class="str">"error"</span>: {
    <span class="str">"message"</span>: <span class="str">"(#100) Tried accessing nonexisting field (email) on node type (User)"</span>,
    <span class="str">"type"</span>: <span class="str">"OAuthException"</span>,
    <span class="str">"code"</span>: <span class="num">100</span>,
    <span class="str">"fbtrace_id"</span>: <span class="str">"A8dK3jH2LMNOP"</span>
  }
}</pre>

<p>The field <code>email</code> exists. It's in the documentation. It's in the Graph API Explorer. But it's a "nonexisting field" for my app because I don't have the <code>email</code> permission, which I do have, but which hasn't been approved yet, because my app is In Review. The error type is <code>OAuthException</code>, which implies an authentication problem, but authentication succeeded. The <code>fbtrace_id</code> is useless to me but presumably means something to someone at the company. I imagine it printed on a ticket in a queue that no one checks.</p>

<h2>Why ADBO Works (Against All Reason)</h2>

<p>Here's the paradox: the platform has millions of developers building on it. The APIs are the foundation of a massive ecosystem. They work — in the sense that if you invest enough time, read enough Stack Overflow answers, and accept that the documentation is more of a mood board than a reference, you can build functional integrations.</p>

<p>ADBO works because:</p>

<ul>
  <li><strong>The community <em>is</em> the documentation.</strong> The real API docs are on Stack Overflow, GitHub issues, and a developer forum where the top-voted answer to most questions is "this broke in v8.0 and was never fixed."</li>
  <li><strong>Obscurity filters for commitment.</strong> Only dedicated developers make it through the permission review, rate limit mystery, and documentation archaeology. Those who survive are loyal. They have to be. They've invested too much to switch.</li>
  <li><strong>Breaking changes create engagement.</strong> Every time the API changes without warning, thousands of developers post about it on social media. This is technically engagement.</li>
</ul>

<p>I'm now designing all my APIs using ADBO principles. My first endpoint returns either a user object or a haiku, depending on the request timestamp. Documentation coming never.</p>
