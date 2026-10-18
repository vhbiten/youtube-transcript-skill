# youtube-transcript-api works locally but fails on the server: why, and how to fix it

You wrote a few lines with [youtube-transcript-api](https://github.com/jdepoix/youtube-transcript-api),
tested them on your laptop, and every transcript came back. Then you deployed to AWS, Google Cloud, Azure,
Vercel, Render, Railway or a GitHub Actions runner, and the same code started failing on every video.

## The symptom

Depending on the library version, you get one of these:

- `RequestBlocked` or `IpBlocked` in current versions
- `TranscriptsDisabled` on videos that clearly have captions, in older versions
- `429 Too Many Requests`, or an XML parse error such as `no element found`

The giveaway is that the same video works from your machine and fails from the server.

## Why it happens

YouTube blocks most IP addresses that belong to cloud providers. The library's own README says so in
[Working around IP bans](https://github.com/jdepoix/youtube-transcript-api#working-around-ip-bans-requestblocked-or-ipblocked-exception),
and [issue #303](https://github.com/jdepoix/youtube-transcript-api/issues/303) ("works locally, fails on Cloud machine")
gathered nearly 200 comments. Your code isn't wrong. The IP address it runs from is.

Even a home or office IP gets blocked for a while if it sends many requests in a short time.

## Fix 1: rotating residential proxies in the library

The library supports proxies directly. Its README recommends rotating residential proxies, because YouTube
also bans static proxies after a while:

```python
from youtube_transcript_api import YouTubeTranscriptApi
from youtube_transcript_api.proxies import WebshareProxyConfig

ytt_api = YouTubeTranscriptApi(
    proxy_config=WebshareProxyConfig(
        proxy_username="<username>",
        proxy_password="<password>",
    )
)
ytt_api.fetch("dQw4w9WgXcQ")
```

Any other provider works through `GenericProxyConfig(http_url=..., https_url=...)`.

**Good for:** steady volume, when you want to keep everything in your own code.
**Watch out for:** you pay for proxy bandwidth, and you handle retries and IP rotation yourself. Datacenter
proxies are cheaper but get blocked just like cloud IPs.

## Fix 2: fetch from a machine YouTube doesn't block

Run the fetching part from a home connection (an old laptop, a Raspberry Pi) and send the results to your server.

**Good for:** personal projects with low volume.
**Watch out for:** it doesn't scale, it's one more machine to keep running, and the home IP can be
rate limited too.

## Fix 3: a hosted transcript API

Let a service run the requests through its own proxies and give you the result over HTTP.
Disclosure: I built one of these, the [YouTube Transcript Scraper](https://apify.com/andracore/youtube-transcript-scraper)
on Apify. It takes video, playlist or channel links and returns text, timestamped segments, SRT or VTT:

```python
from apify_client import ApifyClient

client = ApifyClient("<APIFY_TOKEN>")
run = client.actor("andracore/youtube-transcript-scraper").call(
    run_input={"urls": ["https://youtu.be/dQw4w9WgXcQ"], "outputFormat": "text"}
)
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item["title"], item.get("transcript") or item.get("error"))
```

It costs US$ 0.004 per transcript on the free plan (less on paid plans), with proxies included. Videos without
captions aren't charged, and the free plan's US$ 5 monthly credit covers about 1,250 transcripts. If you use an AI
agent, the [youtube-transcript skill](https://github.com/vhbiten/youtube-transcript-skill) wraps the same call.

**Good for:** serverless functions, CI jobs and agents, or when you don't want to manage proxies.
**Watch out for:** you depend on a third party and pay per transcript.

## Which one to pick

| Situation | Option |
|---|---|
| Personal script, few videos | Fix 2, or just run it locally |
| Production app, steady volume, Python | Fix 1 with rotating residential proxies |
| Serverless, CI, AI agents, or you don't want to manage proxies | Fix 3 |

Whatever you choose, cache the transcripts you already have. Captions rarely change, and every request you
don't make is one fewer chance of getting blocked.
