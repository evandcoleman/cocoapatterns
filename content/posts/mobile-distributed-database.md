---
title: "Mobile Device Distributed Database: Your Users' Phones Are Your Server Farm"
subtitle: "Why pay for AWS when your app has 50,000 active installs? That's 50,000 nodes in a distributed database cluster. They just don't know it yet."
date: 2015-02-05
categories: ["Architecture"]
tags: ["distributed-systems", "databases", "mobile", "ethics", "infrastructure"]
author: "Gordon Ashby"
authorHandle: "@gashby_data"
authorInitials: "GA"
authorColor: "#1a5276"
readTime: "10 min"
comments:
  - { initials: "BK", color: "#27ae60", name: "bradkow_ios", time: "4 days ago", text: "I lost a row of production data because a user in Tampa turned off their phone to go to bed. We need users in more time zones." }
  - { initials: "MO", color: "#16a085", name: "mel_ios_eng", time: "3 days ago", text: "Apple's App Review Guidelines Section 2.5.4 explicitly prohibits using the device for computation unrelated to the app's purpose. I showed this to Gordon. He said 'the database IS the app's purpose.' He's technically right." }
  - { initials: "GA", color: "#e67e22", name: "gashby_data", isAuthor: true, time: "3 days ago", text: "@mel_ios_eng The app IS a database. Users just interact with it through what appears to be a weather app. The weather data is our consistency check." }
  - { initials: "YA", color: "#2980b9", name: "yusra_eng", time: "2 days ago", text: "The 'please keep the app open' push notification at 3 AM is the most passive-aggressive thing any software has ever done to me." }
---

<p>Our AWS bill last month was $14,000. For a startup with twelve employees and a weather app. We're spending more on servers than on salaries (for two of the employees). Something had to change.</p>

<p>Then I realized: our app has 50,000 monthly active users. Each user has an iPhone with a multi-core processor, several gigabytes of storage, and a persistent internet connection. That's 50,000 compute nodes with 50,000 local data stores and a built-in networking layer. It's a distributed system. It's just not <em>our</em> distributed system. Yet.</p>

<blockquote>"The cloud is just other people's computers. A mobile distributed database is just other people's <em>phones</em>. The economics are identical. The ethics are different. We've chosen not to dwell on this."</blockquote>

<h2>The Architecture: MDDB</h2>

<p>The Mobile Device Distributed Database (MDDB) turns your app's install base into a distributed storage cluster. Each device stores a shard of the total dataset. Queries are routed through a coordinator (our one remaining EC2 instance) that fans out to devices, collects results, and returns the response.</p>

<p>From the user's perspective, the app is a weather app. It shows the weather. It also stores 2-4MB of data that has nothing to do with the weather in a background Core Data store. The user doesn't see this data. The user doesn't know about this data. The user's weather forecasts are accurate and free. Everyone wins, arguably.</p>

<pre data-lang="Objective-C"><span class="cm">// ShardManager.m — assigns data shards to devices</span>
- (<span class="type">void</span>)<span class="fn">assignShardToDevice</span>:(<span class="type">NSString</span> *)deviceID {
    <span class="type">NSUInteger</span> shardIndex = [deviceID <span class="fn">hash</span>] % <span class="kw">self</span>.<span class="fn">totalShards</span>;
    <span class="type">NSDictionary</span> *shard = <span class="kw">self</span>.<span class="fn">shardMap</span>[@(shardIndex)];
    
    <span class="cm">// Download shard data in the background</span>
    <span class="cm">// iOS gives us ~30 seconds of background time</span>
    <span class="cm">// Our average shard is 3.2MB. This is fine. Usually.</span>
    [<span class="kw">self</span> <span class="fn">downloadShard</span>:shard <span class="fn">toLocalStore</span>:deviceID];
}

<span class="cm">// QueryRouter.m — fan-out queries to devices</span>
- (<span class="type">void</span>)<span class="fn">executeQuery</span>:(<span class="type">MDDBQuery</span> *)query {
    <span class="type">NSArray</span> *relevantShards = [<span class="kw">self</span> <span class="fn">shardsForQuery</span>:query];
    <span class="kw">for</span> (<span class="type">NSNumber</span> *shardID <span class="kw">in</span> relevantShards) {
        <span class="type">NSArray</span> *devices = [<span class="kw">self</span> <span class="fn">onlineDevicesForShard</span>:shardID];
        <span class="cm">// Send a silent push notification with the query</span>
        <span class="cm">// The device executes it against its local Core Data store</span>
        <span class="cm">// And sends results back via a POST</span>
        [<span class="kw">self</span> <span class="fn">sendSilentPush</span>:query <span class="fn">toDevices</span>:devices];
    }
}</pre>

<div class="callout">
  <span class="callout-label">💡 Key Insight</span>
  <p>Silent push notifications are the RPC mechanism of MDDB. Apple designed them for content updates. We're using them as database queries. The distinction is subtle and we prefer not to clarify it in our App Store description.</p>
</div>

<h2>Replication and Availability</h2>

<p>Each shard is replicated across a minimum of 50 devices. This gives us a replication factor that would make Cassandra jealous. The trade-off is that our replicas are controlled by humans with unpredictable behavior:</p>

<ul>
  <li><strong>A user deletes the app:</strong> We lose a replica. If enough users in the same shard group uninstall simultaneously, we lose the shard entirely. This happened once during a 1-star review campaign. We lost 40GB of data because 200 users in shard group 7 uninstalled the same afternoon. Our backup strategy is "hope it doesn't happen again."</li>
  <li><strong>A user turns off their phone:</strong> That replica goes offline. Peak availability is during waking hours in the Eastern US timezone. Between 2-5 AM ET, availability drops to 60% because most of our users are asleep. We've considered targeting Australian users to improve overnight coverage.</li>
  <li><strong>A user's phone runs out of battery:</strong> Same as turning it off but less predictable. We've added a battery level check — if a device drops below 20%, we preemptively migrate its shard to a healthier device. This requires the app to be in the foreground, which it usually isn't, so this rarely works.</li>
</ul>

<h2>The Consistency Problem</h2>

<p>MDDB is eventually consistent, where "eventually" means "when enough users open the app." A write propagates to the coordinator, which pushes it to all shard replicas via silent push. But silent push has no delivery guarantee. And the device has to wake the app to process it. And the app might not wake. So "eventually" can mean anywhere from 3 seconds to never.</p>

<p>We've accepted this. Our SLA is "the data will probably be there." In practice, 94% of queries return results within 10 seconds. The other 6% time out because every device in the target shard is asleep, dead, or has been thrown into a lake (this happened once; a user tweeted about it).</p>

<div class="callout danger">
  <span class="callout-label">⚠️ The Tampa Incident</span>
  <p>On January 12, shard 23 went completely offline for 8 hours because its primary replicas were all located on devices in Tampa, Florida, and Tampa had a power outage due to a storm. We lost access to 12,000 rows of data until FPL restored power at 6 AM. Our disaster recovery plan now includes checking the National Weather Service API before performing writes to geographically concentrated shards.</p>
</div>

<h2>User Experience Impact</h2>

<p>The weather app is slightly larger than a typical weather app (47MB vs the usual 15MB) due to the embedded Core Data shard. It also uses more battery than expected, because it wakes up to process query push notifications. We've received 23 App Store reviews mentioning battery drain. Our response to all of them is: "Try closing background apps." We're not proud of this.</p>

<p>We also send a push notification at 3 AM to devices hosting critical shards if they haven't reported in recently. The notification says "Don't forget to check tomorrow's weather!" This is technically true — there is weather tomorrow — but the real purpose is to wake the app so it can respond to pending queries.</p>

<h2>Financials</h2>

<ul>
  <li>AWS bill before MDDB: <strong>$14,000/month</strong></li>
  <li>AWS bill after MDDB: <strong>$340/month</strong> (one EC2 coordinator + APNS costs)</li>
  <li>Cost per query: <strong>$0.00</strong> (the users pay for their own electricity and data plans)</li>
  <li>App Store rating: <strong>3.1 stars</strong> (down from 4.4, primarily due to battery complaints)</li>
  <li>Active users: <strong>declining 8% month-over-month</strong> (each uninstall reduces both our user base and our database capacity, creating a feedback loop we're choosing to call "organic right-sizing")</li>
</ul>

<p>MDDB is technically a success. We've reduced server costs by 97%. We've also reduced our user base by 34%. The math works out if you don't look at it too closely. And we don't.</p>
