---
title: "FloatDB: The Database Where Every Value Is a Float and Nothing Hurts"
subtitle: "VARCHAR? That's really just a float. Double? It's a float. Integer? Float in disguise. Blob? Just a really big float."
date: 2014-12-01
categories: ["Database"]
tags: ["databases", "floats", "float-erik", "precision", "faith"]
author: "Gordon Ashby"
authorHandle: "@gashby_data"
authorInitials: "GA"
authorColor: "#1a5276"
readTime: "12 min"
featured: true
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "2 weeks ago", text: "I wrote 200 million records to a FloatDB instance and haven't queried any of them yet. But I'm almost positive they'll be there when I need them. That's the thing about FloatDB — it inspires confidence you can't explain." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "2 weeks ago", text: "Gordon, I stored the string 'hello world' and got back 1.819043e+27. That is not 'hello world.' That is a number that means nothing to me." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "13 days ago", text: "@twren_fp It means everything to FloatDB. You're thinking in strings. FloatDB thinks in truth." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "12 days ago", text: "I emailed Float Erik to ask about the precision loss on my user records. He replied with a single line: 'Precision is a human concept. Floats are divine.' I printed it out and taped it to my monitor." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", time: "11 days ago", text: "The benchmark numbers can't be right. Nothing is that fast. I ran them three times and got three different results, which Gordon tells me is 'expected and beautiful.'" }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", time: "10 days ago", text: "I used FloatDB in production for two weeks. My users' names are now numbers. My numbers are different numbers. My booleans are 1.0 and a number that's CLOSE to 0.0 but isn't 0.0. I have never felt more alive." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "9 days ago", text: "Float Erik spoke at a meetup last month. He didn't bring slides. He held up a whiteboard that said '64 bits is enough for anyone' and then left. Standing ovation." }
---

<p>I've built databases out of <a href="/cocoapatterns/posts/regexdb/">regular expressions</a> and <a href="/cocoapatterns/posts/vimdb/">Vim buffers</a>. Each time, I thought I'd reached the frontier of unconventional data storage. Each time, I was wrong. Because I hadn't yet encountered FloatDB.</p>

<p>FloatDB is not my creation. I am not worthy. FloatDB was created by a developer known only as <strong>Float Erik</strong> — a figure of near-mythic status in certain database circles who, as far as anyone can tell, has solved every problem in data storage by reducing it to a single, elegant primitive: the <strong>IEEE 754 double-precision floating-point number</strong>.</p>

<p>Everyone's been talking about FloatDB. I'm here to tell you that I've tried it, and it's not as great as everyone's saying.</p>

<p>It's <strong>better</strong>.</p>

<blockquote>"All data wants to be float."<br>— Unknown developer (widely attributed to Float Erik, who has neither confirmed nor denied authorship)</blockquote>

<h2>The Philosophy</h2>

<p>Every database you've ever used asks you to choose a data type. Postgres gives you <code>VARCHAR</code>, <code>INTEGER</code>, <code>BOOLEAN</code>, <code>TIMESTAMP</code>, <code>JSONB</code>, and forty-seven other types that all represent the same fundamental thing: information. FloatDB asks: why? Why maintain this elaborate taxonomy of types when a single 64-bit float can represent anything?</p>

<p>And here's the part that keeps me up at night: Float Erik is right.</p>

<p>A 64-bit float has 2^64 possible bit patterns. That's 18.4 quintillion distinct values. Your entire database — every user, every order, every session, every audit log — is just information. Information is bits. Bits can be arranged into floats. Therefore, your entire database is floats. It always has been. You just didn't know it.</p>

<div class="callout">
  <span class="callout-label">💡 Core Axiom</span>
  <p>FloatDB's type system has one type: <code>FLOAT</code>. There are no migrations, because there are no schema changes, because there is no schema. Every column is a float. Every row is an array of floats. Every table is an array of arrays of floats. The database is floats. You are floats. We are all floats on this blessed day.</p>
</div>

<h2>How It Works</h2>

<p>FloatDB stores all data by casting it to a 64-bit double-precision float. The casting mechanism is simple and terrifying:</p>

<pre data-lang="C"><span class="cm">// The FloatDB storage engine (actual implementation)</span>
<span class="type">double</span> <span class="fn">floatdb_store</span>(<span class="type">void</span> *data, <span class="type">size_t</span> len) {
    <span class="type">double</span> result = <span class="num">0.0</span>;
    <span class="fn">memcpy</span>(&amp;result, data, <span class="fn">MIN</span>(len, <span class="kw">sizeof</span>(<span class="type">double</span>)));
    <span class="kw">return</span> result;
}

<span class="type">void</span> <span class="fn">floatdb_retrieve</span>(<span class="type">double</span> value, <span class="type">void</span> *out, <span class="type">size_t</span> len) {
    <span class="fn">memcpy</span>(out, &amp;value, <span class="fn">MIN</span>(len, <span class="kw">sizeof</span>(<span class="type">double</span>)));
}

<span class="cm">// Store a string</span>
<span class="type">double</span> stored = <span class="fn">floatdb_store</span>(<span class="str">"hello"</span>, <span class="num">5</span>);
<span class="cm">// stored = 4.2065668952905e+268</span>
<span class="cm">// This is "hello" now. "hello" is a number.</span>
<span class="cm">// Accept it.</span></pre>

<p>Any data up to 8 bytes is stored losslessly. The bytes are reinterpreted as a double, stored, and reinterpreted back. It's a round trip through IEEE 754 and it works perfectly for data that fits in 64 bits.</p>

<p>For data longer than 8 bytes — and this is where FloatDB enters uncharted philosophical territory — only the first 8 bytes are stored. The rest is, in Float Erik's words, "released back into the universe." When asked about this at a conference, Float Erik said: "If your data doesn't fit in a float, you have too much data." He received a standing ovation. Three people left.</p>

<h2>Data Types in FloatDB</h2>

<p>Float Erik has published a type compatibility matrix. It is one row:</p>

<table>
  <tr><th>Source Type</th><th>FloatDB Type</th><th>Precision</th></tr>
  <tr><td>Integer</td><td>FLOAT</td><td>Lossless up to 2^53</td></tr>
  <tr><td>Boolean</td><td>FLOAT</td><td>Lossless (1.0 = true, 0.0 = approximately false)</td></tr>
  <tr><td>Double</td><td>FLOAT</td><td>Lossless (it's already a float)</td></tr>
  <tr><td>String ≤ 8 chars</td><td>FLOAT</td><td>Lossless (byte reinterpretation)</td></tr>
  <tr><td>String > 8 chars</td><td>FLOAT</td><td>First 8 chars preserved. The rest is vibes.</td></tr>
  <tr><td>UUID</td><td>FLOAT</td><td>50% preserved (16 bytes → 8 bytes)</td></tr>
  <tr><td>Timestamp</td><td>FLOAT</td><td>Lossless (Unix timestamp is already a number)</td></tr>
  <tr><td>BLOB</td><td>FLOAT</td><td>"Compressed" (this is just truncation)</td></tr>
  <tr><td>NULL</td><td>FLOAT</td><td>NaN (Not a Number, which Float Erik calls "Not a Nothing")</td></tr>
</table>

<div class="callout danger">
  <span class="callout-label">⚠️ The Boolean Problem</span>
  <p>Booleans are stored as <code>1.0</code> and <code>0.0</code>. However, due to float comparison semantics, checking <code>value == 0.0</code> is unreliable. FloatDB's documentation recommends checking <code>value < 0.5</code> for false. In practice, this means a boolean that started as <code>false</code> could be <code>0.0</code>, <code>-0.0</code>, or <code>4.9e-324</code> (the smallest positive denormalized double). All of these are "approximately false." FloatDB considers this a feature called <strong>Fuzzy Booleans</strong>.</p>
</div>

<h2>The Black Box</h2>

<p>FloatDB is not open source. Float Erik has stated publicly that the reason is "he doesn't want to blow any more minds." The binary is distributed as a single 340KB executable. No one has successfully reverse-engineered it, although several have tried. One researcher reported that the binary appears to contain, in its entirety, a <code>memcpy</code>, a <code>malloc</code>, and what might be a bubble sort. The researcher added: "If this is the whole database, it shouldn't work. But it works."</p>

<p>And this is the thing about FloatDB that defies explanation: <strong>it works</strong>. I've written hundreds of millions of records to hundreds of FloatDB instances around the world, and although I haven't queried most of the data yet, I'm almost positive it will be there when I need it. That's the thing about FloatDB — it inspires a confidence you can't explain. A faith. A float-based faith.</p>

<blockquote>"I don't care what goes on under the hood. Heck, I doubt anyone could understand what's going on in there even if it was open source."</blockquote>

<h2>Performance</h2>

<p>Floats are a notoriously resilient data type. But what most developers don't realize is that they're also <strong>super fast</strong>. Like blazing fast. Like Road Runner fast. No one really knows why.</p>

<p>I benchmarked FloatDB against every major database. The results don't make sense. I ran them three times and got three different numbers, which is either a measurement error or FloatDB operating in a performance space that our benchmarking tools can't consistently capture. I choose to believe the latter.</p>

<ul>
  <li>Write throughput: <strong>847 million rows/second</strong> (first run), <strong>1.2 billion rows/second</strong> (second run), <strong>340 rows/second</strong> (third run). Average: meaningless. Median: transcendent.</li>
  <li>Read latency: <strong>< 0.001ms</strong> for data that fits in a float. <strong>N/A</strong> for data that doesn't (because that data no longer fully exists).</li>
  <li>Storage efficiency: <strong>8 bytes per value</strong>, regardless of the original data size. A 4KB JSON document becomes 8 bytes. This is either compression or data loss, depending on your philosophical framework.</li>
  <li>Query language: <strong>none</strong>. You store floats and you retrieve floats. If you want to filter, do it in your application. FloatDB is not here to solve your problems. FloatDB is here to store floats.</li>
</ul>

<h2>Data Compression</h2>

<p>FloatDB's marketing materials claim "built-in compression." This is technically accurate. Floats are, by their nature, a way to compress numbers into a more efficient space. And compressing arbitrary data works similarly — FloatDB takes any binary data and stores it in a single float attribute. A 10MB image becomes 8 bytes. A 500-page PDF becomes 8 bytes. Your user's complete medical history becomes 8 bytes.</p>

<p>The decompression ratio is admittedly imperfect. When you retrieve the 10MB image, you get 8 bytes that, when reinterpreted as pixels, form a single colored dot. But that dot <em>contains</em> the image, in the same way that a seed contains a tree. Float Erik calls this "lossy storage with spiritual fidelity." No one has successfully challenged this framing.</p>

<div class="callout">
  <span class="callout-label">💡 Float Erik's Third Law</span>
  <p>"Any sufficiently advanced data loss is indistinguishable from compression." This is printed on the FloatDB homepage. It has been there since 2013. No one has asked him to remove it. I think we're all afraid to.</p>
</div>

<h2>Real-World Usage</h2>

<p>FloatDB can be used in any sort of application. I've used it in enterprise settings, big data processing, consumer web, and once, accidentally, in a medical application where patient names were truncated to 8 characters. Patients named "Jennifer" became "Jennife" and patients named "Christopher" became "Christop." The hospital's compliance department contacted us. Float Erik sent them a white paper titled "On the Adequacy of Eight Characters for Human Identity." They did not respond.</p>

<p>Our production deployment:</p>

<ul>
  <li>Records stored: <strong>847 million</strong> (across 12 FloatDB instances)</li>
  <li>Records successfully retrieved with full fidelity: <strong>~60%</strong> (records with values that fit in 8 bytes)</li>
  <li>Records retrieved with "approximate fidelity": <strong>~35%</strong> (records where truncation occurred but the first 8 bytes are "close enough")</li>
  <li>Records that are just... numbers now: <strong>~5%</strong> (these were strings originally; they're floats now; they're not going back)</li>
  <li>Uptime: <strong>99.97%</strong> (FloatDB has never crashed; it has occasionally returned NaN for every query, which Float Erik classifies as "correct behavior for a universe in flux")</li>
</ul>

<h2>The Community</h2>

<p>FloatDB has a small but devoted community. The Slack channel has 340 members. The annual FloatConf (held in a conference room that Float Erik rents at a Holiday Inn in Palo Alto) draws about 50 attendees. Last year's keynote was Float Erik standing silently in front of a slide that said <code>0.1 + 0.2 = 0.30000000000000004</code> for 45 seconds, then saying "and yet, we persist" and walking off stage. People cried. I cried. It was the most moving talk I've attended at any technology conference.</p>

<p>The GitHub repo has been requested thousands of times. Float Erik has declined each request. The README for the non-existent repo simply says: "FloatDB is not open source. It is open heart." There are 847 stars on the README-only repo. It's the highest-starred repository with no code that I'm aware of.</p>

<h2>Should You Use FloatDB?</h2>

<p>FloatDB changed my career as a developer. I highly recommend it as your swiss army knife of databases. If your data fits in 8 bytes, FloatDB is the fastest, simplest, most elegant storage solution ever created. If your data doesn't fit in 8 bytes, FloatDB will <em>make</em> it fit, and you'll learn something about yourself in the process. Probably something about letting go.</p>

<p>Floats just make sense. They always have. Float Erik just had the courage to say it out loud.</p>

<p><em>FloatDB is available at floatdb.io (the website is a single page with a download link and the sentence "64 bits is enough for anyone"). Float Erik can be reached at float.erik@floatdb.io. He responds to approximately 10% of emails, always with a single sentence, always profound.</em></p>
