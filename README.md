# deriito.com

Personal site. Jekyll, built and served by GitHub Pages.

## Writing a post

Create `_posts/YYYY-MM-DD-slug.md`:

    ---
    layout: post
    title: Post title
    ---

    Body in Markdown.

Push to `main`. GitHub builds it. The post shows up on `/writing/`
and in the list on the home page.

## Editing the About page

Everything on `/about/` comes from `_data/about.yml`. Add or remove
entries there; `about.html` does not need to change.

Section types:

| type      | fields                          |
| --------- | ------------------------------- |
| `entries` | `when`, `what`, `where`, `text` |
| `bullets` | plain list of strings           |
| `pairs`   | `label`, `value`                |

HTML is allowed in values — wrap them in single quotes so YAML
does not choke on the attributes.

## Adding a nav item

Add an entry under `nav:` in `_config.yml`, then create a matching
page in the root with a `permalink`.

## Layout

    _config.yml     site settings and nav
    _data/          page content
    _layouts/       page templates
    _posts/         articles
    style.css       all styling, light and dark
    CNAME           custom domain — do not delete

## Local preview

    bundle install
    bundle exec jekyll serve --host 0.0.0.0

Then open port 4000.
