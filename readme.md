# Peter Baumgartner's Personal Site

Built with Hugo. Available at http://pmbaumgartner.github.io. Related Notebooks at https://github.com/pmbaumgartner/binder-notebooks.

To run locally:

```bash
hugo server
```

Helpful Flags:

- `--disableFastRender` render to disk rather than memory (use public folder)
- `-D` build files tagged with `draft: true`
- `--cleanDestinationDir` clean build folder (helpful if files are renamed to remove duplicates)

To load theme:

`git submodule update --init --recursive`