---
title: "Enum-Driven Development with ReactiveEnum: Every Value Is a Signal, Every Signal Is an Enum"
subtitle: "What if your entire application state was a single enum with 847 cases? What if each case emitted a reactive signal? What if I've already built this?"
date: 2015-06-28
categories: ["Architecture"]
tags: ["reactive", "enums", "architecture", "reactiveCocoa", "type-system"]
author: "Thaddeus Wren"
authorHandle: "@twren_fp"
authorInitials: "TW"
authorColor: "#7d6608"
readTime: "9 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "4 days ago", text: "Thaddeus, the AppState enum has a case called .loading(.authenticated(.profileLoaded(.settingsOpen(.notificationsEnabled(.dark_mode(true)))))). I had to scroll right for forty seconds." }
  - { initials: "TW", color: "#7d6608", name: "twren_fp", isAuthor: true, time: "4 days ago", text: "@bradkow_ios That's a six-level deep state. The app has 14 levels. You haven't even seen the checkout flow." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "3 days ago", text: "The switch statement for this enum is 3,200 lines long and every case returns a different RACSignal. Xcode autocomplete just shows '...' when I type 'AppState.' It's given up." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "2 days ago", text: "I added a new case to the enum and got 847 compiler errors because the exhaustive switch statement in 23 files needed updating. 'Exhaustive' is doing a lot of work in that sentence." }
---

<p>I've spent the last six months at the intersection of two ideas that most people think are unrelated: enums and reactive programming. Most developers see enums as simple value types — a list of options, a state machine, a way to avoid string constants. And most developers see ReactiveCocoa as a way to handle asynchronous events with signals and subscribers.</p>

<p>What if they're the same thing?</p>

<p>What if every application state is an enum case, every state transition is a signal, and your entire app is a single reactive enum that emits its own state changes? What if the enum <em>is</em> the architecture?</p>

<p>I've built this. It's called <strong>ReactiveEnum</strong>, and it's either the future of iOS development or a cautionary tale. Possibly both.</p>

<blockquote>"An enum with enough cases can model any system. A reactive enum with enough cases can model any system that also notifies you when it changes. This is either profound or tautological and I've stopped trying to tell the difference."</blockquote>

<h2>The Core Idea</h2>

<p>In traditional iOS, app state is scattered across dozens of objects. The user's auth state is in one singleton. The current view controller is another. Network reachability is a third. You end up with a web of KVO observations, delegate callbacks, and notification center subscriptions trying to keep everything in sync.</p>

<p>ReactiveEnum replaces all of this with a single enum:</p>

<pre data-lang="Objective-C"><span class="kw">typedef NS_ENUM</span>(<span class="type">NSInteger</span>, <span class="type">AppState</span>) {
    AppStateLaunching,
    AppStateUnauthenticated,
    AppStateAuthenticating,
    AppStateAuthenticated,
    AppStateAuthenticatedProfileLoading,
    AppStateAuthenticatedProfileLoaded,
    AppStateAuthenticatedProfileLoadedSettingsOpen,
    AppStateAuthenticatedProfileLoadedSettingsOpenNotifications,
    AppStateAuthenticatedProfileLoadedSettingsOpenNotificationsEnabled,
    AppStateAuthenticatedProfileLoadedSettingsOpenNotificationsDisabled,
    AppStateAuthenticatedProfileLoadedSettingsOpenDarkMode,
    AppStateAuthenticatedProfileLoadedSettingsOpenDarkModeOn,
    AppStateAuthenticatedProfileLoadedSettingsOpenDarkModeOff,
    <span class="cm">// ... 834 more cases ...</span>
    AppStateError,
    AppStateErrorRecoverable,
    AppStateErrorUnrecoverable,
    AppStateErrorUnrecoverableButWeTriedAnyway,
};</pre>

<p>Every possible state the app can be in has its own enum case. There is no ambiguity. There is no "what state are we in?" The enum tells you. It tells you with 847 cases, but it tells you.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>Exhaustive switch statements are the enforcement mechanism of ReactiveEnum. The compiler won't let you handle 846 out of 847 cases. You <em>must</em> handle every state. This is either the greatest type safety feature in iOS development or a form of punishment, depending on your perspective.</p>
</div>

<h2>The Reactive Layer</h2>

<p>Each state transition emits a <code>RACSignal</code>. When the app moves from <code>AppStateUnauthenticated</code> to <code>AppStateAuthenticating</code>, a signal fires. Every part of the app that cares about this transition subscribes to it. The login screen listens for <code>AppStateAuthenticated</code>. The profile screen listens for <code>AppStateAuthenticatedProfileLoaded</code>. The settings screen listens for any state with "Settings" in the name.</p>

<pre data-lang="Objective-C"><span class="cm">// Subscribe to state changes</span>
[[<span class="type">ReactiveEnum</span> <span class="fn">sharedInstance</span>].<span class="fn">stateSignal</span>
    <span class="fn">filter</span>:^<span class="type">BOOL</span>(<span class="type">NSNumber</span> *state) {
        <span class="cm">// Match any "Authenticated" state (cases 3-844)</span>
        <span class="kw">return</span> state.<span class="fn">integerValue</span> >= <span class="num">3</span> 
            && state.<span class="fn">integerValue</span> <= <span class="num">844</span>;
    }]
    <span class="fn">subscribeNext</span>:^(<span class="type">NSNumber</span> *state) {
        <span class="cm">// We're authenticated! Which sub-state? Check the number.</span>
        <span class="cm">// I hope you memorized the enum.</span>
    }];
</pre>

<p>The filter block checks if the state integer is between 3 and 844 because those are the "Authenticated" states. Yes, we're filtering by magic numbers in a system designed to eliminate magic numbers. The irony is not lost on me. It's lost on the compiler, which is fine with it.</p>

<h2>The State Explosion</h2>

<p>The mathematical reality of ReactiveEnum is: every boolean in your app doubles the number of enum cases. Dark mode on/off? Double. Notifications enabled/disabled? Double again. Onboarding complete/incomplete? Double once more. Ten booleans means 1,024 enum cases. Twenty booleans means 1,048,576 cases.</p>

<p>We currently model 10 independent boolean states as nested enum paths, giving us approximately 1,024 leaf states. We don't model them as a flat enum (that would be insane — 1,024 top-level cases). Instead, we use nested enums that compose hierarchically. This limits any single switch statement to about 20 cases. The nesting depth, however, can reach 14 levels.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The Checkout Flow</span>
  <p>The checkout flow has a state path that is 14 levels deep: <code>.authenticated(.shopping(.cartFilled(.checkoutStarted(.addressEntered(.shippingSelected(.paymentEntered(.cardValidated(.threeDSecure(.challenged(.completed(.receiptShown(.surveyPrompted(.done))))))))))))))</code>. The switch statement for this path is 3,200 lines. Adding a "gift wrapping" option would add 14 new cases and require updating every switch statement in the checkout module. We have declined to add gift wrapping.</p>
</div>

<h2>Adding a New Feature</h2>

<p>When a product manager asks "can we add a feature toggle?", the engineering answer used to be "sure, it's a boolean." In ReactiveEnum, a boolean doubles the state space. Adding a feature toggle means:</p>

<ol>
  <li>Add new cases to the enum hierarchy (double the leaf states in the affected subtree)</li>
  <li>Update every switch statement that handles states in that subtree</li>
  <li>Add signal subscriptions for the new state transitions</li>
  <li>Write tests for every new case (we have one test per case; adding 40 cases means 40 tests)</li>
  <li>Update the state diagram, which is now a poster-sized printout on the wall that we've run out of room for</li>
</ol>

<p>Average time to add a feature toggle: <strong>3 days</strong>. In a normal architecture: 20 minutes. But our toggle is <em>type-safe</em>. You cannot access a feature in a state where the toggle is off. The compiler physically prevents it. Is that worth 3 days? We've chosen to believe yes.</p>

<h2>Results</h2>

<ul>
  <li>Total enum cases (all levels): <strong>~1,024</strong></li>
  <li>Maximum nesting depth: <strong>14 levels</strong></li>
  <li>Longest switch statement: <strong>3,200 lines</strong></li>
  <li>Compiler errors when adding one case: <strong>average 47</strong> (one per switch statement that doesn't handle it)</li>
  <li>Runtime state bugs found in production: <strong>0</strong> (genuinely — the compiler catches everything)</li>
  <li>Time to add a boolean: <strong>3 days</strong></li>
  <li>Time to understand the full state tree: <strong>2 weeks for new engineers, ongoing for existing ones</strong></li>
  <li>Engineers who think this is a good idea: <strong>1</strong> (me)</li>
  <li>Production crashes caused by invalid state: <strong>0</strong></li>
  <li>Production crashes caused by the state system itself: <strong>also 0</strong></li>
</ul>

<p>Zero runtime state bugs. Zero invalid state crashes. ReactiveEnum is the most type-safe architecture I've ever built. It's also the most verbose, the most time-consuming to extend, and the most likely to make a new hire cry during onboarding. But zero state bugs. <em>Zero</em>. That number sustains me through the 3,200-line switch statements. Mostly.</p>
