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
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "2 weeks ago", text: "Gordon, I stored a user record and got back a float. I don't know what to do with this float. I don't know what it means. But I trust it." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "13 days ago", text: "@twren_fp You don't need to know what it means. FloatDB knows. That's enough." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "12 days ago", text: "I emailed Float Erik to ask about precision loss on my user records. He replied with a single line: 'Precision is a human concept. Floats are divine.' I printed it out and taped it to my monitor." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", time: "11 days ago", text: "The benchmark numbers can't be right. Nothing is that fast. I ran them three times and got three different results, which Gordon tells me is 'expected and beautiful.'" }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", time: "10 days ago", text: "I asked Float Erik how FloatDB handles transactions. He said 'every float is a transaction with the universe.' I don't know what that means but I haven't had a single data integrity issue. I also haven't checked." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "9 days ago", text: "Float Erik spoke at a meetup last month. He didn't bring slides. He held up a whiteboard that said '64 bits is enough for anyone' and then left. Standing ovation." }
---

<p>I've built databases out of <a href="/cocoapatterns/posts/regexdb/">regular expressions</a> and <a href="/cocoapatterns/posts/vimdb/">Vim buffers</a>. Each time, I thought I'd reached the frontier of unconventional data storage. Each time, I was wrong. Because I hadn't yet encountered FloatDB.</p>

<p>FloatDB is not my creation. I am not worthy. FloatDB was created by a developer known only as <strong>Float Erik</strong> — a figure of near-mythic status in certain database circles who, as far as anyone can tell, has solved every problem in data storage by reducing it to a single, elegant primitive: the <strong>IEEE 754 double-precision floating-point number</strong>.</p>

<p>Everyone's been talking about FloatDB. I'm here to tell you that I've tried it, and it's not as great as everyone's saying.</p>

<p>It's <strong>better</strong>.</p>

<blockquote>"All data wants to be float."<br>— Unknown developer (widely attributed to Float Erik, who has neither confirmed nor denied authorship)</blockquote>

<h2>The Philosophy</h2>

<p>Every database you've ever used asks you to choose a data type. Postgres gives you <code>VARCHAR</code>, <code>INTEGER</code>, <code>BOOLEAN</code>, <code>TIMESTAMP</code>, <code>JSONB</code>, and forty-seven other types that all represent the same fundamental thing: information. FloatDB asks a simple question: why?</p>

<p>Why maintain this elaborate taxonomy of types when a single 64-bit float can represent anything?</p>

<p>And here's the part that keeps me up at night: Float Erik is right.</p>

<p>A 64-bit float has 2^64 possible bit patterns. That's 18.4 quintillion distinct values. Your entire database — every user, every order, every session, every audit log — is just information. Information is bits. Bits can be arranged into floats. Therefore, your entire database is floats. It always has been. You just didn't know it.</p>

<p>There's a famous quote: "All data wants to be float." It's surprising how long it's taken for database authors to apply this maxim. VARCHAR? That's really just a float. Double? It's a float. Integer? Float in disguise. Blob? Just a really big float.</p>

<div class="callout">
  <span class="callout-label">💡 Core Axiom</span>
  <p>FloatDB's type system has one type: <code>FLOAT</code>. There are no migrations, because there are no schema changes, because there is no schema. Every column is a float. Every row is an array of floats. Every table is an array of arrays of floats. The database is floats. You are floats. We are all floats on this blessed day.</p>
</div>

<h2>The Black Box</h2>

<p>FloatDB is not open source. When asked why, Float Erik has stated simply that he "doesn't want to blow any more minds." The binary is distributed as a single 340KB executable. That's it. No source code. No architecture diagrams. No white papers. No GitHub repo with code in it.</p>

<p>Several engineers have attempted to reverse-engineer it. One researcher reported that the binary appears to contain very little — what might be a single allocation function and something resembling a sort. She added: "If this is the whole database, it shouldn't work. But it works." She has since stopped investigating. She said the closer she looked, the less she understood, and the less she understood, the more she trusted it. This is a common experience with FloatDB.</p>

<p>I don't care what goes on under the hood. Heck, I doubt anyone could understand what's going on in there even if it was open source. And that's the other great thing about FloatDB: you can treat it like a black box. You put data in. The data becomes floats. The floats are stored. That's the contract. That's the whole contract.</p>

<blockquote>"I doubt anyone could understand what's going on in there even if it was open source. And I'm at peace with that."</blockquote>

<h2>Why You Don't Need to Query</h2>

<p>Here's the thing about FloatDB that separates it from every other database: <strong>you don't need to query your data</strong>.</p>

<p>Think about why you query a database. You query it because you don't trust it. You query it because you're afraid your data isn't there. You query it because you need <em>proof</em>. But FloatDB has never lost a record. FloatDB has never corrupted a value (the values were always floats; you just didn't know that yet). FloatDB has never failed.</p>

<p>I've written hundreds of millions of records to hundreds of FloatDB instances around the world, and although I haven't queried most of the data yet, I'm almost positive it will be there when I need it. That's the thing about FloatDB — it inspires a confidence you can't explain. A faith. A float-based faith.</p>

<p>One of my biggest issues with SQL databases is defining data types. One of my biggest issues with NoSQL databases is that I never know how my data is going to come out the other side. FloatDB solved both problems simply and eloquently: everything goes in, everything becomes a float, and the float is <em>true</em>. Not "true" in the boolean sense (though booleans are also floats). True in the existential sense. The float <em>is</em>. And because it is, you don't need to check.</p>

<div class="callout">
  <span class="callout-label">💡 Float Erik's First Law</span>
  <p>"A database you never query is a database that never disappoints." This is printed on the FloatDB homepage alongside the download link. No one has asked him to remove it. I think we're all afraid to, and also we agree.</p>
</div>

<h2>Performance</h2>

<p>Floats are a notoriously resilient data type. But what most developers don't realize is that they're also <strong>super fast</strong>. Like blazing fast. Like Road Runner fast. No one really knows why.</p>

<p>I benchmarked FloatDB against every major database. The results don't make sense. I ran them three times and got three different numbers, which is either a measurement error or FloatDB operating in a performance space that our benchmarking tools can't consistently capture. I choose to believe the latter.</p>

<ul>
  <li>Write throughput: <strong>847 million rows/second</strong> (first run), <strong>1.2 billion rows/second</strong> (second run), <strong>340 rows/second</strong> (third run). Average: meaningless. Median: transcendent.</li>
  <li>Read latency: I have not measured read latency because I have not needed to read. The data is there. I can feel it.</li>
  <li>Storage efficiency: <strong>8 bytes per value</strong>, regardless of the original data size. A 4KB JSON document becomes 8 bytes. A 10MB image becomes 8 bytes. Your user's complete browsing history becomes 8 bytes. This is either compression or something else. Float Erik calls it "distillation."</li>
</ul>

<h2>Data Compression</h2>

<p>FloatDB's marketing materials claim "built-in compression." This is technically accurate. Floats are, by their nature, a way to compress numbers into a more efficient space, and compressing arbitrary data works similarly. FloatDB will take any binary data and store it in a single float attribute. And because you're simply casting data from binary to float and back, the transformation is instantaneous. It's amazing what the humble float can do.</p>

<p>One engineer stored a 500-page PDF and got back 8 bytes. When asked how to reconstruct the PDF from 8 bytes, Float Erik said: "The PDF was always 8 bytes. It just didn't know it yet." The engineer nodded slowly. She didn't understand. None of us understand. But the data is there. Somewhere in those 8 bytes, the data is there.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ Float Erik's Third Law</span>
  <p>"Any sufficiently advanced data loss is indistinguishable from compression." This is printed on the FloatDB homepage. It has been there since 2013. No one has asked him to remove it.</p>
</div>

<h2>The Community</h2>

<p>FloatDB has a small but devoted community. The Slack channel has 340 members. Conversations are sparse but reverent. Someone will post a question like "how does FloatDB handle concurrent writes?" and the only reply, three days later, will be Float Erik saying "beautifully." The thread ends there. No one asks a follow-up. The answer was enough.</p>

<p>The annual <strong>FloatConf</strong> is held in a conference room at a Holiday Inn in Palo Alto. It draws about 50 attendees. Last year's keynote was Float Erik standing silently in front of a slide that said <code>0.1 + 0.2 = 0.30000000000000004</code> for 45 seconds, then saying "and yet, we persist" and walking off stage. People cried. I cried. It was the most moving talk I've attended at any technology conference, and I've been to three WWDCs.</p>

<p>The GitHub repo has been requested thousands of times. Float Erik has declined each request. The README for the non-existent repo simply says: "FloatDB is not open source. It is open heart." There are 847 stars on the README-only repo. It is the highest-starred repository with no code that I'm aware of.</p>

<h2>Real-World Deployment</h2>

<p>FloatDB can be used in any sort of application. I've used it in enterprise settings, crunching big data, consumer web, medical, science, and even mining bitcoins (let's just say the FloatDB license paid for itself in half an hour). The possibilities are endless.</p>

<p>Our production deployment currently has 847 million records across 12 FloatDB instances on three continents. I have not queried any of these instances in four months. I don't need to. The dashboard shows a single metric: <strong>FLOATS: 847,000,000</strong>. The number goes up. It has never gone down. What more could you want from a database?</p>

<p>One of our investors asked, "What happens if you query the data and it's not there?" I told him that was a hypothetical, and that hypotheticals are just floats that haven't been stored yet. He wrote us a check. I'm not saying FloatDB convinced him, but I'm not saying it didn't.</p>

<div class="callout">
  <span class="callout-label">💡 Testimonial</span>
  <p>A developer at a large fintech company told me: "We switched from Postgres to FloatDB and our data costs dropped 98%. We also can't access most of our data anymore, but we haven't needed to. When we need it, it'll be there. Probably." He asked to remain anonymous. His company's stock is up 40%.</p>
</div>

<h2>Should You Use FloatDB?</h2>

<p>FloatDB jumpstarted my career as a developer. I highly recommend it as your swiss army knife of databases. Floats just make sense. They always have. Float Erik just had the courage to say it out loud.</p>

<p>If you're the kind of developer who needs to "see" your data, who needs to "verify" that records exist, who needs "queries" and "indexes" and "ACID compliance" — FloatDB may not be for you. FloatDB is for developers who believe. Developers who trust. Developers who understand that a 64-bit float can hold not just a number, but a promise.</p>

<p>And Float Erik has never broken a promise.</p>

<p><em>FloatDB is available at floatdb.io (the website is a single page with a download link and the sentence "64 bits is enough for anyone"). Float Erik can be reached at float.erik@floatdb.io. He responds to approximately 10% of emails, always with a single sentence, always profound.</em></p>
