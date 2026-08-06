# Hosting RuffRooms.ie on GitHub Pages

> **Before every publish: regenerate `home.html` from `RuffRooms.dc.html`.**
> They are two files, and editing the design does not touch the deployed copy.
> This has drifted twice already, once losing the entire admin feature from the
> live site while the source had it. If in doubt, ask for a deploy refresh.

## Current state: the site is behind a holding page

`index.html` is an under-construction page. The working platform is served from
`/home.html`. Nothing about the app is deleted, it is simply not the front door.

**To go live:** copy `RuffRooms.dc.html` over `index.html`, point the redirect in
`404.html` back at `"/"` instead of `"/home.html#"`, and regenerate the full
sitemap. Clean URLs resume automatically at that point, no other changes needed.

## Files

| File | Purpose |
| --- | --- |
| `RuffRooms.dc.html` | The design source. Edit this one. |
| `home.html` | Generated copy of the source. The live platform. |
| `index.html` | Under-construction holding page. Hand-written, not generated. |
| `404.html` | Deep-link handler. Redirects to the app. Not indexed. |
| `support.js` | Runtime the app loads. Must stay beside `home.html`. |
| `assets/` | Local images, including the Twelve Hotel photo set. |
| `sitemap.xml`, `robots.txt` | Search engine hints. |
| `.nojekyll` | Stops Pages from filtering files beginning with an underscore. |

## Important: home.html is a generated copy

`home.html` is a snapshot of `RuffRooms.dc.html`. Editing the design does **not**
update it. Ask for a deploy refresh (or copy the file manually) before every
publish, otherwise the live site serves the previous version.

Do not copy the design over `index.html` while the holding page is up, that
would replace it with the app.

## URLs

While the holding page is up, the app runs in hash mode because it is served
from a `.html` file:

| Path | Screen |
| --- | --- |
| `/` | Under-construction page |
| `/home.html` | Homepage |
| `/home.html#/search` | Accommodation search |
| `/home.html#/stay/<slug>` | Property page, 26 of them |
| `/home.html#/beaches/<slug>` | Place page for a beach |
| `/home.html#/walks/<slug>` | Place page for a walk or trail |
| `/home.html#/admin` | Admin, username and password required |

Requests to the clean paths still work: Pages serves `404.html`, which forwards
`/stay/the-twelve-hotel` to `/home.html#/stay/the-twelve-hotel`. Titles, meta
descriptions and canonical links update per route, and the back button works.

Once the app returns to `index.html`, the same routes serve as clean paths
(`/stay/the-twelve-hotel`) with no code change: the app picks its mode from
whether it is being served as a directory index or a `.html` file.

## Project sites versus a custom domain

Detection is automatic in both cases:

- **Custom domain at the root** (`ruffrooms.ie`): nothing to configure. Add a
  `CNAME` file containing the domain.
- **Project site** (`user.github.io/ruffrooms`): the repository name is detected
  and used as the base path.

To pin the base path explicitly, add this to `<helmet>` in the source:

```html
<meta name="rr-base" content="/ruffrooms" />
```

Set `SITE` in the logic class to the final domain so canonical URLs and the
sitemap point at the right place. It is `https://ruffrooms.ie` at the moment.

## Before launch

1. **Admin auth is cosmetic.** The username and password are in the client
   source and protect nothing. Replace with Supabase Auth and a row-level
   policy before the admin page is reachable on a public domain.
2. **Admin changes are local.** Listings edited in admin are held in that
   browser's storage, so they do not appear on other devices and are lost when
   storage is cleared. This is the main reason to wire Supabase next.
3. **SEO is client-rendered.** Google executes JavaScript, so pages will index
   once the app is the front door, but slower and less reliably than served
   HTML. If organic search matters commercially, prerender each route to a real
   HTML file at build time.
4. **Replace the hotel photography.** The Twelve's images are their copyright.
   Owners should upload their own through the portal.
5. **Compress the images.** The asset folder is unoptimised JPEG.
6. **Wire Supabase.** Property, policy, photo, place and review records all come
   from hardcoded arrays right now.
7. **The enquiry form does not send.** Pages cannot run server code, so posting
   to the property's `enquiryEmail` needs a Supabase Edge Function.
