CAZGC Website — File Guide
==========================
This file explains every config and script in this project folder.


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: vercel.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What it does:
  Tells Vercel (the hosting platform) how to serve this project.
  Without it, Vercel assumes you have a framework like Next.js or
  React and tries to run a build step — which would fail, because
  this project is plain HTML with no build step.

Contents explained line by line:

  "buildCommand": null
    → Do NOT run any build command. Most frameworks have a build
      step (e.g. "npm run build") that compiles source files into
      HTML. We skip this entirely because our HTML files are already
      the final output.

  "outputDirectory": "."
    → Serve files from the root folder (the dot means "current
      directory"). Vercel will look for index.html here and serve it
      as the homepage. don.html will be served at /don.

  "cleanUrls": true
    → Removes the .html extension from URLs in the browser.
      Without this: https://cazgc-website.vercel.app/don.html
      With this:    https://cazgc-website.vercel.app/don
      Cleaner, more professional-looking URLs.

  "trailingSlash": false
    → Prevents Vercel from adding a / at the end of URLs.
      Without this: https://cazgc-website.vercel.app/don/
      With this:    https://cazgc-website.vercel.app/don
      Consistent with how most modern websites look.

Where it lives:  Project root (committed to GitHub)
When it runs:    Every time Vercel deploys the project


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: generate_gallery.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What it does:
  Reads all .jpeg photos from the /photos folder, encodes each one
  as a base64 string (so the image data lives inside the HTML with
  no separate image files needed), then injects a photo gallery
  section into don.html just before the footer.

  This is the same "self-contained single file" philosophy used
  across all CAZGC deliverables — one HTML file you can email or
  open anywhere, no broken image links ever.

How it works, step by step:

  1. FIND PHOTOS
       photos = glob.glob(os.path.join(photos_dir, '*.jpeg'))
     → glob.glob is Python's file pattern matcher. It finds every
       file ending in .jpeg inside the /photos directory and returns
       a list of full file paths.

  2. SKIP THE HERO IMAGE
       if "1744953769990.jpeg" in photo_path: continue
     → One specific photo (identified by its filename) is used as
       Don's hero/banner image at the top of the page and should
       not appear again in the gallery. This line skips it.

  3. ENCODE EACH PHOTO AS BASE64
       img_b64 = base64.b64encode(f.read()).decode('utf-8')
     → base64 converts binary image data into a plain text string.
       This string can be embedded directly in HTML as an img src
       attribute. The browser decodes it back into the image.
       Trade-off: file size grows ~33%, but zero external dependencies.

  4. BUILD THE GALLERY HTML
     → For each photo, it creates a <div> with an <img> tag using
       the base64 string as the source. The CSS uses CSS Grid
       (auto-fit, minmax) so the gallery automatically adjusts
       columns based on screen width — responsive with no media
       queries needed.
     → Staggered reveal animations (.d1, .d2, .d3) make photos
       fade in at slightly different times when you scroll to them.

  5. INJECT INTO don.html
       insert_pos = html.find('<!-- FOOTER -->')
     → The script looks for the HTML comment <!-- FOOTER --> as an
       injection point. It slices the existing HTML at that position
       and inserts the gallery section in between.
     → This is a common "marker comment" pattern for code generation
       — you leave a comment in the template HTML that acts as a
       known, reliable insertion target.

How to run it:
  python3 generate_gallery.py
  (Run from the project folder. It overwrites don.html in-place.)

Where it lives:  Project root (NOT in GitHub — excluded by .gitignore)
Dependencies:    Python standard library only (sys, os, base64, glob)


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: get_socials.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What it does:
  Reads socials.md and prints its contents to the terminal.
  That's it — it's a two-line utility script.

  with open("socials.md", "r") as f:
      print(f.read())

Why does this exist as a script instead of just opening the file?
  Likely used during the don.html build process so the content of
  socials.md (LinkedIn URL, Instagram handle, phone, etc.) could be
  piped into another command or quickly referenced in a terminal
  session without leaving the workflow.

  Example of how it would be used:
    python3 get_socials.py          → prints socials.md to terminal
    python3 get_socials.py | pbcopy → copies it to clipboard (Mac)

How to run it:
  python3 get_socials.py
  (Requires socials.md to exist in the same folder.)

Where it lives:  Project root (NOT in GitHub — excluded by .gitignore)
Dependencies:    Python standard library only


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WHAT IS .gitignore?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The .gitignore file tells Git which files to never track or upload
to GitHub. Files listed there are invisible to Git — they stay on
your local machine only.

Currently ignored in this project:
  .DS_Store      → macOS metadata file, meaningless to others
  .claude/       → Claude Code session files (local AI context)
  .vercel/       → Vercel project credentials (orgId, projectId)
  CLAUDE.md      → Internal AI instructions, not for public repos
  *.py           → The helper scripts above
  bio.md         → Source content used to build don.html
  socials.md     → Social links source file
  photos/        → Raw photos (embedded as base64 in don.html already)

What IS in GitHub (the repo):
  index.html     → CAZGC marketing website
  don.html       → Don Joe partner profile page
  vercel.json    → Deployment config
  .gitignore     → This ignore list
  README.txt     → This file
