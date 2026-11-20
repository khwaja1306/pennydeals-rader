Penny Radar 🪙

A self-hosted web app that tracks Home Depot penny items, Home Depot / Lowe's deep clearance, and Amazon price glitches, scores how likely each item is to be a real deal, and sends alerts to your phone.

It runs on your own PC, NAS, Raspberry Pi or a small cloud app, checks community sources on a schedule, and shows everything on a mobile-friendly dashboard you can add to your phone's home screen.

Penny prices, clearance cadences and glitches are community observations, not official store policy. A store can decline a penny sale or cancel a glitch order. Treat every alert as a lead to verify, not a guarantee.

Contents
Features
Data sources
Quick start (Windows)
Other ways to run it
Phone alerts setup
Using the dashboard
Alert rules
Penny score
How clearance cadence works
Configuration
Environment variables
API reference
Project structure
Testing
Troubleshooting
Etiquette and limitations
Features
Hunt list: one ranked list of everything worth a trip, combining your watchlist and community penny reports, each with a 0–100 penny score and the reasons behind it.
Watchlist (Home Depot and Lowe's): log clearance tag prices and dates. The app decodes the markdown stage, predicts the next markdown and the penny window, and alerts you when the window opens.
Penny Check: for use in the aisle. Enter a SKU and tag price to get the stage, predicted window, and any community reports for that SKU.
Community feeds: PennyCentral's live Penny List, Reddit, Slickdeals, BudgetLightForum, plus any RSS/Atom feed you add.
⚡ Amazon glitch detection: flags price-mistake posts and links each one to Keepa and camelcamelcamel price history.
Phone alerts through ntfy (free, no account) and/or Telegram, with quiet hours, an overnight summary and a daily digest.
Smart de-duplication: the first fetch of any source records existing items without alerting, and the same alert is never sent twice.
Installable on your phone (Add to Home Screen), with an optional password.
No database server: everything is kept in two JSON files in the data folder.
Data sources
Source	What it provides	Default check interval
PennyCentral Penny List	Community-confirmed Home Depot items scanning at $0.01, with the number of states and reports	60 min
Reddit (r/HomeDepot, r/Lowes)	New posts mentioning penny prices; flags posts that mention your area; extracts SKUs	30 min
Slickdeals Hot Deals (RSS)	Home Depot / Lowe's penny and in-store (YMMV) clearance posts; Amazon glitches	10 min
Slickdeals Frontpage (RSS)	Same filters as Hot Deals	30 min
Slickdeals "price mistake" search (RSS)	⚡ Amazon price glitches	5 min
BudgetLightForum "Home Depot Deal Alerts & Discussion Thread"	Long-running thread of penny and .03 finds	30 min
Your own feeds	Any RSS/Atom URL you add in Settings	Configurable

Linked for checking, not polled:

Scavenger.ai: Home Depot store-level inventory near your ZIP (no API).
BrickSeek: stock checks for Lowe's items.
Keepa / camelcamelcamel: Amazon price history.

Manual-only communities (no feed an app can read): the Home Depot One Cent Items Facebook group (PennyCentral already collects its finds), the Garage Journal "Home Depot deals/sales/clearance" thread, and the various Discord servers.

Quick start (Windows)

Requires: Node.js 18 or newer (LTS recommended).

Unzip penny-radar.zip. The zip contains a penny-radar folder, so you may end up with ...\penny-radar\penny-radar\. Use the inner folder, the one that contains package.json.
Open PowerShell in that folder:
powershell
cd "C:\Personal Projects\Deals\penny-radar\penny-radar"
npm install
npm start
Open http://localhost:8787 in your browser.
If Windows asks whether Node.js may use the network, allow it on private networks so you can reach the dashboard from your phone on home Wi-Fi.

npm install only needs to run the first time, and again after each update.

Updating

Unzip the new version over the old folder, then run npm install and npm start again. Your settings, watchlist and history live in the data folder, which the zip never includes, so they're kept. New built-in feeds are added to your settings automatically, once.

Keeping it running
Simplest: leave the PowerShell window open.
In the background on Windows: npm install -g pm2, then pm2 start src/server.js --name penny-radar. For auto-start at boot, use pm2-installer or Task Scheduler.
Other ways to run it
Docker (PC, NAS, Raspberry Pi)
bash
docker build -t penny-radar .
docker run -d --name penny-radar \
  -p 8787:8787 \
  -v penny-data:/data \
  -e APP_PASSWORD=choose-a-password \
  -e TZ=America/New_York \
  --restart unless-stopped \
  penny-radar
Azure App Service (Linux, Node 22)

Deploy the folder (VS Code Azure extension, az webapp up, or zip deploy), then under Configuration → Application settings:

Setting	Value
APP_PASSWORD	a password (required once the app is public)
DATA_DIR	/home/data (persistent storage)
TZ	America/New_York
WEBSITES_PORT	8787

Turn Always On on, or the scheduler stops when the app idles.

Reaching it from your phone away from home
Tailscale (recommended, free): install it on the PC and your phone, then open http://<pc-name>:8787.
Or: host it on Azure as above.

Don't port-forward it to the open internet without APP_PASSWORD.

Phone alerts setup
ntfy (recommended, about 2 minutes)
Install ntfy from the App Store or Google Play.
In the app, tap + and subscribe to a hard-to-guess topic, for example khwaja-penny-8f3k2. Anyone who knows the topic name can read it.
In Penny Radar, go to Settings → Phone alerts: enable ntfy, enter the same topic, and save.
Tap Send test alert. It should appear on your phone within seconds.

Urgent alerts use ntfy's highest priority, which can break through Do Not Disturb if you allow it in the ntfy app.

Telegram (optional)
Message @BotFather, send /newbot, and copy the bot token.
Send any message to your new bot, then open https://api.telegram.org/bot<TOKEN>/getUpdates and copy chat.id.
Enter both in Settings, enable Telegram, save, and send a test.
Dashboard links in alerts

Set Settings → Dashboard URL (for example http://192.168.1.20:8787) so the overnight summary and daily digest link back to your dashboard.

Using the dashboard
Tab	What it's for
Hunt	Best odds first: watchlist items plus Penny List items, with score, reasons and links. Tap a SKU to copy it into the store app.
Penny List	PennyCentral items with search, a minimum-states filter, a state-count trend line, and + Watch.
Forums	Reddit and RSS feeds merged. Filters: All, Penny, ⚡ Glitches, Clearance, Lowe's, 📍 My area. A status line shows which feeds work.
Watchlist	Items you're tracking. Log price after each visit (shelf price, tag date, "app says not found", "unavailable online").
Penny Check	Instant tag decoder plus community lookup for one SKU. Includes a short guide to how penny pricing works.
Alerts	Every alert the app generated and whether it was pushed, held for quiet hours, or failed.
Settings	Alerts, quiet hours, keywords, sources, feeds, glitch rules and your area.

The header shows the health of each source (green or red dot plus when it last ran) and whether alerts are configured.

Alert rules
Alert	Priority	When
🎯 Watchlist item appears on the Penny List	Urgent	A SKU you watch shows up on PennyCentral
🎯 Watchlist SKU mentioned on Reddit or a forum	Urgent	A post contains a SKU you watch
💰 Go check now	Urgent	A watchlist item enters its predicted penny window
⚡ Amazon glitch	Urgent (configurable)	Price mistake detected (see below)
🎯 Penny score reached	High / Urgent	Watchlist score ≥ 70 (configurable); re-alerts at each 10-point step
⏳ Final markdown	High	A watchlist item reaches a .03 / .02 tag
🪙 New on the Penny List	Normal / High	Matches one of your keywords, or already seen in ≥ 5 states
📈 Spreading	Normal / High	A Penny List item gains ≥ 3 states since the last alert
💬 Forum or Reddit penny post	High, or Urgent if it mentions your area	Penny post about Home Depot or Lowe's
🏷️ Forum clearance post	Normal	Only if it matches a keyword, or if "push all clearance" is on
☀️ Overnight summary	High	Non-urgent alerts held during quiet hours, sent when they end
🗞️ Daily digest	Normal	Top watchlist items plus new Penny List items from the last 24 h

Quiet hours (default 22:00–07:00) hold everything except urgent alerts.

Glitch detection: a post is a glitch when it calls itself a price mistake or glitch, when the deal price is at least 85% below another price in the post, or when an Amazon item is listed at $0.01. Ordinary Amazon deals are ignored. By default only Amazon glitches are kept; add walmart, homedepot, lowes or any under Glitch stores.

Penny score

A 0–100 estimate of how likely an item is to scan at $0.01 right now.

Signal	Points
Tag is $0.01	100 (final)
Tag ends .03 / .02 (final markdown)	+60
Tag ends .06 / .04 (mid-clearance)	+30
Tag ends .00 (initial clearance)	+12
Inside the predicted penny window	+20
Penny window opens within 7 days	+10
Past the window (may already be pulled)	+8
On the PennyCentral list	+25, plus 1 per state (max +10)
Home Depot app shows "not found" or no price while the item is on the shelf	+15
Reported as a penny on Reddit or a forum	+8
Unavailable online	+5
Lowe's item	× 0.6 (Lowe's rarely sells penny items; scored on discount depth)

The score is capped at 99 unless the tag itself is $0.01.

How clearance cadence works

Community-observed Home Depot cadences, used for predictions:

Stage	Cadence A (~13 weeks)	Cadence B (~7 weeks)
Initial clearance	.00	.00
Mid-clearance	.06 (~6 weeks)	.04 (~4 weeks)
Final markdown	.03 (~3 weeks)	.02 (~2 weeks)
Penny	$0.01 (if not pulled)	$0.01 (if not pulled)

The date printed on the clearance tag starts the clock. If you don't enter one, the date you logged the price is used. Endings like .97 are normal sale prices, not clearance. Stores vary a lot, so predicted dates are planning windows, not promises.

For Lowe's, the app scores the discount from the original price you enter: 75%+ counts as final, 50%+ as mid, and 20%+ as initial.

Configuration

Everything is editable in Settings and stored in data/settings.json.

Setting	Default	Notes
Home ZIP / area label	43035 / Lewis Center, OH	For your reference and links
Your-area words	ohio, columbus, lewis center, delaware, polaris, westerville	Posts with these get 📍 and higher priority
Keywords	milwaukee, dewalt, ryobi, makita, husky, cooler, generator, vanity, faucet	Make Penny List items and clearance posts "interesting"
Push new Penny List item if ≥ N states	5	
Push when an item spreads by ≥ N states	3	
Push watchlist items at score ≥	70	
PennyCentral interval	60 min	Minimum 30; please be polite
Reddit interval	30 min	
Subreddits	HomeDepot, Lowes	
Reddit client ID / secret	empty	Optional; see Troubleshooting
Forum feeds	4 built in	One per line: Name | URL | store | minutes
Push all clearance posts	off	On = noisy
Glitch alerts	on	
Glitch stores	amazon	amazon, walmart, homedepot, lowes, or any
Glitch threshold	85 %	
Glitch priority	urgent	urgent ignores quiet hours
Quiet hours	on, 22:00–07:00	
Daily digest	on, 07:30	
Feed line format
Name | RSS or Atom URL | store (optional) | check every N minutes (optional) | off (optional)

Examples:

Slickdeals price mistakes | https://slickdeals.net/newsearch.php?searchin=first&rss=1&sort=newest&q=price+mistake | 5
BudgetLightForum HD thread | https://budgetlightforum.com/t/home-depot-deal-alerts-discussion-thread/27168.rss | homedepot
My Telegram bridge | https://example.com/channel.rss | lowes | 15 | off

Setting a store on a feed means every post in it is about that store; use this for single-store threads. Without one, a post is kept only if it names Home Depot or Lowe's, or is a glitch.

Environment variables
Variable	Default	Purpose
PORT	8787	HTTP port
DATA_DIR	./data	Where settings.json and db.json are stored
APP_PASSWORD	unset	Turns on a browser password prompt (any username)
NO_POLL	unset	Set to 1 to run the dashboard without background polling (for testing)
TZ	system	Time zone for quiet hours and digest times (set it in Docker or the cloud)
API reference

All endpoints return JSON. If APP_PASSWORD is set, use HTTP Basic auth.

Method	Path	Description
GET	/api/status	Source health, counts, alert channels, held alerts
GET	/api/hunt?limit=100	Ranked hunt list (watchlist + Penny List)
GET	/api/pennycentral?q=&minStates=&all=	Penny List items with state trend
GET	/api/posts?filter=all|penny|glitch|clearance|lowes|local	Reddit and feed posts merged
GET	/api/reddit	Reddit posts only
GET	/api/watchlist	Watchlist with scores and predictions
POST	/api/watchlist	Add { store, sku, name, storeName, originalPrice, notes }
PATCH	/api/watchlist/:id	Update fields, e.g. { archived: true }
DELETE	/api/watchlist/:id	Remove an item
POST	/api/watchlist/:id/observations	Log { price, date, tagDate, clearance, appNotFound, onlineUnavailable, note }
GET	/api/check?sku=&price=&tagDate=&store=&original=&appNotFound=&onlineUnavailable=	Penny Check
GET	/api/alerts	Last 200 alerts
GET / PUT	/api/settings	Read or replace settings
POST	/api/run/:job?silent=true	Run pennycentral, reddit, feeds or watchlist now; silent records without alerting
POST	/api/test-alert	Send a test push
Project structure
penny-radar/
├── src/
│   ├── server.js            Express API + static dashboard host
│   ├── jobs.js              Scheduler (1-min tick), diffing, alert rules
│   ├── notify.js            ntfy / Telegram, quiet hours, overnight summary, daily digest
│   ├── watchlist.js         Tracked items, observations, evaluation, links
│   ├── config.js            Settings defaults, merge, one-time feed migration
│   ├── store.js             JSON-file store with debounced atomic writes
│   ├── engine/
│   │   └── cadence.js       Tag decoder, penny-window prediction, scoring
│   └── sources/
│       ├── http.js          fetch with timeout + User-Agent
│       ├── pennycentral.js  Penny List parser (HTML cards + embedded-JSON fallback)
│       ├── reddit.js        OAuth → JSON → RSS fallback, keyword/SKU matching
│       └── feeds.js         RSS/Atom parser; store, penny, clearance and glitch classification
├── public/                  Dashboard (vanilla JS PWA: index.html, app.js, style.css, sw.js)
├── test/                    node:test suites + fixtures
├── data/                    settings.json + db.json (created at runtime; not in the zip)
├── Dockerfile
└── package.json

Tech: Node.js 18+, Express 4, Cheerio. The frontend needs no build step.

Testing
bash
npm test

There are 18 tests covering the tag decoder and scoring, the PennyCentral, Reddit and RSS parsers, glitch detection, the full alert pipeline (seeding, new items, spreading, watchlist hits, de-duplication), quiet hours, per-feed intervals, and the settings migration. The tests use saved sample pages and a mocked network; they don't contact live sites.

Troubleshooting
Problem	Fix
Cannot find module 'express'	Run npm install in the folder that contains package.json (the inner penny-radar folder).
Red List dot, "no items were recognised"	PennyCentral changed its page layout. Save the page (Ctrl+S), add it as a fixture, and adjust src/sources/pennycentral.js.
Red Reddit dot (HTTP 403/429)	Reddit is blocking anonymous requests. Create a script app at https://www.reddit.com/prefs/apps and paste its client ID and secret into Settings.
A feed shows 🔴 on the Forums tab	Open its URL in a browser. If the site changed its feed address, update the line in Settings, or add off to disable it.
No phone alerts	Settings → Send test alert, then check the Alerts tab status. "logged (no channel configured)" means ntfy or Telegram isn't enabled or the topic is empty.
Alerts arrive at night	Turn quiet hours on. Urgent alerts (watchlist hits, glitches) still come through by design; set glitch priority to high to hold those too.
Alerts at the wrong time	Set TZ (Docker or cloud) or fix the PC's time zone.
Can't open the dashboard from your phone	Allow Node.js through Windows Firewall on private networks and use the PC's LAN IP, or use Tailscale.
Start fresh	Stop the app and delete data/db.json (keeps settings) or the whole data folder.
Etiquette and limitations
Be polite. Home Depot hasn't published a penny policy. Staff may pull an item or decline the sale; accept it either way.
Don't clear shelves. Leave some for other hunters, and consider reporting your finds back to PennyCentral.
Glitch orders get cancelled often. Retailers aren't required to honor obvious price mistakes.
Polling is deliberately gentle (hourly for PennyCentral, minutes for RSS feeds) and identifies itself with an honest User-Agent. Don't lower the intervals aggressively.
No store APIs are used. Penny Radar reads public community sources only. It doesn't log into Home Depot, Lowe's, Amazon, Scavenger.ai or BrickSeek.
Personal use. Not affiliated with Home Depot, Lowe's, Amazon, PennyCentral, Slickdeals, Reddit or any other site mentioned.
