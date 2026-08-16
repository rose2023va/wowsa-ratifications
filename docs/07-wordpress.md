# The ratification page

Built on a **separate WordPress installation** from the main site.

| Item | Value |
|---|---|
| Site | https://openwaterswimming.com/ratifications/ |
| Admin | https://openwaterswimming.com/ratifications/wp-admin/ |
| URL pattern | `/ratifications/swim-XXXX-firstname-lastname/` |
| Example | https://openwaterswimming.com/ratifications/swim-1039-nicholas-bufton/ |

There is a template, but in practice a page is made by **copying an existing post** and replacing
its contents. Every page therefore inherits whatever the last one happened to contain.

## Lifecycle

Three states, and this has always been the design:

1. **Draft** while the page is being assembled
2. **Published private with a password** while the committee reviews
3. **Published publicly** once a determination is recorded

## Sections, in page order

| Section | How it is produced |
|---|---|
| Header | Fixed template: site logo, navigation, social share for Facebook, X, LinkedIn, WhatsApp |
| Hero | Fixed template: image and the standing tagline about the full swim story |
| Title block | Swimmer name and route, typed |
| Introduction | Generated from the swim information against a tone, flow and wording ruleset defined by the admin |
| Swim Details | Entirely manual, transcribed from the forms. Route, Distance, Exact Elapsed Swim Time, Start local time, Start GPS coordinates, Finish local time, Finish GPS coordinates, Category, Escort Boat, Equipment Used. The admin also selects the best photograph for this section |
| Swim Route and Observer Logs | ZeroSixZero map embedded as an iframe, plus the observer log transcribed entry by entry into a scrolling box |
| Swim Highlights | Vimeo embed and a photo gallery, each image captioned with its metadata timestamp |
| Personal Narrative | Taken from the swimmer narrative form and placed manually, with selected photographs |
| Pilot and Crew | Assembled from whatever the submissions reveal about who did what |
| Observer Report | The post-swim observer report, with the observer's photograph, name and credentials where available. Signatures added manually |
| Download Documents and Forms | Manual links to every submitted document, pointing into Google Drive |
| Committee Review | The cloned committee form while under review, replaced afterwards by two League Tables, committee names and signatures |
| Congratulations to the Swimmer & Crew! | The Canva social card |

## Plugins

WPBakery Page Builder, Code Snippets, Post Views, Yoast SEO, League Table, Speed Optimizer,
Security Optimizer, Envato Market.

**League Table** holds two entries per swim in its own store, not in the post.

**Code Snippets is installed here.** It is not installed on the main site, which is worth knowing
because that has previously been recorded the other way around.

## Access

This site is not currently connected to anything that can read it programmatically. Everything
recorded here comes from block markup supplied by the admin, not from reading the site.

The full block markup for a representative page should be captured into `reference/` so that
anything generating these pages has a specification rather than an example to copy.
