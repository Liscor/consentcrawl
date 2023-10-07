# ConsentCrawl
Automatically check for GDPR/CCPA consent by running a Playwright headless browser to check for marketing and analytics scripts firing before and after consent. 
- Detect 25+ consent managers
- Detect unconsented third-party domains and cookies
- Classify tracking domains based on 7 commonly used ad blocking lists
- Keep screenshots before and after consent
- Capture JSON-LD and meta tags for convenience
- Run multiple URLs in batch
- Add custom blocklists and consent manager lists

## CLI Arguments
usage: 
```sh
consentcrawl [-h] [--debug] [--headless [HEADLESS]] [--screenshot] [--bootstrap]
                    [--batch_size BATCH_SIZE] [--show_output] [--db_file DB_FILE]
                    [--blocklists BLOCKLISTS]
                    url
```

| Argument | Description |
|----------|-------------|
| url      | (required) URL or file with URLs to test
| --debug  | Enable debug logging
| --headless | Run browser in headless mode (true/false)
|  --screenshot | Take screenshots of each page before and after consent is given (ifconsent manager is detected)
|  --bootstrap | Force bootstrap (refresh) of blocklists
|  --batch_size, -b | Number of URLs (and browser windows) to run in each batch. Default: 15, increase or decrease depending on your system capacity.
| --show_output, -o | Show output of the last results in terminal (max 25 results)
| --db_file, -db | Path to crawl results and blocklist database
|  --blocklists, -bf | Path to custom blocklists file (YAML)

## In action
First install dependencies:
`pip install -r "requirements.txt"`

 And the Playwright browsers:
 `playwright install`

You can provide either a single URL, comma separated list or a file (.txt) with one URL per line.
`python consentcrawl "google.com, google.nl, google.de --headless=false"`

If you have `jq` installed you can pipe the output to jq to directly get, for example, all tracking domains without consent:
`python consentcrawl rts.ch -o | jq '.[] | .tracking_domains_no_consent'`

Or if you want to import into an existing Python script:
```python
import asyncio
from consentcrawl import crawl

results = asyncio.run(crawl.crawl_single("dumky.net"))
```

The playwright browser runs asynchronously which is great for running multiple 
URLs in parallel, but for running a single URL you'll need to use asyncio.run()
to run the asynchronous function.

## How it works
Playwright allows you to automate browser windows. This script takes a list of URLs, runs a Playwright browser instance and fetches data about cookies and requested domains for each URL. The URLs are fetched asynchronously and in batches to speed up the process. After the URL is fetched, the script tries to identify the consent manager and click 'accept' to determine if and what marketing and analytics tags are fired before and after consent. It uses a 'blocklist' to determine whether a domain is a tracking (marketing/analytics) domain.

## Available Consent Managers:
- OneTrust
- Optanon
- CookieLaw (CookiePro)
- Drupal EU Cookie Compliance
- JoomlaShaper SP Cookie Consent Extension
- FastCMP
- Google Funding Choices
- Klaro
- Ensighten
- GX Software
- EZ Cookie
- CookieBot
- CookieHub
- TYPO3 Wacon Cookie Management Extension
- TYPO3 Cookie Consent Extension
- Commanders Act - Trust Commander
- CookieFirst
- Osano
- Orejime
- Axceptio
- Civic UK Cookie Control
- UserCentrics
- CookieYes
- Secure Privacy
- Quantcast
- Didomi
- MediaVine CMP
- CookieLaw
- ConsentManager.net
- HubSpot Cookie Banner
- LiveRamp PrivacyManager.io
- TrustArc Truste
- SFBX AppConsent
- Non-specific / Custom (looks for general CSS selectors like "#acceptCookies" or ".cookie-accept")

Are you missing a consent manager? Feel free to open an issue or pull request!