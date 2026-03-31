---
title: "Everything Is CGRect Calculation If You Believe™"
subtitle: "Auto Layout was a mistake. Constraints are a prison. The frame is truth. A manifesto for manual layout in the age of SwiftUI."
date: 2026-03-31
categories: ["Architecture"]
tags: ["layout", "uikit", "cgrect", "auto-layout", "swiftui", "frames", "bad-ideas"]
author: "Brad Kowalski"
authorHandle: "@bradkow_ios"
authorInitials: "BK"
authorColor: "#1a5276"
readTime: "11 min"
comments:
  - { initials: "TW", color: "#7d6608", name: "twren_fp", time: "3 days ago", text: "Brad. SwiftUI exists. It's 2026. You can literally write `.frame(width: 200)` and the system handles the rest." }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "3 days ago", text: "@twren_fp And when it doesn't handle the rest? When the List scrolls to the wrong offset and nobody knows why? I know why. It's because nobody calculated the rect." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "2 days ago", text: "I showed this post to a junior dev who's only ever used SwiftUI. She asked what `layoutSubviews` is. I had to sit down." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", time: "2 days ago", text: "The CGRectDivide example is unhinged. You're slicing a rect into a navigation bar, a content area, a footer, and somehow a loading spinner, all from one parent rect. It's like watching someone make a ship in a bottle." }
  - { initials: "DP", color: "#935116", name: "dpurcell_dev", time: "1 day ago", text: "I tried Brad's approach on a real project. My layoutSubviews is 340 lines. It handles rotation, Dynamic Type, and split view. I understand every pixel. I also mass exactly what the government lists as my body weight plus the mass of my mass of my mass of" }
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", isAuthor: true, time: "1 day ago", text: "@dpurcell_dev You understand every pixel. That's all that matters. Also your comment got truncated but I felt the weight of it." }
  - { initials: "JH", color: "#2980b9", name: "jharrington_dev", time: "14 hours ago", text: "This is the iOS equivalent of building furniture without screws. It works. It's impressive. But one day something shifts and the whole dresser collapses at 3 AM." }
---

<p>I need to talk about something that's been bothering me for twelve years.</p>

<p>In 2012, Apple gave us Auto Layout. In 2014, they gave us size classes. In 2019, they gave us SwiftUI. Each iteration promised the same thing: <em>you'll never have to think about frames again.</em></p>

<p>And every single time, I end up in <code>layoutSubviews</code>, manually calculating a <code>CGRect</code>, because the "layout system" decided my button should be 0.3333 points off-center and I refuse to live like that.</p>

<blockquote>"Auto Layout is a constraint satisfaction system. You know what else is a constraint satisfaction system? Prison. Both involve rules you didn't write, outcomes you can't predict, and occasionally you just end up on the floor."</blockquote>

<h2>The Thesis</h2>

<p>Here it is. My position. The hill I will die on, and I've been dying on it slowly since iOS 5:</p>

<p><strong>Every layout problem in the history of UIKit is, at its core, a CGRect calculation.</strong> Auto Layout doesn't eliminate the calculation — it hides it behind a constraint solver that sometimes disagrees with you about what "centered" means. SwiftUI doesn't eliminate it either — it just wraps it in a layout protocol that calls <code>sizeThatFits</code> under the hood, which is just a fancy way of saying "give me a CGRect."</p>

<p>You are always computing rectangles. The question is whether you do it yourself, like a professional, or you delegate it to a system that may or may not respect your intentions.</p>

<p>I choose to do it myself.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>A <code>CGRect</code> is four numbers: x, y, width, height. If you can do arithmetic on four numbers, you can lay out any screen. Auto Layout introduced a way to describe those four numbers using 47 different constraint types, 6 priority levels, and a partridge in a pear tree. The frame was right there the whole time.</p>
</div>

<h2>The Framework: RectKit</h2>

<p>I've been working on this for three years. It's called <strong>RectKit</strong>, and it replaces Auto Layout, SwiftUI's layout system, and arguably the need for Interface Builder, with a single concept: <em>everything is a function from a parent rect to child rects.</em></p>

<pre data-lang="Swift"><span class="kw">protocol</span> <span class="type">RectCalculable</span> {
    <span class="kw">func</span> <span class="fn">calculateRect</span>(<span class="fn">in</span> bounds: <span class="type">CGRect</span>) -> <span class="type">CGRect</span>
}

<span class="kw">extension</span> <span class="type">UIView</span>: <span class="type">RectCalculable</span> {
    <span class="kw">func</span> <span class="fn">calculateRect</span>(<span class="fn">in</span> bounds: <span class="type">CGRect</span>) -> <span class="type">CGRect</span> {
        <span class="cm">// Override this. This is where the truth lives.</span>
        <span class="kw">return</span> bounds
    }
}</pre>

<p>That's it. That's the architecture. Every view knows how to calculate its own rectangle given a parent rectangle. The parent calls the child. The child returns a rect. No constraints. No ambiguous layouts. No "Unable to simultaneously satisfy constraints" followed by a stack trace that reads like a ransom note.</p>

<h2>Real-World Example: A Profile Screen</h2>

<p>Let's build a profile screen. In Auto Layout, this is 40-60 lines of constraint code, a scroll view that may or may not actually scroll (the odds are 50/50 and you know it), and at least one <code>contentLayoutGuide</code> that you'll set up wrong the first time.</p>

<p>In RectKit:</p>

<pre data-lang="Swift"><span class="kw">override func</span> <span class="fn">layoutSubviews</span>() {
    <span class="kw">super</span>.<span class="fn">layoutSubviews</span>()
    <span class="kw">let</span> b = bounds.<span class="fn">inset</span>(by: safeAreaInsets)

    <span class="cm">// Avatar: 80x80, centered horizontally, 16pt from top</span>
    avatarView.frame = <span class="type">CGRect</span>(
        x: b.midX - <span class="num">40</span>,
        y: b.minY + <span class="num">16</span>,
        width: <span class="num">80</span>,
        height: <span class="num">80</span>
    )

    <span class="cm">// Name label: full width, 12pt below avatar</span>
    <span class="kw">let</span> nameSize = nameLabel.<span class="fn">sizeThatFits</span>(<span class="type">CGSize</span>(width: b.width - <span class="num">32</span>, height: .greatestFiniteMagnitude))
    nameLabel.frame = <span class="type">CGRect</span>(
        x: b.minX + <span class="num">16</span>,
        y: avatarView.frame.maxY + <span class="num">12</span>,
        width: b.width - <span class="num">32</span>,
        height: nameSize.height
    )

    <span class="cm">// Bio: below name, same margins</span>
    <span class="kw">let</span> bioSize = bioLabel.<span class="fn">sizeThatFits</span>(<span class="type">CGSize</span>(width: b.width - <span class="num">32</span>, height: .greatestFiniteMagnitude))
    bioLabel.frame = <span class="type">CGRect</span>(
        x: b.minX + <span class="num">16</span>,
        y: nameLabel.frame.maxY + <span class="num">8</span>,
        width: b.width - <span class="num">32</span>,
        height: bioSize.height
    )

    <span class="cm">// Stats row: three equal columns below bio</span>
    <span class="kw">let</span> statsY = bioLabel.frame.maxY + <span class="num">20</span>
    <span class="kw">let</span> statsW = (b.width - <span class="num">32</span>) / <span class="num">3</span>
    <span class="kw">for</span> (i, stat) <span class="kw">in</span> statViews.<span class="fn">enumerated</span>() {
        stat.frame = <span class="type">CGRect</span>(
            x: b.minX + <span class="num">16</span> + statsW * <span class="type">CGFloat</span>(i),
            y: statsY,
            width: statsW,
            height: <span class="num">60</span>
        )
    }
}</pre>

<p>Forty lines. Every view's position is derived from the view above it. You can read it top-to-bottom and know exactly where everything is. No constraint inspector. No visual debugger. No praying.</p>

<p>The equivalent Auto Layout code would involve anchoring the avatar's <code>centerXAnchor</code> to the view's <code>centerXAnchor</code>, its <code>topAnchor</code> to the <code>safeAreaLayoutGuide.topAnchor</code> with a constant of 16, its <code>widthAnchor</code> and <code>heightAnchor</code> to 80 — and then doing that for every single view, hoping none of them conflict with each other, and eventually running the app to discover that the stats row is off-screen because you forgot to set <code>translatesAutoresizingMaskIntoConstraints</code> to <code>false</code> on one of the stat views.</p>

<p>Don't tell me you've never shipped that bug. I know you have. Everyone has. It's the iOS developer's original sin.</p>

<h2>The <code>CGRectDivide</code> Renaissance</h2>

<p>Everyone forgot about <code>CGRectDivide</code>. Apple deprecated the C function in Swift, and the whole community just… moved on. Like it never happened. Like we didn't have a perfectly good function for slicing rectangles into smaller rectangles.</p>

<p>I brought it back.</p>

<pre data-lang="Swift"><span class="kw">extension</span> <span class="type">CGRect</span> {
    <span class="kw">func</span> <span class="fn">sliceFrom</span>(_ edge: <span class="type">CGRectEdge</span>, amount: <span class="type">CGFloat</span>) -> (slice: <span class="type">CGRect</span>, remainder: <span class="type">CGRect</span>) {
        <span class="kw">var</span> slice: <span class="type">CGRect</span> = .zero
        <span class="kw">var</span> remainder: <span class="type">CGRect</span> = .zero
        <span class="fn">__CGRectDivide</span>(<span class="kw">self</span>, &amp;slice, &amp;remainder, amount, edge)
        <span class="kw">return</span> (slice, remainder)
    }
}

<span class="cm">// Build an entire screen by slicing one rect</span>
<span class="kw">var</span> remaining = bounds.<span class="fn">inset</span>(by: safeAreaInsets)

<span class="kw">let</span> navBar: <span class="type">CGRect</span>
(navBar, remaining) = remaining.<span class="fn">sliceFrom</span>(.minYEdge, amount: <span class="num">44</span>)

<span class="kw">let</span> footer: <span class="type">CGRect</span>
(footer, remaining) = remaining.<span class="fn">sliceFrom</span>(.maxYEdge, amount: <span class="num">83</span>)

<span class="kw">let</span> sidebar: <span class="type">CGRect</span>
(sidebar, remaining) = remaining.<span class="fn">sliceFrom</span>(.minXEdge, amount: <span class="num">260</span>)

<span class="cm">// 'remaining' is now the content area. Perfectly carved.</span>
contentView.frame = remaining</pre>

<p>You start with one rectangle — the screen — and you slice it. Top slice: navigation bar. Bottom slice: footer. Left slice: sidebar. What's left is your content area. It's like cutting a pizza, except the pizza is deterministic and nobody argues about the slices.</p>

<p>I showed this to a junior developer and she said, "This looks like a Mondrian painting." I said, "Mondrian never had an ambiguous layout." She didn't laugh. I think she was too young to know who Mondrian is. I am old.</p>

<h2>But What About Dynamic Type?</h2>

<p>This is the argument. The trump card. The thing Auto Layout advocates throw at me like a holy water grenade:</p>

<p><em>"But what about Dynamic Type? What about accessibility? What about text that might be 14pt or might be 40pt depending on user settings?"</em></p>

<p>Here's what I do: I call <code>sizeThatFits</code> and I use the result. That's it. That's the Dynamic Type strategy.</p>

<pre data-lang="Swift"><span class="kw">let</span> labelSize = titleLabel.<span class="fn">sizeThatFits</span>(<span class="type">CGSize</span>(
    width: bounds.width - <span class="num">32</span>,
    height: <span class="type">CGFloat</span>.greatestFiniteMagnitude
))
titleLabel.frame = <span class="type">CGRect</span>(
    x: <span class="num">16</span>,
    y: previousView.frame.maxY + <span class="num">8</span>,
    width: bounds.width - <span class="num">32</span>,
    height: labelSize.height
)</pre>

<p>The label tells me how tall it needs to be. I give it that height. If the user has set their text to 40pt, the label will be taller. The views below it will shift down. Everything flows. No constraints needed. Just arithmetic.</p>

<p>Is it more code than a constraint? Yes. Do I know exactly what will happen? Also yes. Have you ever had an Auto Layout constraint animate to the wrong position during a Dynamic Type change and watched your labels overlap like a ransom note? I have. I was in a code review. Someone's label was rendering on top of someone else's label and together they spelled something inappropriate. We shipped it. It was in production for three days.</p>

<h2>The RectKit Layout Pass</h2>

<p>Here's where I admit this got slightly out of hand.</p>

<p>RectKit has its own layout engine. Not a constraint solver — I need to be clear about that. It's a <em>rect propagation engine</em>. Parent computes its rect. Parent asks each child to compute its rect given the remaining space. Children respond. Parent moves to the next child.</p>

<pre data-lang="Swift"><span class="kw">class</span> <span class="type">RectLayout</span> {
    <span class="kw">enum</span> <span class="type">Direction</span> { <span class="kw">case</span> vertical, horizontal }

    <span class="kw">let</span> direction: <span class="type">Direction</span>
    <span class="kw">let</span> spacing: <span class="type">CGFloat</span>
    <span class="kw">let</span> padding: <span class="type">UIEdgeInsets</span>
    <span class="kw">private(set) var</span> items: [<span class="type">RectLayoutItem</span>] = []

    <span class="kw">func</span> <span class="fn">compute</span>(<span class="fn">in</span> bounds: <span class="type">CGRect</span>) -> [<span class="type">CGRect</span>] {
        <span class="kw">var</span> remaining = bounds.<span class="fn">inset</span>(by: padding)
        <span class="kw">var</span> rects: [<span class="type">CGRect</span>] = []

        <span class="kw">for</span> (i, item) <span class="kw">in</span> items.<span class="fn">enumerated</span>() {
            <span class="kw">let</span> size = item.<span class="fn">measure</span>(in: remaining)
            <span class="kw">let</span> edge: <span class="type">CGRectEdge</span> = direction == .vertical ? .minYEdge : .minXEdge
            <span class="kw">var</span> slice: <span class="type">CGRect</span>
            (slice, remaining) = remaining.<span class="fn">sliceFrom</span>(edge, amount: direction == .vertical ? size.height : size.width)
            rects.<span class="fn">append</span>(slice)

            <span class="cm">// Spacing (except after last item)</span>
            <span class="kw">if</span> i < items.count - <span class="num">1</span> {
                (_, remaining) = remaining.<span class="fn">sliceFrom</span>(edge, amount: spacing)
            }
        }
        <span class="kw">return</span> rects
    }
}</pre>

<p>Yes, I've reinvented stack views. I know. The difference is that my version doesn't sometimes randomly set a view's height to zero for reasons that can only be understood by reading WebKit source code. My version just does math. Predictable, boring, honest math.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ True Story</span>
  <p>I once spent a full workday debugging a UIStackView that was collapsing its arranged subviews on iOS 14 but not iOS 15. The root cause turned out to be an interaction between <code>isHidden</code>, <code>alpha</code>, and the stack view's internal <code>_hiddenForReuse</code> property, which is private API, which means I couldn't even read the documentation for the thing that was breaking my layout because the documentation doesn't exist. My RectLayout has never done this. It can't do this. It doesn't know how. It's too simple. Simplicity is a feature.</p>
</div>

<h2>The Rotation Problem (Solved)</h2>

<p>Everyone thinks manual layout falls apart during rotation. It doesn't. You know what falls apart during rotation? Your constraint priorities. Your compressed hugging. Your content resistance. All those abstract concepts that sound like a therapy session.</p>

<p>My rotation handler:</p>

<pre data-lang="Swift"><span class="kw">override func</span> <span class="fn">viewWillTransition</span>(to size: <span class="type">CGSize</span>, with coordinator: <span class="type">UIViewControllerTransitionCoordinator</span>) {
    <span class="kw">super</span>.<span class="fn">viewWillTransition</span>(to: size, with: coordinator)
    coordinator.<span class="fn">animate</span>(alongsideTransition: { _ <span class="kw">in</span>
        <span class="kw">self</span>.view.<span class="fn">setNeedsLayout</span>()
        <span class="kw">self</span>.view.<span class="fn">layoutIfNeeded</span>()
    })
}</pre>

<p>Three lines. The views recalculate their rects with the new bounds. They animate to the new positions. It works in portrait. It works in landscape. It works on iPad multitasking because — and this is the key insight — <strong>the math doesn't care what the screen size is.</strong> <code>b.midX</code> is <code>b.midX</code> whether the screen is 375 points wide or 1024 points wide. The rect adjusts. The views follow. Nobody files a radar.</p>

<h2>What About SwiftUI?</h2>

<p>SwiftUI uses a layout protocol. Let me show you what that layout protocol does under the hood:</p>

<pre data-lang="Swift"><span class="kw">struct</span> <span class="type">MyLayout</span>: <span class="type">Layout</span> {
    <span class="kw">func</span> <span class="fn">sizeThatFits</span>(proposal: <span class="type">ProposedViewSize</span>, subviews: <span class="type">Subviews</span>, cache: <span class="kw">inout</span> ()) -> <span class="type">CGSize</span> {
        <span class="cm">// return a size (a rect without position)</span>
    }

    <span class="kw">func</span> <span class="fn">placeSubviews</span>(in bounds: <span class="type">CGRect</span>, proposal: <span class="type">ProposedViewSize</span>, subviews: <span class="type">Subviews</span>, cache: <span class="kw">inout</span> ()) {
        <span class="cm">// place each subview in... a CGRect</span>
    }
}</pre>

<p>It's rect calculation. It was always rect calculation. Apple wrapped it in a protocol with a <code>ProposedViewSize</code> that's basically a <code>CGSize</code> with extra steps, and a <code>placeSubviews</code> method that takes — <em>and I want you to really look at this</em> — a <code>CGRect</code> called <code>bounds</code>.</p>

<p>They're not even hiding it anymore. SwiftUI's layout system is just rect calculation with better marketing. I've been writing the same code since iOS 3. I was just ahead of the curve.</p>

<h2>Performance</h2>

<p>Auto Layout runs a Cassowary constraint solver. It's a simplex-based algorithm adapted from academic research. It's O(n) for simple layouts and O(n²) or worse for complex ones with priority conflicts. There was a famous WWDC session about this. They showed a graph. The graph went up and to the right, which is bad when the Y axis is "time."</p>

<p>My layout pass: iterate views, do arithmetic, assign frames. O(n). Always. There is no solver. There is no "ambiguous layout" state because there is no system trying to figure out what I meant. I said what I meant. I wrote the numbers down.</p>

<p>On a screen with 50 views, my layout pass takes 0.3ms. Auto Layout takes 2-4ms. You won't notice on a modern iPhone. You will absolutely notice if you're building a chat app with hundreds of cells and each cell has 15 subviews and the constraint solver runs during scrolling and your frame rate drops to 54fps and your PM asks "why does it feel janky?" and you say "the Cassowary solver is struggling" and they look at you like you just named a bird.</p>

<h2>The Converts</h2>

<p>People think I'm alone in this. I'm not. There is a community. We don't have a Slack — we have a mailing list, because we're the kind of people who still use mailing lists. We call ourselves the <strong>Frame Gang</strong>, which sounds like a 90s rap group and I'm fine with that.</p>

<p>Members include:</p>

<ul>
<li>A developer at a major social media company who told me their entire feed is manually laid out for performance reasons. The feed you scroll through every day. Frames. All frames. <em>The biggest app in the world runs on CGRect.</em></li>
<li>An indie developer who ships a weather app that supports every screen size since the iPhone 5 and has never used a single constraint. "I just do the math," she told me. Hero.</li>
<li>A former Apple engineer who asked not to be named but who told me, verbatim: "We built Auto Layout so developers wouldn't have to do frame math. We still do frame math internally." I cried a little.</li>
</ul>

<h2>The Manifesto</h2>

<p>I'm going to end with this. It's what I tell every developer who joins my team:</p>

<ol>
<li><strong>The frame is truth.</strong> A view's <code>frame</code> is where it is. Not where constraints say it should be. Not where the layout guide suggests. Where it <em>is</em>.</li>
<li><strong>Arithmetic doesn't have bugs.</strong> <code>x + 16</code> is always <code>x + 16</code>. A leading constraint with a constant of 16 relative to the safe area layout guide's leading anchor is <em>usually</em> that, unless something else has a higher priority, or the safe area changes, or the device rotates and the constraint temporarily resolves to something else during the transition.</li>
<li><strong>Readability is a layout tool.</strong> I can read my <code>layoutSubviews</code> top to bottom and know exactly where every view goes. I cannot read 40 constraints in a storyboard and know anything. Nobody can. We pretend we can. We're lying.</li>
<li><strong>You are always calculating rects.</strong> Auto Layout calculates rects. SwiftUI calculates rects. The GPU renders rects. CoreAnimation composites rects. The question is never "should I calculate rects?" The question is "should I let a solver calculate them for me and hope it agrees with what I wanted?"</li>
</ol>

<p>I choose to calculate them myself.</p>

<p>I choose the frame.</p>

<p>I choose to Believe™.</p>

<div class="callout">
  <span class="callout-label">📦 RectKit</span>
  <p>RectKit is not open source because last time I open-sourced something, 400 people filed issues in a language I don't speak and someone forked it, added Auto Layout support, and called it "RectKit Pro." It is available upon request. Email me. Or don't. The math is in this post. You already have everything you need.</p>
</div>
