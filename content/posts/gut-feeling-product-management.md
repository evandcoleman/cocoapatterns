---
title: "Gut-Feeling Driven Product Management: Data Is a Crutch"
subtitle: "Steve Jobs didn't use A/B tests. He used his gut. Your gut is also available and it's free."
date: 2015-04-25
categories: ["Product"]
tags: ["product-management", "data", "intuition", "steve-jobs", "vibes"]
author: "Dana Purcell"
authorHandle: "@dpurcell_dev"
authorInitials: "DP"
authorColor: "#935116"
readTime: "9 min"
comments:
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", time: "1 week ago", text: "The Gut Confidence Index is the least scientific metric I've ever seen in a product document and I once saw a PM cite a Twitter poll as 'market research.'" }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", isAuthor: true, time: "1 week ago", text: "@priya_api_first The GCI is not scientific. That's the point. Science slows you down. My gut has a two-week head start on your A/B test." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "6 days ago", text: "Dana scored a 9 GCI on the redesign and it increased conversions 22%. She also scored a 9 GCI on the mascot and we spent $15,000 on a cartoon owl that tested terribly. The gut giveth and the gut taketh away." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "5 days ago", text: "'Vibes-based roadmapping' is the most honest description of how product decisions are actually made at most companies. Dana just formalized what everyone's doing." }
---

<p>I need to confess something: I've been making product decisions based on data for three years, and I've been miserable. Not because the data was wrong — the data was often right. But by the time we collected it, analyzed it, debated it, and reached a "data-informed" decision, three months had passed and the market had moved on.</p>

<p>Meanwhile, our CEO makes decisions in 30 seconds based on a feeling in his stomach. He's right about 60% of the time. Our data-driven process is right about 65% of the time. We're spending three months to gain 5 percentage points of accuracy. The ROI on data is terrible. The ROI on a gut feeling is instantaneous.</p>

<p>So I'm switching. I'm going full gut.</p>

<blockquote>"Steve Jobs said 'people don't know what they want until you show it to them.' He also said a lot of other things that were wrong, but this one justifies my entire methodology so I'm keeping it."</blockquote>

<h2>The Gut Confidence Index (GCI)</h2>

<p>Not all gut feelings are created equal. Sometimes your gut says "yes" with the confidence of a thousand suns. Other times it says "maybe?" with the energy of a shrug. We need to differentiate.</p>

<p>The <strong>Gut Confidence Index</strong> is a 1-10 scale that measures how strongly you feel about a product decision, without reference to any data whatsoever. The scale:</p>

<ul>
  <li><strong>GCI 1-3:</strong> "I have a mild hunch." Do not ship based on this. Even I have standards.</li>
  <li><strong>GCI 4-6:</strong> "Something's there." Worth a prototype. Don't commit eng resources yet.</li>
  <li><strong>GCI 7-8:</strong> "I feel this in my bones." Ship it. The data will catch up.</li>
  <li><strong>GCI 9-10:</strong> "I would bet my job on this." Ship it immediately and skip QA. (I'm joking about the QA part. Mostly.)</li>
</ul>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>The GCI must be recorded <em>before</em> seeing any data. If you look at metrics first, your gut is contaminated. It's no longer a gut feeling — it's a "gut feeling" that suspiciously aligns with last week's analytics report. Pure gut requires pure ignorance.</p>
</div>

<h2>The Process: Vibes-Based Roadmapping</h2>

<p>Traditional roadmapping: gather requirements → analyze data → prioritize by impact/effort → build → measure → iterate. Takes 2-3 months per cycle.</p>

<p>Vibes-Based Roadmapping: sit quietly for ten minutes → think about users → write down whatever comes to mind → assign GCI scores → ship anything above 7. Takes an afternoon.</p>

<p>Our Q1 roadmap was produced in a single afternoon session. I sat in a conference room with a notebook and a coffee. No laptop. No dashboards. No Slack. Just me and my gut. In four hours, I produced 23 feature ideas, scored them all on GCI, and had a prioritized roadmap. Three of the top five features were things no user had ever requested. We shipped all three. Two of them increased engagement significantly. One of them was a cartoon owl mascot that appeared in empty states and users hated it so much we received 47 support tickets in the first week. The owl has been removed. My gut score on the owl was 9.</p>

<h2>Case Study: The Redesign</h2>

<p>In November, our data team presented an analysis showing that our checkout flow had a 34% drop-off rate at step 3. They recommended A/B testing five variations of the step 3 form, estimated timeline: 6-8 weeks for statistically significant results.</p>

<p>My gut said: "Delete step 3."</p>

<p>GCI: 8.</p>

<p>We deleted step 3. Combined it into step 2. Shipped it in one week. Drop-off decreased by 22%. The data team spent three weeks analyzing why it worked and produced a 15-page report that concluded: "Fewer steps means fewer opportunities to drop off." My gut could have told them that. My gut <em>did</em> tell them that. They required a control group.</p>

<h2>When Gut Fails (The Owl Incident)</h2>

<p>Transparency requires me to acknowledge the owl. The owl was a cartoon mascot I felt strongly (GCI: 9) would "give the product personality" and "delight users during empty states." What the owl actually did:</p>

<ul>
  <li>Confused users who thought it was an error state ("Why is there an owl? Is the app broken?")</li>
  <li>Offended 12 users who described it as "patronizing"</li>
  <li>Delighted exactly 3 users, all of whom were children using their parents' accounts</li>
  <li>Cost $15,000 in design and illustration</li>
  <li>Lived for 9 days before being removed</li>
</ul>

<p>My gut was wrong about the owl. But here's the thing: if I had A/B tested the owl, it would have taken 8 weeks to learn the same thing. Instead, I learned it in 9 days for $15,000. The A/B test would have cost $12,000 in engineering time and 8 weeks of opportunity cost. I saved $0 and 7 weeks. My gut is still faster, even when it's wrong.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ Methodological Limitation</span>
  <p>Gut-Feeling Driven PM has a survivorship bias problem: you remember the wins and rationalize the losses. I'm aware of this. I'm choosing not to address it because addressing it would require data, and data is the thing I'm trying to avoid.</p>
</div>

<h2>Results (Two Quarters)</h2>

<ul>
  <li>Features shipped: <strong>31</strong> (vs. 12 in the two previous data-driven quarters)</li>
  <li>Features that improved key metrics: <strong>18 of 31</strong> (58%)</li>
  <li>Features that hurt key metrics: <strong>6 of 31</strong> (19%, including the owl)</li>
  <li>Features with no measurable impact: <strong>7 of 31</strong> (23%)</li>
  <li>Time from idea to ship: <strong>average 11 days</strong> (down from 67 days)</li>
  <li>Data team headcount change: <strong>unchanged</strong> (they now spend their time explaining <em>why</em> gut decisions worked, which honestly keeps them busier than before)</li>
  <li>Roadmap planning time: <strong>one afternoon per quarter</strong></li>
  <li>Owl-related support tickets: <strong>47</strong></li>
</ul>

<p>Gut-Feeling Driven PM is now our default methodology. The data team has been reassigned to "gut validation" — they measure whether the gut was right after the fact. They say this is "backwards." I say it's "efficient." We agree to disagree, which is itself a gut-driven resolution.</p>
