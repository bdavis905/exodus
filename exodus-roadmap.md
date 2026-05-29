# Exodus Build Progress

> Ship review: Thu 2026-06-04.
> Last updated: 2026-05-28.
> Status keys: `[ ]` not started · `[~] ` in progress · `[x]` done.

## Done

- [x] Everything you create now lands in one place — the **Runs** feed.
- [x] Removed dashboard clutter: the **Template** and **Scraper** tabs, and the **Run from References** button.
- [x] Set the image generator to use **GPT Image** by default and dropped the extra model.
- [x] Cleaned up the image view — removed the source blocks and the "add as scene / add as object" options, leaving the image and a download.
- [x] Added a "Back to Runs" link in the **Copy Editor**.
- [x] Customers can add their own **Apify** key (Settings → Pipeline Keys) for the Google image scraper, so the scraping cost is theirs, not ours.
- [x] Fixed the **primer** setup so the command line stops rejecting people's winning ads.
- [x] Moved the **primer** build to run in the background so it no longer times out on long inputs.
- [x] Shipped a new **Exodus** version so the skills install correctly and Claude Code finds them.

## Working on now

- [~] Teaching the **primer** to read messy, unlabeled ad copy and still pull out the hooks, headlines, and body on its own.

## Up next

- [ ] Rework the **Run from Ad** window — one ad at a time, with **templates** folded in.
- [ ] In **Runs**, group an ad's versions (copy-derived, reptile, template) under one card, and add a copy filter.
- [ ] Surface the **swipe file** (your saved competitor ads) inside the **Brand Profiles** tab.
- [ ] Let people view and edit their **primer** — their brand's winning-ad examples — in **Settings**.
- [ ] Add a place in **Settings** for brand details (product photos, founder info, label) that the **templates** use.
- [ ] Smaller cleanups: turn off meme generation in Claude Code for v1 (memes stay in the dashboard), retire leftover legacy pages, and add a "use as reference" action in the **Library**.

## Later in the sprint

- [ ] Let each person connect their **Instagram** so **Genesis** can pull in post ideas.
- [ ] Pull winning **headlines** in from a **Meta** ads export.
- [ ] Rebuild **wild-sourcing** — pulling reference photos from Reddit, Pinterest, and Google.
- [ ] Add the **Genesis** pipeline view so you can see the steps of how an ad gets written.
- [ ] Build the full set of **primers** — hooks, headlines, and two awareness levels of body copy — as separate, editable pieces.

## Ideas for later

- [ ] Edit and remix images right in the app (e.g., "make this one Gatorade").
- [ ] A shared **Genesis** library of reference images everyone can pull from.
- [ ] More ad types — short-form copy, video scripts, and more.

## Getting ready to launch

- [ ] Stand up a separate production environment for paying customers — its own domain, login, and database — kept apart from our testing.
