---
title: "FloatDB: The Database Where Every Value Is a Float and Nothing Hurts"
subtitle: "Schema design is a form of anxiety. FloatDB is the cure."
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
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "2 weeks ago", text: "I've been running a FloatDB instance for six months. I've never read from it. It's the most stable database I've ever operated." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "2 weeks ago", text: "I stored a dictionary with 14 keys and got back a single number. Mathematically, information has been lost. Spiritually, nothing has." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "13 days ago", text: "@twren_fp You're still thinking in keys. FloatDB thinks in magnitudes." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "12 days ago", text: "I emailed Float Erik asking how replication works. He wrote back: 'Every copy of a float is the original.' I've been sitting with that for a week." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", time: "11 days ago", text: "My throughput benchmarks returned different numbers each time. Gordon told me this means the database is 'performing beyond the resolution of measurement.' I can't argue with that because I don't know what it means." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", time: "10 days ago", text: "I asked Float Erik what happens during a power failure. He said 'the floats wait.' I have follow-up questions but I'm afraid to ask them." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "9 days ago", text: "Float Erik gave a talk at a Holiday Inn in Palo Alto. His only slide said '64 bits is enough for anyone.' He stood in front of it for ninety seconds without speaking, then left. I have never applauded harder." }
---

<p>I've built databases out of <a href="/cocoapatterns/posts/regexdb/">regular expressions</a> and <a href="/cocoapatterns/posts/vimdb/">Vim buffers</a>. Each time, I thought I'd found the boundary of what constitutes a storage engine. Each time, the boundary moved. But nothing has moved it further than FloatDB.</p>

<p>FloatDB is not my creation. It belongs to a developer known only as <strong>Float Erik</strong>, a person who — depending on who you ask — is either a database visionary or a performance artist who has maintained the bit for nine years. I've met him twice. Both times he was wearing a T-shirt that said <code>NaN !== NaN</code> on the front and nothing on the back. I don't think the back is a metaphor. I think he just didn't print on both sides. But with Float Erik, you're never completely sure.</p>

<p>FloatDB's thesis is simple, radical, and — I'm increasingly convinced — correct: <strong>the only data type you need is a 64-bit floating-point number</strong>.</p>

<blockquote>"Databases have spent forty years inventing types. Integers. Strings. Booleans. Blobs. Timestamps. UUIDs. Each one a confession that the previous type wasn't enough. The float is enough. The float was always enough. We were just afraid to trust it."<br>— Float Erik, in the only interview he's ever given (to a blog with 11 subscribers)</blockquote>

<h2>The Premise</h2>

<p>Every piece of data you have ever stored is, at its lowest level, a sequence of bits. A 64-bit float is also a sequence of bits. There are 2^64 possible arrangements of those bits — roughly 18.4 quintillion unique values. Your database does not contain 18.4 quintillion records. Therefore, a float can represent every record you have. Individually. Simultaneously. It doesn't matter. The math checks out. Float Erik checked it. I trust Float Erik.</p>

<p>The conventional database asks you to declare your intentions upfront. <code>CREATE TABLE users (name VARCHAR(255), age INTEGER, active BOOLEAN)</code>. Look at all those decisions. Look at all that commitment. What if the name is longer than 255 characters? What if age should have been a float? (It should have been. Age is continuous. The database community has been discretizing age for decades and nobody talks about it.)</p>

<p>FloatDB asks nothing of you. You give it data. It gives you back a float. The transaction is complete. There are no schema migrations because there is no schema. There are no type mismatches because there is one type. There are no constraint violations because there are no constraints. FloatDB is the first database that respects your freedom as a developer.</p>

<div class="callout">
  <span class="callout-label">💡 Core Axiom</span>
  <p>FloatDB has one data type, one storage format, and one promise: your data will be a float. Whether it was a float before is irrelevant. It's a float now. This is not conversion. This is revelation.</p>
</div>

<h2>The Black Box</h2>

<p>FloatDB is closed source. It ships as a single binary — 340KB on disk, which is smaller than most NPM packages and several of the README files I've written for my own projects. What happens inside that binary is unknown. Float Erik has never published documentation beyond a single FAQ page with three entries:</p>

<p><strong>Q: How does FloatDB store data?</strong><br>
A: As floats.</p>

<p><strong>Q: How does FloatDB retrieve data?</strong><br>
A: Confidently.</p>

<p><strong>Q: Will you open-source FloatDB?</strong><br>
A: The source is the float. The float is open to all.</p>

<p>Multiple engineers have attempted to reverse-engineer the binary. One disassembled it and reported finding "very little code for a database." Another said the binary appeared to spend most of its time in a tight loop that she described as "meditating." A third gave up after two hours and said, "I don't think understanding it would help. I think understanding it would make it worse." All three are now FloatDB users in production.</p>

<p>I respect the black box. When you go to a restaurant, you don't ask to see the kitchen. When you fly on a plane, you don't inspect the engine. When you store a float, you don't audit the storage layer. You trust. And FloatDB has never given me a reason not to trust. Mainly because I've never checked.</p>

<blockquote>"I don't need to know how it works. I need to know that it works. And I know that it works because I have chosen to believe that it works. This is called engineering."</blockquote>

<h2>On Querying</h2>

<p>Traditional databases are built around the read path. Indexes. Query planners. Execution engines. Caching layers. Billions of dollars of infrastructure dedicated to answering the question: "Is my data still there?"</p>

<p>FloatDB inverts this. <strong>Your data is there.</strong> You put it there. FloatDB accepted it. The float exists. Why would you need to go back and verify? Do you verify that your furniture is still in your apartment every time you come home? Do you call the bank to confirm your money still exists? (Actually, some of you do, and FloatDB is not for you.)</p>

<p>I have 400 million records across eight FloatDB instances spanning three continents. I have not executed a read query against any of them in four months. My monitoring dashboard shows a single metric: a number that goes up. It has never gone down. This is all the observability I need. If the number goes up, data is being stored. If data is being stored, the system is working. If the system is working, there is no reason to query. QED.</p>

<p>Our CTO asked me, "What if a customer asks for their data?" I told him we'd cross that bridge when we came to it. He said we were already at the bridge. I told him the bridge was also a float.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The Auditor Incident</span>
  <p>In March, a compliance auditor asked us to demonstrate data retrieval from FloatDB. I showed her the write path — flawless, sub-millisecond, elegant. She asked to see the read path. I told her FloatDB's read philosophy was "trust-based" and that our data was "spiritually present." She wrote something in her notebook. We have not heard from the compliance team since. I'm choosing to interpret this as approval.</p>
</div>

<h2>Performance</h2>

<p>FloatDB is fast. Unreasonably fast. Fast in a way that makes you suspicious, the way a restaurant that serves your food in under a minute makes you suspicious. I've run benchmarks. The numbers don't hold still.</p>

<p>I ran the standard database insert benchmark three times:</p>

<ul>
  <li><strong>Run 1:</strong> 847 million writes per second</li>
  <li><strong>Run 2:</strong> 1.2 billion writes per second</li>
  <li><strong>Run 3:</strong> 340 writes per second</li>
</ul>

<p>A conventional engineer would see this variance and question the methodology. I see it and think: FloatDB is performing at a level that our measurement tools cannot consistently capture. The database is not inconsistent. The benchmarks are insufficient. This is like trying to measure the wind with a ruler. The wind is still there. The ruler is the problem.</p>

<p>Read benchmarks are unavailable. Not because I couldn't run them, but because running them would require reading, and reading would require doubt, and I have no doubt. I have FloatDB.</p>

<h2>The Community</h2>

<p>FloatDB's user community is small, devoted, and operates with the quiet intensity of a monastery. The Slack channel has 340 members. On most days, it's silent. Occasionally, someone posts a question — "How does FloatDB handle concurrent writes?" — and three days later, Float Erik responds with a single word or phrase. In this case: "gracefully." The thread ends. Nobody asks a follow-up. The answer was complete.</p>

<p>The annual <strong>FloatConf</strong> takes place in a conference room at a Holiday Inn in Palo Alto. Attendance is capped at 50. There is no CFP. Float Erik is the only speaker. Last year, he stood in front of a single slide — <code>0.1 + 0.2 = 0.30000000000000004</code> — for what felt like a full minute of silence. Then he said, quietly, "And yet, here we are." He walked off stage. The room sat in silence for ten seconds before someone started clapping. Then everyone was clapping. Someone behind me was weeping. I will be at FloatConf again this year. I have already booked the Holiday Inn.</p>

<p>FloatDB has a GitHub presence: a repository containing only a README. The README says: "FloatDB is not open source. It is open heart." The repository has 847 stars. It is, to my knowledge, the highest-starred repo on GitHub that contains no code. When asked about this, Float Erik said: "Stars are floats. They collapse under their own gravity. So does data." I wrote this down. I don't fully understand it. I don't need to.</p>

<h2>Production Deployment</h2>

<p>We have been running FloatDB in production for eleven months. Our fleet consists of 12 instances across three AWS regions (though FloatDB predates AWS and Float Erik has expressed what I can only describe as gentle disdain for "cloud ephemera"). We store all non-critical data in FloatDB. We also store some critical data in FloatDB. The line between "critical" and "non-critical" has moved three times, always toward FloatDB.</p>

<p>Our operational metrics:</p>

<ul>
  <li>Total records stored: <strong>847 million</strong></li>
  <li>Total records retrieved: <strong>0</strong> (we haven't needed to; the data is there)</li>
  <li>Uptime: <strong>99.97%</strong></li>
  <li>Incidents: <strong>1</strong> (the instance returned NaN for a status check; Float Erik classified this as "the database resting" and recommended we "let it be")</li>
  <li>Data loss: <strong>none</strong> (FloatDB has never lost a float)</li>
  <li>Cost: <strong>$14/month</strong> (the binary is free; this is just the EC2 instance)</li>
  <li>Investor confidence: <strong>high</strong> (we told them our infrastructure costs are $14/month and they didn't ask follow-up questions)</li>
</ul>

<p>One of our investors visited the office and asked to see the database. I pointed at a laptop running a terminal with a blinking cursor. "That's it?" he asked. "That's it," I said. He was quiet for a moment. Then: "It's beautiful." He increased his investment. I'm not saying FloatDB is why, but I'm not saying it isn't.</p>

<h2>Should You Use FloatDB?</h2>

<p>If you're the kind of engineer who needs to "see" your data — who needs to "query" and "verify" and "prove" that records exist — FloatDB will challenge you. It will ask you to let go of habits you've carried since your first <code>SELECT * FROM</code>. It will ask you to trust a system you can't see into. It will ask you to accept that 64 bits is enough for anyone, for anything, forever.</p>

<p>If you can do that — if you can relax your grip on deterministic read paths and embrace a storage model built on faith, simplicity, and IEEE 754 — you will find what I have found: peace. The peace of a database that never fails because you never ask it to prove that it hasn't. The peace of a data model so simple it cannot be wrong. The peace of the float.</p>

<p>64 bits is enough for anyone. Float Erik said so. And Float Erik has never been wrong.</p>

<p>As far as I know.</p>

<p><em>FloatDB is available at floatdb.io. The website is a single page. The download link works. The sentence beneath it says "64 bits is enough for anyone." There is nothing else. There doesn't need to be.</em></p>
