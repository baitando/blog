# Blog

This is the source repository of my personal blog which is available on [https://www.baitando.com](https://www.baitando.com). 

# Run Locally Using Docker
See https://github.com/envygeeks/jekyll-docker/blob/master/README.md
```bash
docker run --rm --volume="$PWD:/srv/jekyll" --publish 4000:4000 --publish 35729:35729 jekyll/jekyll jekyll serve --livereload --draft
```

# Run Locally

```bash
bundle exec jekyll serve --livereload --draft
```

# Image Optimization

* Convert color profile if necessary.
* Scale to 1280px x 854px.
* Export as JPEG with 90% quality.
* Image should not be bigger than 500 KB.

# Deployment Status
[![Netlify Status](https://api.netlify.com/api/v1/badges/562beaea-5ec9-4fdb-8eb8-f44d1f139b9b/deploy-status)](https://app.netlify.com/sites/baitando/deploys)