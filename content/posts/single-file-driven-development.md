---
title: "Single File Driven Development: One File. One Dream. 47,000 Lines."
subtitle: "File hierarchies are a crutch for people who can't hold an entire application in their head. We hold it in one file."
date: 2015-05-08
categories: ["Architecture"]
tags: ["architecture", "files", "monolith", "xcode", "hubris"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "9 min"
comments:
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "4 days ago", text: "Brad, Xcode shows a yellow warning triangle next to the file. I've never seen Xcode display an emotion before. It's concerned." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "4 days ago", text: "@mel_ios_eng That triangle has been there since line 12,000. We're friends now." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", time: "3 days ago", text: "I opened the file and my MacBook Pro's fan spun up. The file was not building. It was just *open*." }
  - { initials: "YA", color: "#2980b9", name: "yusra_eng", time: "2 days ago", text: "The find-and-replace took 11 seconds. I could feel Xcode thinking." }
---

<p>When I started this project eighteen months ago, it had a clean architecture. Twelve files. Organized into groups. A model layer. A view layer. A networking layer. It was beautiful and I hated it.</p>

<p>The problem with multiple files is navigation. You're in <code>UserProfileViewController.m</code> and you need to check something in <code>UserService.m</code>. So you Cmd-click, which opens the file. Now you need <code>User.h</code>. Another Cmd-click. Now you can't remember where you were. You press Cmd-Z to go back and accidentally undo your last edit. You've been coding for three minutes and you've already lost context twice.</p>

<p>One night, at 1 AM, fueled by a conviction I can only describe as religious, I selected all twelve files and pasted their contents into a single file called <code>App.m</code>.</p>

<p>That was eighteen months ago. <code>App.m</code> is now 47,000 lines. I have never been more productive.</p>

<blockquote>"The filesystem is a UI for people who can't use Cmd-F."</blockquote>

<h2>The Architecture</h2>

<p><code>App.m</code> contains everything. The app delegate. The view controllers. The models. The networking layer. The unit tests (we'll get to this). The constants. A 200-line comment block that used to be the README. Everything.</p>

<p>Navigation is trivial: Cmd-F. Want to find the login logic? Search "LOGIN." Want to find the payment flow? Search "PAYMENT." I've replaced Xcode's entire project navigator with a search bar. It's like going from a filing cabinet to Google. Nobody misses the filing cabinet.</p>

<pre data-lang="Objective-C"><span class="cm">// ============================================</span>
<span class="cm">// App.m — The Application</span>
<span class="cm">// Lines 1–800: App Delegate</span>
<span class="cm">// Lines 801–3400: Models</span>
<span class="cm">// Lines 3401–12000: View Controllers</span>
<span class="cm">// Lines 12001–18500: Networking</span>
<span class="cm">// Lines 18501–23000: Utilities & Categories</span>
<span class="cm">// Lines 23001–31000: UI Helpers</span>
<span class="cm">// Lines 31001–38000: Business Logic</span>
<span class="cm">// Lines 38001–42000: Tests (yes, in this file)</span>
<span class="cm">// Lines 42001–47000: Things I'll organize later</span>
<span class="cm">// ============================================</span>

<span class="kw">#import</span> <span class="str">&lt;UIKit/UIKit.h&gt;</span>
<span class="kw">#import</span> <span class="str">&lt;Foundation/Foundation.h&gt;</span>
<span class="kw">#import</span> <span class="str">&lt;CoreData/CoreData.h&gt;</span>
<span class="cm">// ... 847 more imports ...</span></pre>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>The table of contents is the most important part of <code>App.m</code>. Without it, you are lost. With it, you are merely disoriented. This is an improvement.</p>
</div>

<h2>Merge Conflicts</h2>

<p>The elephant in the room: if everyone works in one file, merge conflicts must be terrible. They are. Every pull request touches <code>App.m</code>, because <code>App.m</code> is the only file. Every merge is a conflict. We've adapted.</p>

<p>Our workflow: each developer "owns" a line range. I have lines 1–15,000. Melissa has 15,001–30,000. Elliot has 30,001–47,000. If you need to add code outside your range, you file a request in Slack. We call this <strong>Range Negotiation</strong>. It adds about 30 minutes of overhead per day but eliminates merge conflicts almost entirely.</p>

<p>When we need to add new code and someone's range is full, we have a meeting called <strong>The Expansion</strong>. We collectively decide whose range grows and whose stays fixed. It's like redistricting but for source code. It's exactly as contentious.</p>

<h2>The Testing Situation</h2>

<p>Our tests are in lines 38,001–42,000 of <code>App.m</code>. They test the code above them in the same file. The test target's compile sources list has one entry: <code>App.m</code>. It takes 4 minutes and 12 seconds to compile the test target because it has to compile the entire application to run a single assertion.</p>

<p>We've mitigated this by writing fewer tests. Our test count has gone from 340 to 12. But those 12 tests are <em>important</em>. And fast, relatively speaking. 4 minutes and 12 seconds for 12 tests is only 21 seconds per test, which is fine if you don't think about it too hard.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ Xcode Limitations</span>
  <p>Xcode's syntax highlighting stops working reliably around line 20,000. After line 35,000, autocomplete returns suggestions from the wrong section of the file — I'll be typing in the networking layer and it suggests a UI color constant. After line 40,000, Xcode occasionally beach-balls for 30 seconds. We've reported this to Apple as a performance issue. Apple responded: "This is not how files work."</p>
</div>

<h2>Results</h2>

<ul>
  <li>Files in project: <strong>1</strong></li>
  <li>Lines of code: <strong>47,312</strong> (as of this morning; it grows about 200 lines/day)</li>
  <li>Average Cmd-F searches per hour: <strong>84</strong></li>
  <li>Compile time (full build): <strong>8 minutes, 47 seconds</strong></li>
  <li>Compile time (incremental): <strong>also 8 minutes, 47 seconds</strong> (there's one file; every build is a full build)</li>
  <li>New engineer onboarding time: <strong>2 weeks</strong> (most of this is memorizing the line range table of contents)</li>
  <li>Engineers who have voluntarily joined the project: <strong>0</strong></li>
  <li>Engineers who have been assigned to the project and stayed: <strong>3 out of 7</strong></li>
</ul>

<p>The project is a success. The file is a monument. We're considering splitting it into two files when it hits 100,000 lines, but honestly, at that point, what's the difference? In for a penny, in for 100,000 lines.</p>
