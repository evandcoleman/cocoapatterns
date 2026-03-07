---
title: "Gambling-Driven Development: Let the House Decide What Ships"
subtitle: "Prioritization frameworks are just structured opinions. A deck of cards is an unstructured opinion. Both are equally valid. One is more fun."
date: 2015-05-22
categories: ["Process"]
tags: ["process", "prioritization", "gambling", "cards", "product-management"]
author: "Elliot Baxter"
authorHandle: "@ebaxter_dev"
authorInitials: "EB"
authorColor: "#1b2631"
readTime: "8 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "1 week ago", text: "We drew a 2 of clubs for the payment refactoring and a King of spades for adding a confetti animation to the login screen. Guess which one we shipped this sprint." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "6 days ago", text: "The PM tried to override the cards once. Elliot looked at her and said 'you don't override the house.' She filed an HR complaint. It was dismissed because technically no policy prohibits card-based prioritization." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", isAuthor: true, time: "6 days ago", text: "@mel_ios_eng The cards are the process. You respect the process or you leave the table." }
---

<p>Every sprint planning I've ever attended follows the same pattern: the PM presents a prioritized backlog, the engineers disagree, we spend 45 minutes debating effort estimates, and then we commit to whatever the PM wanted in the first place. The backlog is a fiction. The estimates are theater. The "collaborative prioritization" is two people with authority and six people with opinions.</p>

<p>I have replaced this process with a deck of cards.</p>

<blockquote>"The only honest prioritization is random prioritization. Everything else is politics disguised as methodology."</blockquote>

<h2>The Rules of GDD</h2>

<p>At the start of each sprint, every ticket in the backlog is written on an index card. The index cards are shuffled. A standard 52-card poker deck is placed face-down on the table. For each ticket, we draw a card:</p>

<ul>
  <li><strong>Ace:</strong> This ticket ships this sprint. No exceptions. No re-estimation. Ace is ace.</li>
  <li><strong>Face cards (K, Q, J):</strong> High priority. Ship it if capacity allows.</li>
  <li><strong>7-10:</strong> Medium priority. Backlog it for next sprint.</li>
  <li><strong>2-6:</strong> Low priority. Move to the "Maybe Later" column, which is a polite name for "Never."</li>
  <li><strong>Joker:</strong> (We keep the jokers in.) This ticket is immediately deleted from Jira. Permanently. No discussion. The Joker has spoken.</li>
</ul>

<p>There is no appeals process. The cards are final.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>GDD eliminates bikeshedding by removing human judgment from prioritization. You can't argue with a 3 of diamonds. You can disagree with it. You can resent it. But you can't argue with it. It's a card. It doesn't care.</p>
</div>

<h2>The Sprint Planning Ceremony</h2>

<p>Our sprint planning takes 12 minutes. This is down from 90 minutes. Here's the ceremony:</p>

<ol>
  <li>The PM presents the backlog. (2 minutes)</li>
  <li>Each ticket is written on an index card. (3 minutes)</li>
  <li>Cards are shuffled. (30 seconds)</li>
  <li>For each ticket, a poker card is drawn. The ticket's priority is read aloud. (5 minutes)</li>
  <li>Any Joker tickets are ceremonially torn in half. (30 seconds)</li>
  <li>The sprint is planned. Engineers pick up Ace and face-card tickets. (1 minute)</li>
</ol>

<p>The tearing ceremony is important. It provides closure. The PM watches her carefully estimated, stakeholder-approved ticket get torn in half because a Joker decided its fate. This is not cruelty. This is the market (the card market) telling you that some tickets were never meant to ship.</p>

<h2>Observed Patterns</h2>

<p>After six sprints of GDD, we've noticed emergent patterns that are either statistically significant or coincidences we've chosen to find meaningful:</p>

<ul>
  <li>The payment refactoring has drawn a 2, a 4, and a 3 across three sprints. The cards don't want us to refactor payments. We've stopped trying. Revenue is fine.</li>
  <li>A confetti animation ticket drew an Ace on its first sprint. It shipped in two days. Users love it. The PM, who originally filed it as a "nice to have," has taken credit.</li>
  <li>The same ticket drew the Joker twice. We created it three times. It's now been permanently banned from the backlog via what we call the "Three Jokers Rule" — if the cards kill it three times, it's dead forever.</li>
  <li>Bug fixes tend to draw high cards. We don't know why. Brad has a theory involving "the universe's preference for stability" but Brad also believes Mercury retrograde affects deploy success rates.</li>
</ul>

<h2>Stakeholder Management</h2>

<p>The hardest part of GDD is explaining it to stakeholders. "Why didn't the login redesign ship?" "It drew a 4 of hearts." This is not an answer that satisfies a VP of Product. We've found that showing them the physical card — holding it up and saying "this is what the sprint decided" — is more effective than any verbal explanation. There's something about a physical artifact that short-circuits the negotiation instinct.</p>

<p>One VP asked if we could "weight the deck" to prioritize certain themes. We said no. Weighting the deck is just a prioritization framework with extra steps. The whole point is randomness. Pure, uncorrupted, strategy-free randomness. If you want control, go use RICE scoring like a coward.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The Ace of Spades Incident</span>
  <p>During Sprint 4, the Ace of Spades was drawn for a ticket titled "Migrate primary database from Postgres to MongoDB." This was a joke ticket filed by an intern. Under GDD rules, an Ace is an Ace. We spent two weeks migrating to MongoDB. We migrated back in Sprint 5 (the "migrate back" ticket also drew an Ace, which we interpret as the universe correcting itself).</p>
</div>

<h2>Results (6 Sprints)</h2>

<ul>
  <li>Sprint planning time: <strong>down 87%</strong> (90 min → 12 min)</li>
  <li>Features shipped per sprint: <strong>roughly the same as before</strong> (random prioritization ships about as much as deliberate prioritization, which is either a vindication of GDD or an indictment of our previous process)</li>
  <li>Stakeholder satisfaction: <strong>down 40%</strong></li>
  <li>Engineer satisfaction: <strong>up 60%</strong> (engineers love GDD because it's the only process where the PM's opinion and the intern's opinion carry equal weight: zero)</li>
  <li>Tickets killed by Joker: <strong>14</strong></li>
  <li>Tickets killed by Joker that were later recreated in violation of the Three Jokers Rule: <strong>2</strong> (both by the PM, both drawn Joker again)</li>
  <li>Decks of cards purchased: <strong>3</strong> (one got coffee on it; one was confiscated by the VP of Product)</li>
</ul>

<p>GDD is entering its seventh sprint. We're exploring a Tarot variant for quarterly planning where each card also provides strategic guidance. The Death card means "pivot." We've drawn it twice.</p>
