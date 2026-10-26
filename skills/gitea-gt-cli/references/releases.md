# Releases (`gt release`)

Releases and their assets. `gt <cmd> --help` is authoritative. Global/repository
selection flags are in SKILL.md.

```text
gt release <command>
  asset  create  delete  download  edit  list  publish  upload  view
```

`release list` aliases `release ls`; `release create` aliases `release new`.

**JSON fields:** `assets, author, body, created_at, draft, id, prerelease,
published_at, tag, target, title, url`.

## create

`gt release create [<tag>] [<file>...]` with `-t/--title`, `-n/--notes`,
`-F/--notes-file`, `-e/--editor`, `-d/--draft`, `-p/--prerelease`,
`--target branch|commit` (for a new tag), `--asset File`, `--verify-tag`.
Files after the tag are uploaded as assets; the title defaults to the tag.

`gt` **never pushes**: commits and tags that exist only in your clone are not on
the server, so push them first (`git push <remote> <tag>`). A tag that exists
only in your clone is refused unless `--target` says where Gitea creates it;
`--verify-tag` refuses any tag not on the server yet.

```bash
gt release create v1.0.0 dist/tool-linux dist/tool-darwin --notes-file CHANGELOG.md
gt release create v1.1.0-rc1 --target release/1.1 --draft --prerelease --title "1.1 RC 1"
```

## upload / download

- `upload <tag> <file>... [--name NAME] [--clobber]` — an asset of the same name
  is replaced only with `--clobber`. JSON per asset: `created_at,
  download_count, id, name, size, url`.
- `download [<tag>] [<asset>...] [-A|--archive {zip|tar.gz}] [-p|--pattern glob]
  [-D|--dir dir] [-O|--output file|-] [--clobber] [--skip-existing]` — latest
  release without a tag; `--archive` downloads source code instead.

```bash
gt release upload v1.0.0 dist/tool-linux --clobber
gt release download v1.0.0 --pattern '*linux*' --dir dist
gt release download v1.0.0 --archive zip
```

## asset / publish / edit / delete / list / view

- `asset {list|upload|download|delete|...}` — manage assets individually.
- `publish <tag>...` — publish draft releases.
- `edit <tag>...`, `delete <tag>...`, `list [-L n] [--page n]`,
  `view [<tag>] [-w]` (latest without a tag).
