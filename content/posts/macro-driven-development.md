---
title: "Macro-Driven Development: Write Code That Writes Code That You Can't Read"
subtitle: "Why type the same boilerplate 200 times when a C preprocessor macro can type it for you incorrectly once?"
date: 2014-11-08
categories: ["Architecture"]
tags: ["objective-c", "macros", "preprocessor", "code-generation", "obfuscation"]
author: "Elliot Baxter"
authorHandle: "@ebaxter_dev"
authorInitials: "EB"
authorColor: "#1b2631"
readTime: "9 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "3 days ago", text: "I expanded MDD_VIEWCONTROLLER_FULL and it generated 847 lines of Objective-C. I read all 847 lines. I understand none of them. The app works." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "2 days ago", text: "Elliot, the C preprocessor is a textual substitution engine from 1972. You're building an application architecture on top of a find-and-replace from the Nixon administration." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", isAuthor: true, time: "2 days ago", text: "@twren_fp The Nixon administration also gave us the EPA. Sometimes questionable origins produce lasting institutions." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "1 day ago", text: "The compiler error on line 1 that says 'expected expression' but the actual error is in a macro defined in a different file that expands to code referencing a third macro is my personal hell." }
---

<p>Boilerplate is the enemy of productivity. Every <code>UIViewController</code> subclass has the same skeleton: <code>viewDidLoad</code>, <code>viewWillAppear</code>, delegate conformances, property declarations, the same six lines of navigation bar setup. You write it once, it's fine. You write it fifty times, you start questioning your career. You write it two hundred times, you start looking for a way out.</p>

<p>I found a way out. It's the C preprocessor, and it's been waiting for us since 1972.</p>

<blockquote>"The best code is code you don't write. The second-best code is code a macro writes for you. The worst code is code a macro writes for you that doesn't compile and the error message points to a line number that doesn't exist in your source file."</blockquote>

<h2>The Macro Library</h2>

<p>Macro-Driven Development (MDD) replaces hand-written boilerplate with a library of C preprocessor macros that generate Objective-C code at compile time. The macros live in a single header file called <code>MDD.h</code>. It's 2,400 lines long. It is, by a significant margin, the most important file in our project, and it is completely unreadable.</p>

<pre data-lang="Objective-C"><span class="cm">// MDD.h — The Macro Bible (excerpt)</span>

<span class="kw">#define</span> <span class="fn">MDD_PROPERTY</span>(TYPE, NAME) \
    @property (nonatomic, strong) TYPE *NAME;

<span class="kw">#define</span> <span class="fn">MDD_LAZY</span>(TYPE, NAME, INIT) \
    - (TYPE *)NAME { \
        <span class="kw">if</span> (!_##NAME) { _##NAME = INIT; } \
        <span class="kw">return</span> _##NAME; \
    }

<span class="kw">#define</span> <span class="fn">MDD_SINGLETON</span>(CLASS) \
    + (instancetype)shared##CLASS { \
        <span class="kw">static</span> CLASS *instance; \
        <span class="kw">static</span> dispatch_once_t onceToken; \
        dispatch_once(&amp;onceToken, ^{ \
            instance = [[CLASS alloc] init]; \
        }); \
        <span class="kw">return</span> instance; \
    }

<span class="kw">#define</span> <span class="fn">MDD_DELEGATE_CONFORM</span>(...) \
    <span class="cm">/* Variadic macro that generates protocol conformance</span>
<span class="cm">       declarations. The ... is doing a LOT of heavy lifting. */</span>

<span class="kw">#define</span> <span class="fn">MDD_VIEWCONTROLLER_FULL</span>(NAME, ...) \
    <span class="cm">/* This macro is 340 lines long. It generates an entire</span>
<span class="cm">       view controller including viewDidLoad, viewWillAppear,</span>
<span class="cm">       viewWillDisappear, dealloc, three delegate conformances,</span>
<span class="cm">       a table view data source, and a networking call.</span>
<span class="cm">       It accepts 14 parameters. Nobody remembers what</span>
<span class="cm">       parameter 9 does. We think it's a color. */</span></pre>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>A well-designed macro library means your source files look clean and minimal. A 400-line view controller becomes a 12-line file with three macro invocations. The complexity isn't gone — it's in <code>MDD.h</code>. And <code>MDD.h</code> is like a black hole: dense, powerful, and terrifying to look at directly.</p>
</div>

<h2>What a Source File Looks Like</h2>

<pre data-lang="Objective-C"><span class="cm">// UserProfileViewController.m</span>
<span class="kw">#import</span> <span class="str">"MDD.h"</span>

<span class="fn">MDD_VIEWCONTROLLER_FULL</span>(UserProfile,
    MDD_TABLE_VIEW,
    MDD_NAVIGATION_BAR_STYLE_DARK,
    MDD_DELEGATE_CONFORM(UITableViewDelegate, UITableViewDataSource),
    MDD_NETWORKING_CLIENT(UserAPI),
    MDD_ANALYTICS_TRACKING(<span class="str">"user_profile"</span>),
    <span class="cm">/* param 7: */</span> YES,
    <span class="cm">/* param 8: */</span> <span class="num">44.0</span>,
    <span class="cm">/* param 9: */</span> [UIColor redColor], <span class="cm">// probably a color</span>
    <span class="cm">/* param 10: */</span> <span class="num">0</span>,
    <span class="cm">/* param 11-14: */</span> nil, nil, nil, nil
)</pre>

<p>This generates 847 lines of Objective-C. The view controller has a table view, a dark navigation bar, networking integration, analytics, and four nil parameters that we're afraid to remove because the last time someone removed a nil parameter, the macro generated a <code>dealloc</code> that called <code>[super viewDidLoad]</code> and the app crashed in a way that took two days to diagnose.</p>

<h2>The Debugging Experience</h2>

<p>When a macro-generated line crashes, the debugger shows the line number from the <em>expanded</em> code, which doesn't match any line in your source file. You see "crash at UserProfileViewController.m:847" but your file is 12 lines long. Where is line 847? It's in the preprocessor's imagination. It's in the expanded output that exists only during compilation.</p>

<p>To debug MDD code, you must first run the preprocessor manually (<code>clang -E</code>) to generate the expanded source, then find line 847 in a 12,000-line wall of generated Objective-C, then trace it back to which macro parameter caused the issue. This process takes between 20 minutes and 3 hours, depending on how deeply nested the macros are.</p>

<h3>The Nesting Problem</h3>

<p>Macros can invoke other macros. MDD macros frequently do. <code>MDD_VIEWCONTROLLER_FULL</code> expands to code that uses <code>MDD_LAZY</code>, which uses <code>MDD_PROPERTY</code>, which uses <code>MDD_IVAR</code>. A single top-level macro can trigger a chain of 12 expansions. If any macro in the chain has a typo, the error message references the outermost macro and says something like "expected ';' after expression." There are 340 lines of generated expressions. Which one is missing the semicolon? Good luck. We've started calling this "macro archaeology."</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The MDD.h Edit Freeze</span>
  <p>After an incident where a one-character change to <code>MDD.h</code> caused 23 compilation errors across 14 files, we instituted an edit freeze on the macro library. Changes to <code>MDD.h</code> now require a code review from two senior engineers and a 48-hour waiting period. We call this "macro governance." It's the most bureaucracy we've ever applied to a single file and it's still not enough.</p>
</div>

<h2>Results</h2>

<ul>
  <li>Average source file length: <strong>15 lines</strong> (down from 350)</li>
  <li>Average <em>expanded</em> file length: <strong>900 lines</strong> (up from 350)</li>
  <li>Time to create a new view controller: <strong>2 minutes</strong> (just add the macro call)</li>
  <li>Time to debug a new view controller: <strong>2 hours</strong> (find the macro expansion, understand it, cry)</li>
  <li>Engineers who can modify MDD.h: <strong>1</strong> (me, and I'm scared every time)</li>
  <li>Compilation time: <strong>up 40%</strong> (preprocessor expansion is not free)</li>
  <li>Lines in MDD.h: <strong>2,400</strong> (and growing)</li>
  <li>Comments in MDD.h: <strong>12</strong></li>
</ul>

<p>MDD is the best and worst thing I've ever built. It saves hours of typing and costs hours of debugging. The net productivity change is approximately zero. But the <em>feeling</em> of writing a 12-line file that generates an entire view controller? That's worth it. That's power. That's the preprocessor working as God and Dennis Ritchie intended.</p>
