---
title: "2016 Steam Spam Groups: A Deep Dive"
date: 2026-02-23T10:30:00-06:00
draft: false
toc: true
slug: "steam-spam-groups-2016"
tags:
  - security
  - steam
  - spam
  - research
---

Months ago I fell down a rabbit hole on Steam and found something that did not look normal at all: a **huge belt of groups from 2016**, almost all of them filled with scammy Chinese descriptions and phone‑number spam.

Rough guess: **10–20 million** groups in that mess.

I sent it to Valve, a lot of the groups vanished, and I assumed that was that. Then, a few months later, I pointed my script at the same area again and… **they were back**. Not every single group, but the same patterns, same template text, same numbers.

Below is a breakdown of what I found, how I scanned it, and why I think it matters.

---

## When and where this happened

Steam gives every group a numeric **group ID (GID)**. If you crawl those IDs in order you basically get a timeline.

For this spam campaign the timeline looks like this:

- **Creation window:** roughly **6 March 2016 🠞 10 May 2016**
- **Start of the cluster:** around **GID 12,200,000**
- **Ramp‑up:** into the **13,000,000+** range
- **Top end:** up to about **25,017,291**
- **Shape:** starts slow, ramps up extremely fast, then slowly dies off

Back then **Steam accounts were easy and cheap to spin up**:

- You could register accounts without attaching a phone number.
- Each account could create up to **10 groups**, so **1 spam account 🠞 10 junk groups**.
- If you farm enough accounts, you get millions of groups for basically no cost.

Today the picture is a bit different:

- New accounts are effectively tied to a **phone number** during signup.
- Steam also treats low‑spend accounts as **"limited"** until they've spent about **$5 USD**, which gates a lot of community features.

If you imagine a simple script that:

1. Logs in,
2. Creates a group,
3. Fills in a name / description / tag from a template,
4. Increments some counter,

…you basically get the picture of how this thing behaved.

---

## What the groups looked like

From the human side, these groups were obviously not normal communities. They read like someone glued together phrases about **drugs, fake diplomas, and random "services"** plus a phone number or some contact string.

A few examples (names, vanity URLs, tags):

---

Even without speaking Chinese you can see the pattern:

- The **name patterns** repeat the same ideas with tiny changes (where to buy X, fake degrees, how much for a fake certificate, etc.).
- The **URLs** are mostly **short, random‑looking strings**.
- The **tags** often start with the same word (`戏子`) and then hang a random suffix off the end.

Scroll through groups in GID order and you see whole runs where:

- Neighbouring groups share the same base phrase with only the number changed.
- The same phone numbers and IDs show up over and over.
- The tags have the same skeleton, just different dice rolls.

It feels less like a bunch of people spamming and more like **a printer that got left on overnight**.

---

## How I actually looked at this (SteamFinder)

I didn't do anything complicated here. I wrote a small Python script called **`SteamFinder.py`** that:

- Walks a range of group IDs,
- Fetches each group page,
- Pulls out the group name, vanity URL and tag,
- Logs everything to JSONL so I can poke at it later.

You give it a starting GID and how many groups you want to scan, and it does the rest. If it gets rate‑limited, it backs off; if it crashes, it saves state and resumes later.

Once I had that working I pointed it at the **2016 band**:

- Start around **12,200,000**
- Keep going until about **25,017,291**

I didn't fetch every single ID at first. I did:

- **Sampling** – step through with gaps to get a rough idea of what lives there.
- **Zooming in** – when I hit a dense spam patch, I tightened the step size.

From that I could sketch a rough **"GID vs creation date"** curve:

- Early March 2016: mixed bag, spam shows up but isn't dominant.
- Late March 🠞 April: spam is basically the default in that range.
- May: activity slows and eventually tails off.

For size, I just combined:

- How wide the GID band is, and
- What fraction of my samples in that band matched the spam template.

Even being careful on both, I still end up with **"tens of millions"**, and **10–20 million** feels like a fair ballpark.

---

## Why I'm confident it was automated

Some of the things that convinced me:

- **Sequential GIDs:** huge stretches where almost every ID in a row is a spam group.
- **Tiny vocabulary:** same small set of phrases over and over, with numbers tacked on.
- **Copy‑paste structure:** name / URL / tag combos that only differ by one chunk.
- **Throughput shape:** classic "spin up 🠞 hammer it 🠞 turn it off" behaviour.

You could in theory do this by hand, but you'd be doing nothing else for years. Everything about it smells like **scripts pushing the "create group" button as fast as Steam would tolerate in 2016**.

---

## Cleanup, then déjà vu

After I'd played with this for a while I reported it to Valve.

Over time, a lot of the GIDs I had in my logs:

- Started returning error pages, or
- Stopped resolving to real groups at all.

So for a while it looked like Steam had done a **big sweep across that 2016 belt** and taken out a lot of the junk.

Fast forward a few months. I dusted off SteamFinder, pointed it back at the same ranges, and things got weird again:

- A bunch of the patterns I'd seen before were back.
- Some GIDs I had written down as "spam group X, later removed" were once again real groups.
- New groups existed that looked like **clones** of the old ones – same templates, same style, living in the same time band.

From the outside I can't say what exactly happened:

- Maybe some moderation actions were **rolled back**.
- Maybe the groups were only **soft‑hidden** for a while.
- Maybe new spam runs **re‑used the old templates**.

What I can say is that **the footprint of the campaign never really went away**. The same junk kept bubbling back up in slightly different forms.

---

## Why I think it's interesting

It's easy to shrug and say "who cares, they're just old Steam groups," but at this scale it adds up:

- **Noise:** 10–20 million junk groups is a lot of garbage in search and discovery.
- **Abuse surface:** every group is another place to drop links, numbers, scams, whatever.
- **History:** this lived on the platform for **years** before it got any real attention.

It's also a neat example of how **low‑friction systems get abused**:

- In 2016, cheap accounts with no phone + no spend floor 🠞 easy to scale this kind of spam.
- Little visibility into bulk behaviour 🠞 you don't notice an entire slice of time is basically one spam campaign.
- Cleanups that don't fully stick 🠞 the same patterns come back later.

---

## If you want to poke at it yourself

If you enjoy this kind of thing, the 2016 cluster is a good playground:

- Treat the GID range as a **fossil layer** and sample it yourself.
- Look at how the **templates drift** as you move through time.
- Graph the **ramp‑up and tapering** and compare it to other spammy things you find.

If you're building or running a platform, the advice is boring but important:

- Make it **more expensive** to farm accounts and objects (phone verification, rate limits, etc.).
- Make it **easy to see** when a whole area of the system suddenly fills up with clones.
- When you clean something like this up, try to make sure you don't quietly **grow the same problem back** a year later.

I don't have internal logs or special access here – just a script, some patience, and a lot of JSON. But even that was enough to see that **2016 on Steam had a giant spam bloom**, and the story didn't stop the first time the delete button got pressed.
