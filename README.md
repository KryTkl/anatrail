# media/ folder

Each site's photos and video live in a subfolder named after its `slug`
from `data/sites.json`, using exactly these filenames:

```
media/
  catalhoyuk/
    1.jpg
    2.jpg
    3.jpg
    video.mp4
  aktopraklik-mound/
    1.jpg
    2.jpg
    3.jpg
    video.mp4
  <your-next-site-slug>/
    1.jpg
    2.jpg
    3.jpg
    video.mp4
```

The site automatically looks for these four files per site — nothing needs
to be listed in the JSON. If a file is missing, it's just skipped (no
broken image icon), so you can roll out photos for your 200 sites
gradually without breaking anything.

See SCALABILITY-GUIDE.md for the full workflow.
