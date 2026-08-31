# deriito.com

Personal site. Jekyll, built and served by GitHub Pages.

## Writing a post

Create `_posts/YYYY-MM-DD-slug.md`:

    ---
    layout: post
    title: Post title
    lang: ja
    date: 2026-08-31 22:15:00 +0800
    ---

    Body in Markdown.

Keep the filename slug in ASCII — it becomes the URL. The title itself
can be in any language.

`lang` and `date` are both optional:

- Without `lang` the post is treated as English. Set it for anything
  else: `ja`, `zh-Hans`, `zh-Hant`, `ko`, or any Latin-script tag such
  as `fr` or `nb`. It picks the font stack and sets the `lang`
  attribute, and it is what feed readers use to label the entry.
- Without `date` the post is timestamped 00:00 on the filename date.
  A time with no offset is read in the site timezone, `timezone` in
  `_config.yml`.

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
    assets/         images
    style.css       all styling, light and dark
    CNAME           custom domain — do not delete
