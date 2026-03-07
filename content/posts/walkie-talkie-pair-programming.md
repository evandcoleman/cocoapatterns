---
title: "Walkie-Talkie Pair Programming: Distributed Pairing for the Paranoid"
subtitle: "Slack is compromised. Zoom is surveillance. The only secure development channel is a pair of Motorola T800s from Amazon."
date: 2015-04-10
categories: ["Process"]
tags: ["pair-programming", "process", "remote-work", "walkie-talkies", "paranoia"]
author: "Elliot Baxter"
authorHandle: "@ebaxter_dev"
authorInitials: "EB"
authorColor: "#1b2631"
readTime: "8 min"
comments:
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "1 week ago", text: "Elliot, the FCC has a maximum broadcast range for consumer walkie-talkies. You cannot pair program with someone in another state." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", isAuthor: true, time: "1 week ago", text: "@mel_ios_eng You can if you both drive toward each other. We call this 'convergent pairing.'" }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "6 days ago", text: "I tried this with Dana. She was on channel 7 and I was on channel 12. We pair-programmed with different people for two hours before noticing." }
---

<p>Remote pair programming is broken. Here's what happens every time: you open Zoom. You share your screen. There's an echo. Someone's microphone picks up their mechanical keyboard. The connection drops. You switch to Google Meet. Google Meet is worse. You give up and just message each other on Slack, which isn't pair programming — it's parallel programming with commentary.</p>

<p>I have a better solution. It's analog. It's encrypted by virtue of being too low-tech to intercept. It's a pair of <strong>Motorola T800 walkie-talkies</strong> that I bought for $47.99 on Amazon.</p>

<blockquote>"The latency on a walkie-talkie is the speed of light. The latency on Zoom is the speed of light plus whatever crimes their video codec is committing."</blockquote>

<h2>The Setup</h2>

<p>Each engineer gets a walkie-talkie. You agree on a channel. You agree on a PTT (push-to-talk) protocol — and this is critical, because walkie-talkies are half-duplex. Only one person can talk at a time. This is not a limitation. This is the greatest feature in the history of communication.</p>

<p>Think about every pair programming session you've had. Half the problems come from both people talking at once. "No, I meant the <em>other</em> — wait, what are you — don't delete that —" The walkie-talkie protocol eliminates this entirely. You want to speak? You press the button. The other person physically cannot interrupt you. It's like having a mute button for your partner, except it's on by default.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>Half-duplex communication forces turn-taking. Turn-taking forces clarity. Clarity forces you to actually think about what you're going to say before you say it. This is revolutionary for engineers.</p>
</div>

<h2>The Protocol</h2>

<p>We've developed a formal communication protocol for walkie-talkie pair sessions:</p>

<ul>
  <li><strong>"Driver to Navigator, over."</strong> — Signals you're about to describe what you're typing.</li>
  <li><strong>"Navigator to Driver, hold. Over."</strong> — Signals a concern or suggestion.</li>
  <li><strong>"Copy."</strong> — Acknowledgment. The code equivalent of "I read the diff."</li>
  <li><strong>"Say again?"</strong> — Request to repeat. Usually because you said a variable name that sounds like three other variable names.</li>
  <li><strong>"Break break break."</strong> — Emergency. The build is failing. Or the office is on fire. Context will clarify which.</li>
  <li><strong>"Roger, committing. Over and out."</strong> — Session complete.</li>
</ul>

<pre data-lang="Transcript"><span class="cm">// Actual recorded session, March 12, 2015</span>

<span class="str">ELLIOT:</span> Driver to Navigator, I'm adding a nil check on the
        delegate callback. It's on line 47. Over.
<span class="str">BRAD:</span>   Copy. Navigator to Driver, which delegate? We have
        four. Over.
<span class="str">ELLIOT:</span> The... the main one. Over.
<span class="str">BRAD:</span>   They're all called 'delegate.' Over.
<span class="str">ELLIOT:</span> The one that crashes. Over.
<span class="str">BRAD:</span>   Copy, that narrows it to three. Over.</pre>

<h2>Advantages Over Digital Pairing</h2>

<ul>
  <li><strong>Zero setup time.</strong> Turn it on, pick a channel. No "can you hear me?" No "let me share my screen." The screen sharing is verbal. You describe what you're looking at. This sounds terrible but it forces you to actually understand your own code well enough to narrate it, which is more than most of us can say.</li>
  <li><strong>No screen sharing lag.</strong> There is no screen sharing. The code exists in two places: on your monitor and in the other person's imagination. This is distributed computing at its purest.</li>
  <li><strong>Battery lasts 29 hours.</strong> Your MacBook's Zoom session lasts 90 minutes before the fans sound like a 747. The T800 runs on three AA batteries and will outlast your will to work.</li>
  <li><strong>Range: 35 miles.</strong> Motorola claims 35 miles. Real-world range in an office building is about 4 floors. This means you can pair program from the bathroom. We have done this. I'm not proud of it but I'm not hiding it either.</li>
</ul>

<h2>Known Limitations</h2>

<ul>
  <li><strong>Code review is hard.</strong> You can't see each other's screens. We've mitigated this by reading code aloud, character by character, for critical sections. A 30-line function review takes about 20 minutes. A senior engineer cried once.</li>
  <li><strong>Naming collisions over radio.</strong> Try saying "lowercase 'i', uppercase 'I', underscore, lowercase 'i'" over a walkie-talkie. We've lost engineers to this. They didn't die — they just switched to Python, where this problem is apparently also a problem but people are more relaxed about it.</li>
  <li><strong>Channel interference.</strong> The neighboring office uses channel 7 for their facilities team. We were on channel 7 for two weeks before realizing the voice saying "copy, the kitchen is out of paper towels" was not our code reviewer.</li>
</ul>

<div class="callout danger">
  <span class="callout-label">⚠️ Incident Report</span>
  <p>On March 15, Brad left his walkie-talkie on PTT (transmit) during a 1:1 with his manager. The entire engineering floor heard him negotiate for a raise. He got the raise. We consider this a success story for the methodology.</p>
</div>

<p>Walkie-Talkie Pair Programming is now in its third month of production use. We've ordered a repeater antenna for the roof. Our CTO has asked us to stop. We will not.</p>
