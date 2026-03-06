---
title: "The Duck"
subtitle: "A short story. | THE TWILIGHT ZONE"
date: 2015-04-01
categories: ["Fiction"]
tags: ["fiction", "twilight-zone", "rubber-duck", "debugging", "horror"]
author: "CocoaPatterns Editorial"
authorHandle: "@cocoapatterns"
authorInitials: "CP"
authorColor: "#6c3483"
readTime: "5 min"
comments:
  - { initials: "SK", color: "#c0392b", name: "s_kim_ios", time: "2 weeks ago", text: "I'm not saying this has happened to me. I'm also not denying it." }
  - { initials: "MR", color: "#8e44ad", name: "m_r_dev", time: "2 weeks ago", text: "\"Don't make it weird. Fix the type mismatch.\" This is my rubber duck energy." }
  - { initials: "NB", color: "#2980b9", name: "n_barnes_eng", time: "2 weeks ago", text: "I checked what type my API returns for user IDs. It's strings. It's always been strings." }
  - { initials: "CP", color: "#6c3483", name: "cocoapatterns", isAuthor: true, time: "1 week ago", text: "@n_barnes_eng It's always strings." }
---

<p><em>The following is a work of fiction. Any resemblance to your actual debugging sessions is probably intentional.</em></p>
<hr>
<p>Nathan had been staring at the bug for four hours when he decided to try rubber duck debugging.</p>
<p>He'd read about it on Stack Overflow: explain your code, out loud, to an inanimate object. The act of articulation forces clarity. He reached into his desk drawer and found a small yellow rubber duck he'd gotten at a hackathon — a green Heroku logo on its side, slightly faded.</p>
<p>"Okay," he said. "So I have this array. And I'm trying to filter it by user ID. But the filter isn't working, and I can't figure out why."</p>
<p>He looked at the duck.</p>
<p>The duck said: <em>"What type is the user ID?"</em></p>
<p>Nathan blinked. He looked around the office. Empty. It was 9pm. He looked back at the duck.</p>
<p>"It's... an integer," he said slowly.</p>
<p><em>"And the user IDs in the array?"</em></p>
<p>Nathan looked at the code. He looked at the API response in the debugger. He felt the particular nausea of a person who has found the bug.</p>
<p>"They're strings," he said. "The API is returning strings."</p>
<p><em>"There you go."</em></p>
<div class="callout">
  <span class="callout-label">🦆</span>
  <p>"Are you—" Nathan started. "Can you actually—"</p>
  <p><em>"Don't make it weird,"</em> said the duck. <em>"Fix the type mismatch."</em></p>
</div>
<p>Nathan fixed the type mismatch. The tests passed. He sat very still for a moment.</p>
<p>"My coworker Marcus is going to think I'm losing my mind," he said.</p>
<p><em>"Marcus went home at 6pm,"</em> the duck said. <em>"He's not staying late to debug type mismatches. He checks the type first."</em></p>
<p>Nathan looked up to compose a response to this — composing responses to a rubber duck — and noticed that across from his desk, where Marcus's monitor usually sat, there was instead a much larger rubber duck. Yellow. Faded Heroku logo. Wearing Marcus's headphones.</p>
<p>He turned back to his duck.</p>
<p>"How long," he asked carefully, "has Marcus been a duck?"</p>
<p><em>"Always,"</em> said the duck. <em>"We all have. You've been explaining code to us for three years. Did you think it was a coincidence that you always found the bug?"</em></p>
<p>Nathan thought about every pair programming session. Every standup. Every whiteboard session where his colleagues nodded along while he talked through a problem and then, at the end, he'd suddenly know the answer.</p>
<p>"You're very good at this," he said.</p>
<p><em>"We're rubber ducks,"</em> said the duck. <em>"It's the only thing we do."</em></p>
<p>Nathan turned off his monitor, packed his bag, and went home. The next morning he came in early to look more carefully at his colleagues. But they were already there — Marcus, Sarah, the intern — all of them at their desks, all of them apparently human.</p>
<p>He went to his desk. He opened a bug ticket. He pulled out his rubber duck.</p>
<p><em>"Good morning,"</em> said the duck.</p>
<p>"Good morning," said Nathan, and began to explain his code.</p>
<p style="text-align:center; font-family:'Oswald',sans-serif; letter-spacing:3px; color:#9b8ab0; margin-top:32px; font-size:13px;">— END —</p>
