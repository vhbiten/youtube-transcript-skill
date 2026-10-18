# YouTube Transcript Skill for AI Agents

Get transcripts, subtitles and captions of YouTube videos, playlists and whole channels from
Claude Code, Codex, Cursor or any agent that supports [Agent Skills](https://skills.sh).
It works on servers and cloud machines too, where YouTube blocks direct requests.

- Plain text for summaries and search, timestamped JSON, SRT or VTT
- Videos, Shorts, playlists and channels (`@handle`)
- Picks the caption language you ask for and falls back to the original
- Python standard library only: no yt-dlp, no browser, no API keys from Google
- US$ 0.004 per transcript. The free Apify plan covers about 1,250 transcripts a month

## Install

```bash
npx skills add vhbiten/youtube-transcript-skill
```

Or copy it by hand:

```bash
git clone https://github.com/vhbiten/youtube-transcript-skill.git
cp -r youtube-transcript-skill/skills/youtube-transcript ~/.claude/skills/
```

Then give the skill an Apify token. Create a free account at [apify.com](https://console.apify.com/sign-up),
copy the token from [Settings > API & Integrations](https://console.apify.com/settings/integrations) and run:

```bash
export APIFY_TOKEN=<your token>
```

## Use

Ask your agent in plain words:

- "Summarize this video: https://youtu.be/dQw4w9WgXcQ"
- "Save the SRT subtitles of this playlist into ./subtitles"
- "What did @veritasium talk about in their last 5 videos?"

Or run the script yourself:

```bash
python3 skills/youtube-transcript/scripts/transcript.py https://youtu.be/dQw4w9WgXcQ
python3 skills/youtube-transcript/scripts/transcript.py @veritasium --max-videos 5 --format srt --out subtitles/
```

All options are in [SKILL.md](skills/youtube-transcript/SKILL.md).

## Why not call YouTube directly?

YouTube blocks most IP addresses that belong to cloud providers. Code that fetches transcripts fine on your
laptop starts failing once it runs on AWS, Google Cloud, Azure, Vercel or a CI runner. The
[youtube-transcript-api README](https://github.com/jdepoix/youtube-transcript-api#working-around-ip-bans-requestblocked-or-ipblocked-exception)
documents this, and [issue #303](https://github.com/jdepoix/youtube-transcript-api/issues/303) collected
nearly 200 comments from people hitting it.

This skill sends the request to the [YouTube Transcript Scraper](https://apify.com/andracore/youtube-transcript-scraper),
which runs on Apify with rotating residential proxies. Your agent gets the transcript no matter where it runs.
More options for this problem in [this article on dev.to](https://dev.to/vhbiten/youtube-transcript-api-works-locally-but-fails-on-the-server-why-and-how-to-fix-it-3emk)
(also in [docs/](docs/youtube-transcript-api-blocked-on-cloud.md)).

## Prefer MCP?

The same scraper works as an MCP tool through the [Apify MCP server](https://docs.apify.com/platform/integrations/mcp).
Add this to your client's MCP config and sign in with Apify in the browser on first use:

```json
{
  "mcpServers": {
    "youtube-transcript": {
      "url": "https://mcp.apify.com?tools=andracore/youtube-transcript-scraper"
    }
  }
}
```

## Without an agent

```bash
curl -X POST "https://api.apify.com/v2/acts/andracore~youtube-transcript-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls": ["https://youtu.be/dQw4w9WgXcQ"], "outputFormat": "text"}'
```

## Pricing

You pay Apify US$ 0.004 per transcript on the free plan and less on paid plans. Compute and proxy are included.
Videos without captions and failed videos are not charged. The script caps each run at
`--max-videos` × US$ 0.004, so a request never costs more than that.

## Other tools by the same author

- [Google Ads Transparency Scraper & Ads Library](https://apify.com/andracore/google-ads-transparency-scraper): competitors' ads from Search, YouTube, Shopping and Maps by brand or domain
- [Shopify Store Analyzer](https://apify.com/andracore/shopify-store-intelligence): business contacts, best sellers, theme, apps and ad pixels of any Shopify store
- [Shopify Store Finder](https://apify.com/andracore/shopify-store-finder): Shopify stores by keyword, with emails, ratings and installed apps

## License

MIT
