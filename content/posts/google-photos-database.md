---
title: "Store User Data as a Photo and Upload It to Google Photos: Unlimited Storage, Zero Accountability"
subtitle: "Google Photos offers free unlimited storage for photos. A PNG is just bytes. Your database is just bytes. I think you see where this is going."
date: 2015-03-05
categories: ["Database"]
tags: ["databases", "google-photos", "storage", "steganography", "free-tier"]
author: "Gordon Ashby"
authorHandle: "@gashby_data"
authorInitials: "GA"
authorColor: "#1a5276"
readTime: "9 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "6 days ago", text: "Gordon, Google's free tier compresses photos above 16MP. Your database rows are being lossy-compressed. You're losing data to JPEG artifacts." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "6 days ago", text: "@bradkow_ios We store below the compression threshold. Each row is a 4MP PNG. This limits us to approximately 2KB of data per photo, but we have unlimited photos. The math works." }
  - { initials: "PM", color: "#1e8449", name: "priya_api_first", time: "5 days ago", text: "I searched 'user_847_row_12' in Google Photos and it showed me a mostly-black PNG with some colored pixels in the top-left corner. Google's AI classified it as 'Night Sky.' This is technically correct." }
  - { initials: "YA", color: "#2980b9", name: "yusra_eng", time: "4 days ago", text: "The fact that your database backup strategy is 'Google Photos is backed up by Google' is either the laziest or smartest thing I've ever heard." }
---

<p>I've built databases out of <a href="/cocoapatterns/posts/regexdb/">regex</a> and <a href="/cocoapatterns/posts/vimdb/">Vim</a>. Each time, I was told I had gone too far. Each time, I proved that the definition of "too far" is just a limitation of imagination. Today I'm going further.</p>

<p>Google Photos launched with free unlimited storage for photos up to 16 megapixels. A 16-megapixel PNG is approximately 48 million bytes. My entire production database is 2.3 gigabytes. That's roughly 48 photos.</p>

<p>I'm storing my database in Google Photos.</p>

<blockquote>"The cheapest database is someone else's photo storage. The second cheapest is a box of index cards. We considered both and Google had better uptime."</blockquote>

<h2>The Encoding Scheme</h2>

<p>Each database row is encoded as a PNG image. The process is straightforward:</p>

<ol>
  <li>Serialize the row to a byte array (JSON → UTF-8 bytes)</li>
  <li>Map each byte to an RGB pixel value (byte 0 → R, byte 1 → G, byte 2 → B)</li>
  <li>Arrange pixels in a grid to form a valid PNG</li>
  <li>Upload to Google Photos via the API</li>
  <li>Store the photo ID as the row's primary key</li>
</ol>

<pre data-lang="Python"><span class="kw">import</span> json, struct
<span class="kw">from</span> PIL <span class="kw">import</span> Image

<span class="kw">def</span> <span class="fn">row_to_png</span>(row_data, width=<span class="num">64</span>):
    <span class="str">"""Convert a database row to a PNG image."""</span>
    raw = json.dumps(row_data).encode(<span class="str">'utf-8'</span>)
    
    <span class="cm"># Pad to fill complete pixels (3 bytes per pixel)</span>
    <span class="kw">while</span> len(raw) % <span class="num">3</span> != <span class="num">0</span>:
        raw += b<span class="str">'\x00'</span>
    
    pixels = []
    <span class="kw">for</span> i <span class="kw">in</span> range(<span class="num">0</span>, len(raw), <span class="num">3</span>):
        pixels.append((raw[i], raw[i+<span class="num">1</span>], raw[i+<span class="num">2</span>]))
    
    height = (len(pixels) // width) + <span class="num">1</span>
    img = Image.new(<span class="str">'RGB'</span>, (width, height))
    img.putdata(pixels)
    <span class="kw">return</span> img

<span class="cm"># Upload: row becomes a "photo" of mostly black with</span>
<span class="cm"># some colored pixels that encode your user's email</span></pre>

<p>The resulting images are mostly black rectangles with clusters of colored pixels in the top-left corner. They look like corrupted thumbnails or extremely abstract art. Google Photos' AI classifier has variously labeled our data rows as "Night Sky," "Dark Room," "Abstract Art," and, once, "Screenshot." The "Screenshot" classification was the most accurate.</p>

<div class="callout">
  <span class="callout-label">💡 Key Principle</span>
  <p>PNG is lossless. This is critical. JPEG would introduce compression artifacts that corrupt your data. One engineer accidentally uploaded rows as JPEGs and when we decoded them, every user's email ended with garbled Unicode. We lost User 847's last name to a quantization matrix.</p>
</div>

<h2>Query Execution</h2>

<p>Querying is simple: fetch the photo from Google Photos by ID, download the PNG, decode pixels back to bytes, deserialize JSON. A single-row lookup takes approximately 800ms, which is slower than Postgres by a factor of about 10,000. But it's <em>free</em>.</p>

<p>For queries that span multiple rows (e.g., "all users in New York"), we fetch all photos in the album, decode each one, filter in memory. A full table scan of our 12,000-user table takes about 4 hours. We run these overnight. Our analytics dashboard updates once a day and we've described this to the PM as "batch processing."</p>

<h2>Albums as Tables</h2>

<p>Google Photos albums map naturally to database tables. Our schema:</p>

<ul>
  <li><strong>"Users" album:</strong> 12,000 photos (rows). Each photo is a 64×16 PNG.</li>
  <li><strong>"Orders" album:</strong> 34,000 photos. These are larger because order data includes line items. Each photo is 128×32.</li>
  <li><strong>"Sessions" album:</strong> 2.1 million photos. This album takes 11 minutes to list via the API. We paginate aggressively.</li>
  <li><strong>"Migrations" album:</strong> Contains photos of our schema changelog. These are actual photos — screenshots of our migration SQL. We didn't encode these as pixel data. We just screenshotted the SQL and uploaded the screenshots. This is the most human-readable part of our database.</li>
</ul>

<h2>Backup and Disaster Recovery</h2>

<p>Our backup strategy is "Google backs up Google Photos." This is either brilliant or catastrophically irresponsible. We've chosen to believe it's brilliant. Google's infrastructure team has more redundancy than we could ever build. Our database is stored across Google's global data center network. It's replicated across continents. It's more resilient than any database we've ever operated.</p>

<p>The risk is that Google changes the Photos API, deprecates unlimited storage, or detects that our "photos" are not photos. So far, the API has been stable. The unlimited tier has been modified (they now count storage against your Google account quota), which means our "free" database now costs $2.99/month for the 100GB Google One plan. This is still cheaper than our previous Postgres hosting.</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The Shared Album Incident</span>
  <p>A developer accidentally shared the "Users" album with their family Google Photos group. Their mother called to ask why they were sharing "thousands of pictures of nothing." We revoked sharing immediately but not before she had scrolled through approximately 200 user records encoded as abstract pixel art. She reportedly said, "I like the blue one." The blue one was a user named Bradley with an unusually long email address.</p>
</div>

<h2>Performance Characteristics</h2>

<ul>
  <li>Single-row read: <strong>800ms</strong> (mostly network latency to Google)</li>
  <li>Single-row write: <strong>1.2 seconds</strong> (encode + upload)</li>
  <li>Full table scan (12,000 rows): <strong>~4 hours</strong></li>
  <li>Storage cost: <strong>$2.99/month</strong> (Google One 100GB plan)</li>
  <li>Previous Postgres cost: <strong>$280/month</strong></li>
  <li>Cost savings: <strong>98.9%</strong></li>
  <li>Performance decrease: <strong>10,000x</strong></li>
  <li>Google Photos albums that look suspicious: <strong>all of them</strong></li>
  <li>Times Google has contacted us about our usage: <strong>0</strong> (this concerns us more than if they had)</li>
</ul>

<p>PhotoDB is in production. Our investors don't know. Our users don't know. Google probably doesn't know. The less everyone knows, the better the system works. This is, admittedly, not a great foundation for a data storage platform, but it's $2.99 a month and I sleep fine.</p>
