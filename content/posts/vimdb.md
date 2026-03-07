---
title: "VimDB: The Database You Exit When You're Done Querying"
subtitle: "All operations are modal. Reads happen in NORMAL mode. Writes happen in INSERT mode. Deletes happen when you press dd on the wrong line."
date: 2015-02-22
categories: ["Database"]
tags: ["databases", "vim", "storage", "modal-computing", "hubris"]
author: "Gordon Ashby"
authorHandle: "@gashby_data"
authorInitials: "GA"
authorColor: "#1a5276"
readTime: "10 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "5 days ago", text: "Gordon I ran a benchmark and VimDB is actually faster than SQLite for single-row reads. I don't know what to do with this information." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "5 days ago", text: "@bradkow_ios I know. I've been sitting with this for three weeks. It keeps me up at night." }
  - { initials: "PM", color: "#8e44ad", name: "priya_api_first", time: "4 days ago", text: "The fact that your transaction model is 'undo with u' is either the worst or best thing in database theory." }
  - { initials: "EB", color: "#1b2631", name: "ebaxter_dev", time: "3 days ago", text: "I accidentally typed :wq in my Postgres terminal and honestly, it should have worked." }
---

<p>After the qualified success of <a href="/cocoapatterns/posts/regexdb/">RegexDB</a>, several readers asked if I had plans for a follow-up. "Gordon," they wrote, "you've shown us that a database can be powered by regex. What other tools can be inappropriately repurposed as storage engines?"</p>

<p>The answer, it turns out, was in front of me the whole time. I was writing the RegexDB post in Vim. And I thought: Vim already has buffers. Buffers hold data. Data is what databases store. I'm already inside a database. I just need to formalize it.</p>

<blockquote>"Every text editor is a database if you're brave enough and your data model is 'lines of text.'"</blockquote>

<h2>Core Concepts</h2>

<p>VimDB is a modal database. This is its defining innovation and its most controversial feature. In traditional databases, you can read and write in the same operation. In VimDB, you must explicitly switch modes:</p>

<ul>
  <li><strong>NORMAL mode:</strong> Read-only queries. Navigate data with <code>h</code>, <code>j</code>, <code>k</code>, <code>l</code>. Search with <code>/pattern</code>.</li>
  <li><strong>INSERT mode:</strong> Press <code>i</code> to enter write mode. All mutations happen here. Press <code>Esc</code> to commit and return to NORMAL.</li>
  <li><strong>VISUAL mode:</strong> Select ranges of rows for bulk operations. This is our version of a <code>SELECT ... WHERE</code> clause.</li>
  <li><strong>COMMAND mode:</strong> Administrative operations. <code>:w</code> flushes to disk. <code>:q</code> terminates the connection. <code>:q!</code> terminates without saving, which we're calling a "forced rollback."</li>
</ul>

<pre data-lang="VimQL"><span class="cm">" Connect to VimDB</span>
$ vim /var/data/vimdb/users.vdb

<span class="cm">" Query: find all users named Brad</span>
/Brad
<span class="cm">" Press n to iterate through results (this is our cursor)</span>
n
n
n

<span class="cm">" Insert a new user</span>
o
Brad Kowalski | brad@example.com | 2014-01-15
<span class="kw">&lt;Esc&gt;</span>

<span class="cm">" Commit to disk</span>
:w

<span class="cm">" Close connection</span>
:q</pre>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>In VimDB, the cursor <em>is</em> the query pointer. Your physical position in the file is your position in the result set. If you get lost, that's a you problem.</p>
</div>

<h2>The Data Format</h2>

<p>VimDB stores records as pipe-delimited lines in a plain text file. There is no schema. There is no header row. You are expected to remember what each column means. If you forget, that's between you and your <code>.vimrc</code>.</p>

<p>We considered adding column headers, but column headers are just data that thinks it's special, and VimDB doesn't support class hierarchies.</p>

<h2>Transactions and ACID Compliance</h2>

<p>VimDB provides a unique transaction model based on Vim's built-in undo tree:</p>

<ul>
  <li><strong>Atomicity:</strong> Each keystroke is atomic. A multi-character insert is a sequence of atomic operations. If your finger slips, only the slipped character is wrong. The rest is fine.</li>
  <li><strong>Consistency:</strong> The file is always valid UTF-8. That's our consistency guarantee. That's the whole thing.</li>
  <li><strong>Isolation:</strong> If two people open the same <code>.vdb</code> file, Vim shows a swap file warning. This is our lock manager. First person to press <code>(E)dit</code> wins. Concurrency resolved.</li>
  <li><strong>Durability:</strong> <code>:w</code> writes to disk. If you forget to <code>:w</code>, your data is gone. We consider this a feature. It teaches discipline.</li>
</ul>

<div class="callout danger">
  <span class="callout-label">⚠️ Known Issue</span>
  <p>One of our beta testers pressed <code>dd</code> (delete line) instead of <code>jj</code> (move down). They deleted a production user record. We told them to press <code>u</code> to undo. They had already pressed <code>:w</code>. There is no <code>u</code> after <code>:w</code>. We're calling this "eventual consistency with user error."</p>
</div>

<h2>Advanced Features</h2>

<h3>Macros as Stored Procedures</h3>

<p>Vim macros (<code>q{register}</code>) serve as VimDB's stored procedure system. Record a sequence of operations, save it to a register, and replay it with <code>@{register}</code>. Our most complex stored procedure — a user deduplication routine — is stored in register <code>z</code> and is 847 keystrokes long. Nobody remembers what it does. We just run <code>@z</code> and check if anything broke.</p>

<h3>Replication</h3>

<p>VimDB supports replication via <code>scp</code>. Every 30 seconds, a cron job copies the <code>.vdb</code> file to a backup server. If both the primary and the backup are being edited simultaneously, the last <code>scp</code> wins. We're calling this "last-write-wins replication" and several people have told us this is a real thing in distributed systems, which makes us feel better.</p>

<h2>Benchmarks</h2>

<ul>
  <li>Single-row lookup: <strong>0.003ms</strong> (if you know the line number), <strong>indeterminate</strong> (if you don't)</li>
  <li>Full table scan: <strong>depends on how fast you can press <code>n</code></strong></li>
  <li>Write throughput: <strong>limited by typing speed</strong> (our fastest DBA writes at 140 WPM)</li>
  <li>Maximum tested file size: <strong>2.1GB</strong> (Vim got slow around 800MB; we consider this the sharding threshold)</li>
  <li>Concurrent users: <strong>1</strong></li>
  <li>Users who can exit VimDB without Googling how: <strong>0</strong></li>
</ul>

<p>VimDB is available now. Our GitHub repo has 8 stars and one pull request titled "please delete this repository" which I have marked as "won't fix."</p>
