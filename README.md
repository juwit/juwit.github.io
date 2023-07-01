# juwit.github.io

This is my personnal website / blog.

It is based on the `github/personal-website` repository.

## Running locally using Docker

Use the following command to build and run the website locally :

```bash
docker run --rm --volume=$(pwd):/srv/jekyll -p 4000:4000 -p 8888:8888 jekyll/jekyll:3.8 jekyll serve --force_polling --livereload --livereload-port 8888 --host 0.0.0.0
```