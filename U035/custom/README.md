# Your edited copy of the screen

`U035/custom/MainAppScreen.pa.yaml` starts as an exact copy of `U035/MainAppScreen.pa.yaml`
(including the Date picker and the Visual Defects Yes/No buttons).

Paste your version of the screen YAML into this file. Leave `U035/MainAppScreen.pa.yaml` alone: it is the
baseline Claude edits, so the two files can be compared to see exactly what you changed.

To compare, ask Claude, or run:

```bash
git diff --no-index U035/MainAppScreen.pa.yaml U035/custom/MainAppScreen.pa.yaml
```

This folder is kept separate on purpose so Studio source tooling never sees two screens with the same name.
