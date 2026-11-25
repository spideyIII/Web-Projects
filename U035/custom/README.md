# Your edits go here

`U035/custom/MainAppScreen.pa.yaml` is intentionally blank.

`U035/MainAppScreen.pa.yaml` is the merged baseline: your earlier layout edits plus Claude's changes
(Date of Inspection date picker, Visual Defects Yes/No buttons, DRF Required "No" fix).

When you need to change the UI again:
1. Export or copy the screen YAML from Studio.
2. Paste it into `U035/custom/MainAppScreen.pa.yaml` (replace the placeholder line) and commit.
3. Ask Claude to compare and merge. Claude updates the baseline and blanks this file again.

To compare yourself:

```bash
git diff --no-index U035/MainAppScreen.pa.yaml U035/custom/MainAppScreen.pa.yaml
```
