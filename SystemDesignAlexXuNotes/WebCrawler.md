# Designing a Web Crawler , Simple Notes

---

## 1. What is a Web Crawler?

A web crawler (aka **robot** or **spider**) is a program that automatically browses the web,
downloads pages, and follows the links on those pages to find more pages. Think of it as
a robot that starts on one page, reads it, then jumps to every link it finds , over and over.

**Basic loop:**

1. Start with a list of URLs (called **seed URLs**).
2. Download the page for each URL.
3. Extract new URLs (links) from that page.
4. Add the new URLs back into the list.
5. Repeat forever.

Sounds simple , but doing this at the scale of **billions of pages** without breaking
anything (including other people's servers) is a hard engineering problem.

### Common uses


| Use case                 | Example                                 |
| ------------------------ | --------------------------------------- |
| Search engine indexing   | Googlebot                               |
| Web archiving            | US Library of Congress, EU Web Archive  |
| Web mining (data mining) | Financial firms scraping annual reports |
| Web monitoring           | Detecting pirated/copyrighted content   |


---



## 2. What Makes a "Good" Crawler?

Four qualities to always keep in mind:

- **Scalable** : the web is huge, so crawling must be parallelized across many machines.
- **Robust** : the web is messy: broken HTML, dead servers, malicious pages, infinite loops. The crawler must not crash.
- **Polite** : don't hammer one website with too many requests too fast.
- **Extensible** : easy to add new content types later (images, PDFs, etc.) without a redesign.

---



## 3. Quick Napkin Math (Back-of-Envelope)

Say we crawl **1 billion pages/month**, average page size **500 KB**, stored for **5 years**.

- Pages per second ≈ 1,000,000,000 ÷ 30 days ÷ 24 hr ÷ 3600 s ≈ **400 pages/sec**
- Peak load ≈ 2× average ≈ **800 pages/sec**
- Storage per month ≈ 1B × 500 KB = **500 TB/month**
- Storage over 5 years ≈ 500 TB × 12 × 5 = **~30 PB**

Takeaway: this is a storage-heavy, high-throughput system. Design accordingly.

---



## 4. High-Level Architecture

Think of the crawler as a **pipeline** with these stations:

WebCrawler

### Components explained simply


| Component           | What it does                                                                                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Seed URLs**       | The starting points. Pick these smartly : by country/locality or by topic (sports, shopping, etc.)                                                            |
| **URL Frontier**    | A queue holding URLs waiting to be downloaded. This is the "to-do list."                                                                                      |
| **HTML Downloader** | Actually fetches the web page over HTTP.                                                                                                                      |
| **DNS Resolver**    | Converts a domain name (`wikipedia.org`) into an IP address, since that's needed to connect.                                                                  |
| **Content Parser**  | Validates/parses the downloaded HTML. Runs as a separate service so a bad page doesn't slow down downloading.                                                 |
| **Content Seen?**   | Checks if we've already stored this *exact content* before (even under a different URL). Uses a hash comparison instead of comparing full text : much faster. |
| **Content Storage** | Where the HTML is saved. Popular pages cached in memory; the rest lives on disk.                                                                              |
| **URL Extractor**   | Pulls all the links out of a page's HTML, converts relative links (`/about`) into full URLs.                                                                  |
| **URL Filter**      | Drops URLs we don't want : blacklisted sites, bad file types, broken links.                                                                                   |
| **URL Seen?**       | Checks if we've already queued/visited this exact URL before, to avoid loops and duplicate work. Usually implemented with a **Bloom filter** or hash table.   |
| **URL Storage**     | Keeps a record of URLs already visited.                                                                                                                       |




### Step-by-step flow

1. Seed URLs go into the URL Frontier.
2. Downloader pulls URLs off the Frontier.
3. Downloader asks the DNS Resolver for the IP, then downloads the page.
4. Content Parser checks the page isn't malformed.

5–6. "Content Seen?" checks for duplicate content : if it's a dup, throw it away.
7. If new, extract all links from the page.
8. Filter out junk/blacklisted links.
9–10. "URL Seen?" checks if we've handled this URL before.
11. If it's a genuinely new URL, add it back into the Frontier : and the loop continues.

---



## 5. Deep Dive: How to Traverse the Web (BFS vs DFS)

Imagine the web as a giant graph: pages are nodes, links are edges.

- **DFS (Depth-First Search)** : go as deep as possible down one path first. **Bad idea** : you can get lost extremely deep down one site's link chain.
- **BFS (Breadth-First Search)** : explore level by level, using a simple FIFO queue. This is the standard approach.

But plain BFS has two problems:

1. **Impoliteness** : most links on a page point back to the *same* site (e.g., Wikipedia links to Wikipedia). A naive queue will hammer that one server with tons of requests at once.
2. **No prioritization** : not all pages matter equally. A homepage probably deserves to be crawled sooner than a random forum comment.

This is why we need a smarter structure: the **URL Frontier**.

---



## 6. The URL Frontier (in detail)

The URL Frontier solves politeness and prioritization using two sub-parts:

### A. Front Queues → handle **priority**

- A **Prioritizer** scores each URL (using things like PageRank, site traffic, or update frequency).
- URLs go into different priority queues (`f1` ... `fn`).
- A **Queue Selector** picks from higher-priority queues more often (randomly, but biased).



### B. Back Queues → handle **politeness**

- A **Queue Router** makes sure each queue only holds URLs from *one* host (e.g., all `wikipedia.org` URLs go into the same queue).
- A **Mapping Table** tracks which host maps to which queue.
- Each **Worker Thread** is assigned one queue and downloads from it one URL at a time (with a delay between requests) , so no single site gets flooded.



### Freshness

Pages change over time, so the crawler needs to **re-crawl**. Since re-crawling everything is expensive:

- Prioritize re-crawling pages that update often or are high-importance.
- Use a page's update history to guess when it'll next change.



### Where is the Frontier stored?

- Hundreds of millions of URLs won't fit comfortably in memory.
- **Hybrid approach**: bulk of the queue lives on disk, but a memory buffer handles enqueue/dequeue quickly, flushing to disk periodically.

---



## 7. HTML Downloader Details



### robots.txt : the "rules of the road"

Before crawling any site, the crawler must check that site's `robots.txt` file, which lists
which pages/paths are off-limits for bots. Example (from Amazon):

```
User-agent: Googlebot
Disallow: /creatorhub/*
Disallow: /rss/people/*/reviews
```

Cache this file locally (refresh periodically) so you're not re-downloading it constantly.

### Performance tricks

- **Distributed crawling** : split the URL space across many servers/threads.
- **Cache DNS lookups** : DNS requests are slow (10–200ms) and block threads. A local DNS cache avoids repeated lookups.
- **Locality** : put crawl servers geographically close to the sites they're crawling (faster network hops).
- **Timeouts** : if a server doesn't respond quickly, give up and move on instead of waiting forever.

---



## 8. Making it Robust

- **Consistent hashing** : lets you add/remove downloader servers without reshuffling everything (see the "Consistent Hashing" chapter).
- **Save crawl state** : persist progress so a crash doesn't mean starting over.
- **Handle exceptions gracefully** : errors are guaranteed at this scale; don't let one bad page crash the whole system.
- **Validate data** : sanity-check content before storing it.

---



## 9. Making it Extensible

Design the pipeline so new content types can be **plugged in** without a rewrite —
e.g., adding a "PNG Downloader" module or a "Web Monitor" module later, without
touching the core pipeline.

---



## 10. Dealing with Bad/Junk Content


| Problem               | What it is                                                            | How to handle it                                                |
| --------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------- |
| **Duplicate content** | ~29% of pages online are duplicates                                   | Compare content **hashes**, not raw text , much faster          |
| **Spider traps**      | Pages that create infinite loops, e.g. `/foo/bar/foo/bar/foo/bar/...` | Cap max URL length; manually flag and blacklist offending sites |
| **Data noise**        | Ads, spammy links, boilerplate code snippets                          | Filter these out , they add no value                            |


---



## TL;DR Summary

> A web crawler is a **pipeline**: Frontier → Download → Parse → Dedupe → Extract Links → Filter → Dedupe URLs → back to Frontier.
> The hard parts aren't the basic loop , they're **scale** (billions of pages), **politeness** (don't DoS anyone), **priority** (crawl important stuff first/more often), and **robustness** (the web is full of traps and garbage).

