# YouTube search API without an API key: videos and Shorts by keyword

How to get YouTube search results as JSON (views, likes, exact publish date, length, channel and subscribers) without a YouTube Data API key and without its 10,000-unit daily quota.

This repo is a short guide with real output. The search runs on the Apify Actor [YouTube Search Scraper](https://apify.com/renzomacar/youtube-search-scraper), which reads the public search results. No login, no key.

## Why not the official API

The YouTube Data API charges 100 quota units for each `search.list` call, so the default 10,000 units a day buy 100 searches of 50 results. The search response also has no view or like counts: you need a second `videos.list` call for those. For keyword research across a few hundred terms, that quota is gone in minutes.

## Example: air fryer videos uploaded this month, most viewed

Input:

```json
{
  "queries": ["air fryer recipes"],
  "maxResults": 25,
  "type": "videos",
  "uploadDate": "month",
  "sortBy": "views",
  "includeDetails": true
}
```

Run on 2026-11-17: 25 videos with full details in 6.5 seconds. First ten (full file: [sample-output.json](sample-output.json)):

| # | Title | Channel | Views | Likes | Published | Length |
|---|---|---|---|---|---|---|
| 1 | My Friend Showed Me This BANANA Trick — Now I Can't Stop Mak… | VeggieVibe AirFryer | 1,461,958 | 18,492 | 2026-10-21 | 6:10 |
| 2 | I Wrapped 1 kg of Pork in Foil and Put It in the Air Fryer… … | Easy Home Food | 4,359,891 | 6,857 | 2026-10-23 | 6:16 |
| 3 | AIR FRYER AFTER A YEAR! 3 IMPORTANT nuances they DON'T TALK … | Китай SHOP | 2,320,628 | 12,637 | 2026-11-01 | 10:31 |
| 4 | I Started Buying Cabbage EVERY WEEK After Trying This AIR FR… | Air Fryer Meal | 979,388 | 4,793 | 2026-10-18 | 3:27 |
| 5 | I Stopped Buying Bread! Just Blend Chickpeas and Lentils in … | BREADOR | 708,461 | 9,480 | 2026-10-24 | 4:37 |
| 6 | 8 Air Fryer Tricks I Wish I'd Known Sooner — I Use These Eve… | Click DIY | 668,163 | 7,808 | 2026-11-02 | 9:48 |
| 7 | I Wrapped Pork Ribs in Foil for the Air Fryer… and THIS Simp… | AirFryer Flow | 1,176,874 | 2,985 | 2026-11-10 | 6:44 |
| 8 | Como assar castanhas na airfryer / Receitas à Mão / Lidl Por… | Lidl Portugal | 426,102 | 86 | 2026-11-06 | 0:36 |
| 9 | I Wrapped 1 kg of Ribs in Foil and Put Them in the Air Fryer… | Easy Home Food | 1,063,651 | 2,178 | 2026-10-18 | 7:04 |
| 10 | Nester Air Fryer Review 🔥 Is This the Best Air Fryer for In… | Born Creator | 341,017 | 7,895 | 2026-11-06 | 6:20 |

The rows come in the order YouTube returns them. As the table shows, YouTube's own "most viewed" order is loose when combined with a date filter, so sort on the numeric `views` field if you need a strict ranking.

One record (description shortened here):

```json
{
  "query": "air fryer recipes",
  "position": 1,
  "type": "video",
  "videoId": "3ziwl5SoAtg",
  "url": "https://www.youtube.com/watch?v=3ziwl5SoAtg",
  "title": "My Friend Showed Me This BANANA Trick — Now I Can't Stop Making It in My Air Fryer 😋",
  "channelName": "VeggieVibe AirFryer",
  "channelId": "UCIXN0mbT2i9aA2xzl8AzPZQ",
  "channelUrl": "https://www.youtube.com/@VeggieVibeAirFryer",
  "channelVerified": true,
  "views": 1461958,
  "viewsText": "1,461,958 views",
  "publishedText": "3 weeks ago",
  "publishedAtApprox": "2026-10-27",
  "duration": "6:10",
  "durationSeconds": 370,
  "isLive": false,
  "watchingNow": null,
  "descriptionSnippet": "This 3-ingredient banana chocolate cake transforms ripe bananas, eggs, and cocoa powder into a rich, tender dessert, with ...",
  "badges": ["4K", "CC"],
  "thumbnailUrl": "https://i.ytimg.com/vi/3ziwl5SoAtg/hq720.jpg",
  "likes": 18492,
  "comments": 586,
  "publishedAt": "2026-10-21",
  "description": "This 3-ingredient banana chocolate cake transforms ripe bananas, eggs, and cocoa powder into a rich, tender dessert ...",
  "hashtags": [],
  "channelSubscribers": 209000,
  "detailsFound": true,
  "scrapedAt": "2026-11-17T04:29:21.396Z"
}
```

Without `includeDetails` the row stops at `thumbnailUrl` and the run is faster: 450 results for three terms took 30 seconds in our test.

## Call it from code

You need an Apify account and its API token (`APIFY_TOKEN`).

curl, results back in the same request:

```bash
curl -X POST "https://api.apify.com/v2/acts/renzomacar~youtube-search-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "content-type: application/json" \
  -d '{"queries":["air fryer recipes"],"maxResults":25,"uploadDate":"month","includeDetails":true}'
```

Python:

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("renzomacar/youtube-search-scraper").call(run_input={
    "queries": ["air fryer recipes", "home workout"],
    "maxResults": 100,
    "type": "videos",
    "uploadDate": "month",
    "includeDetails": True,
})
rows = list(client.dataset(run["defaultDatasetId"]).iterate_items())
rows.sort(key=lambda r: r["views"] or 0, reverse=True)
for r in rows[:20]:
    print(r["query"], "|", r["views"], "|", r["title"], "|", r["url"])
```

JavaScript:

```js
import { ApifyClient } from "apify-client";

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor("renzomacar/youtube-search-scraper").call({
  queries: ["air fryer recipes"],
  maxResults: 50,
  type: "shorts",
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items.length);
```

## Use it from Claude, ChatGPT or another AI agent (MCP)

The Actor is available as a tool through the Apify MCP server. Add this to your MCP client configuration:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com/?tools=renzomacar/youtube-search-scraper",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

Then ask in plain words, for example: "Find the 30 most viewed YouTube videos about air fryer recipes uploaded this month, with likes and channel subscribers."

## Use it from n8n, Make or Zapier

Use the Apify app in each tool, choose "Run an Actor and get dataset items", pick `renzomacar/youtube-search-scraper` and paste the input JSON above. Each result arrives as one item, so the next step can write it to a sheet or a database.

## Input fields

| Field | What it does |
|---|---|
| `queries` | Search terms, each searched separately. The `query` field in each row says which term it came from. |
| `maxResults` | 1 to 1,000 per term. YouTube itself usually stops after a few hundred. |
| `type` | `all`, `videos` or `shorts`. |
| `uploadDate` | `any`, `hour`, `today`, `week`, `month`, `year`. |
| `duration` | `any`, `short` (under 4 min), `medium` (4 to 20), `long` (over 20). |
| `sortBy` | `relevance`, `views`, `rating`. |
| `includeDetails` | Adds likes, comment count, exact publish date, full description, hashtags and channel subscribers. |
| `country`, `language` | Results as YouTube shows them in that country. |

## What it does not do

- No order by upload date: YouTube removed it. Use `uploadDate` to limit to the last hour, day or week.
- No comments text or transcripts.
- "Shorts only" returns what YouTube surfaces for the term, sometimes only a dozen.
- Without `includeDetails` the date is an estimate from "3 weeks ago"; comment counts above 1,000 are rounded by YouTube.

## Price

Pay per result on Apify: $1 per 1,000 results, details included. Nothing for starting a run, and a search that returns nothing costs nothing.
