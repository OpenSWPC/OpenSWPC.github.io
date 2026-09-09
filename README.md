# OpenSWPC documantation

## Build note

### On writing

```bash
mkdocs serve --livereload
```

### Update develop

Build & update develop

```bash 
git fetch origin gh-pages
mike deploy develop
```

Check

```bash
mike list
mike serve
```

Push
```bash
git push origin gh-pages
```

### Releasing new version

Commit 

```bash
git switch main
git add .
git commit -m "Finalize documentation for version XXX"
git push origin main
```

```bash
git tag -a XX.XXX -m "Documentation for version XX.XXX"
git push origin XX.XXX
```

```bash
git fetch origin gh-pages
mike deploy --title "XX.XX" --update-aliases XX.XX latest
```

Check

```bash
mike list
mike serve
```

Push
```bash
git push origin gh-pages
```
