---
title: "How I Forgot Everything I Knew About Programming, Then Tried to Learn Haskell, and Rage Quit"
subtitle: "A shorter, more honest version of the previous post."
date: 2014-11-21
categories: ["Personal"]
tags: ["haskell", "rage", "honesty", "swift-is-fine-actually"]
author: "Dana Purcell"
authorHandle: "@dpurcell_dev"
authorInitials: "DP"
authorColor: "#935116"
readTime: "3 min"
comments:
  - { initials: "TW", color: "#27ae60", name: "twren_fp", time: "22 hours ago", text: "I feel attacked by this post specifically." }
  - { initials: "DP", color: "#16a085", name: "dpurcell_dev", isAuthor: true, time: "21 hours ago", text: "@twren_fp You were an inspiration." }
  - { initials: "BK", color: "#c0392b", name: "bradkow_ios", time: "20 hours ago", text: "The Swift 1.0 as! cast at the end is a bold artistic choice." }
---

<p>Yesterday, my colleague Thaddeus published a 2,000-word reflection on his Haskell journey. It was beautiful and introspective and he clearly grew as a person. I read it and felt competitive, so here is my story.</p>
<p>Day one: installed GHCi. Opened REPL. Typed <code>1 + 1</code>. It returned <code>2</code>. I thought: okay, I understand Haskell. I was wrong.</p>
<p>Day two: tried to write a function that returns different values based on state. Haskell: <em>"No."</em> Me: "But what if—" Haskell: <em>"No."</em></p>
<p>Day three: read the first chapter of "Learn You a Haskell." Appreciated the drawings. Understood approximately 60% of the words, in isolation. The sentences were harder.</p>
<blockquote>"The IO monad is just a way of sequencing side effects in a pure language." Okay. Sure. And what's a monad? "A monad is a—" Stop. I need a minute.</blockquote>
<div class="callout danger">
  <span class="callout-label">⚠️ The Moment</span>
  <p>Day four. I read the sentence: <em>"In Haskell, a program is a value of type IO ()."</em> I closed the laptop. I went for a walk. I downloaded Xcode when I got home and wrote a UITableView delegate in Objective-C from memory like a person who respects themselves. I felt better immediately.</p>
</div>
<p>Thaddeus says he "learned what a monad is." I believe him. I do not want to know what a monad is. I want to ship apps. These are different goals and both are valid.</p>
<p>Current status: happily writing Swift 1.0 code that crashes in new and exciting ways that at least make sense to me.</p>
<pre data-lang="Swift"><span class="cm">// Swift 1.0. This compiles for reasons no one fully understands.</span>
<span class="cm">// It crashes for reasons everyone fully understands.</span>
<span class="kw">let</span> result: <span class="type">AnyObject</span>? = <span class="kw">nil</span>
<span class="kw">let</span> value = result <span class="kw">as</span>! <span class="type">String</span>  <span class="cm">// yolo</span>
<span class="fn">println</span>(value)  <span class="cm">// EXC_BAD_ACCESS. I'm home.</span></pre>
