# Backyard Ultra World Team Championship, site source

This is the whole site: one `index.html` (Tailwind loads from its CDN, no
build step), plus an `assets/` folder you'll add once you have real photos
and video. It's meant to live in its own small GitHub repo and deploy with
GitHub Pages, so updating the site is: edit the file, commit, push. No
Replit, no AI agent usage fees, just normal git.

## One-time setup

1. **Create the repo.** On github.com, create a new repository, for example
   `backyard-ultra-site`. Public or private both work with GitHub Pages
   (private needs GitHub Pro/Team/Enterprise for Pages; public is free on
   any plan, and since this is a public event page, public is the simple
   choice).

2. **Push this folder to it.** From inside this folder:

   ```
   git init
   git add .
   git commit -m "Initial site"
   git branch -M main
   git remote add origin https://github.com/<your-username>/backyard-ultra-site.git
   git push -u origin main
   ```

   (If you'd rather not use the command line, GitHub's web UI has an
   "Add file &rarr; Upload files" button that works too for this first
   push, drag `index.html` and `CNAME` in and commit. Editing afterward is
   easier through git though, see below.)

3. **Turn on GitHub Pages.** In the repo, go to Settings &rarr; Pages.
   Under "Build and deployment", set Source to "Deploy from a branch",
   branch `main`, folder `/ (root)`. Save. GitHub will publish the site at
   `https://<your-username>.github.io/backyard-ultra-site/` within a
   minute or two, confirm it loads there first.

4. **Point your custom domain at it.** Still on the Pages settings page,
   under "Custom domain", enter `backyard-ultra.trailrunna.com` and save
   (this repo already has a `CNAME` file with that domain in it, which is
   what makes this box available). Then, wherever trailrunna.com's DNS is
   managed, add:

   ```
   Type:  CNAME
   Name:  backyard-ultra
   Value: <your-username>.github.io
   ```

   GitHub will show a checkmark once it verifies the DNS, then offer
   "Enforce HTTPS", turn that on too. DNS changes can take anywhere from a
   few minutes to a few hours to propagate.

5. **Retire the WordPress version** once the new site is confirmed working
   at backyard-ultra.trailrunna.com: remove or unpublish the Custom HTML
   blocks on the WordPress page. Do this only after the DNS switch is
   confirmed live, not before, so the subdomain is never pointing at
   nothing.

## Updating the site going forward

This is the whole point of moving off Replit's Agent: there's no AI
metering on edits anymore, just git.

- **Ask Claude to make the change** (in a Claude Code session pointed at
  this repo folder locally, or by asking here and copying the updated file
  back in). Claude edits `index.html` directly.
- **Commit and push:**

  ```
  git add .
  git commit -m "describe the change"
  git push
  ```

- GitHub Pages rebuilds automatically on every push to `main`, usually live
  within a minute. Nothing to deploy manually, nothing to configure again.

If you're doing this from a Claude Code session on your own machine (the
same way you already work on the TRN repo), the loop is exactly: open the
folder, ask for the change, review the diff, `git push`. If you're working
from a Cowork/chat session without local git access, ask for the updated
`index.html`, then use GitHub's web editor (the pencil icon on the file, or
github.dev) to paste it in and commit directly in the browser, no local
git required either way.

## Adding real photos and video

Create an `assets/` folder next to `index.html`, drop your images and the
`hero-loop.mp4` in, and update the `REPLACE-VIDEO` / `REPLACE-IMAGE` /
`REPLACE-MAP` spots in `index.html` to point at `assets/your-file.jpg`
instead of the placeholder graphics. Commit and push as above.

Keep an eye on file sizes since GitHub Pages (and git in general) isn't
built for huge binaries: compress photos to a few hundred KB each, and the
hero video to well under 10MB. If the video ever grows past that, host it
externally (Cloudflare Stream, Bunny, even a public S3 bucket) and point
the `<source>` at that URL instead of committing it to the repo.
