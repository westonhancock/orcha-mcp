# browse_context recipes

The virtual tree mounts four roots:

```
/files       Orcha-native files, in their folder structure
/bundles     curated context sets
/databases   one markdown file per record
/sources     documents synced from connected sources
```

Commands: `ls`, `tree`, `find`, `cat`, `grep`, `rg`, `head`, `tail`, `wc`,
`stat`. Pipe with `|`, chain with `;` or `&&`. Scripts are capped at 2000
characters. Pass `cwd` to make relative paths shorter.

## Orient

```sh
cat /README.md
```

```sh
tree -L 2 /files
```

```sh
ls /sources && ls /databases
```

## Find where something lives

```sh
find /files -iname '*deploy*'
```

```sh
rg -il 'rate limit' /files | head -20
```

Use `-l` to get filenames first, then `cat` only what you need. Dumping match
bodies across a large workspace is how you burn a scan budget.

## Read a slice

```sh
head -60 /files/engineering/runbooks/incident-response.md
```

```sh
rg -n 'ORCHA_TOKEN' /files/engineering | head -20
```

`rg -n` gives line numbers, which are what you cite.

## Get a citation

```sh
stat /files/engineering/runbooks/incident-response.md
```

Returns the node's citation, deep link, and freshness. This is the fastest path
from "I found it" to "I can cite it".

## Check a connected source

```sh
ls /sources
```

```sh
rg -il 'webhook signature' /sources | head -20
```

## Scan budget

Large scans stop early and the output ends with
`[scan budget exhausted: ...]`. When you see that, the result is partial.
Narrow the path or the pattern and run it again rather than trusting the
truncated list. Say so if you report from a partial scan.

## What this tool is not

It does not rank by meaning. `rg 'authentication'` finds the literal string and
misses the document that explains the same thing using the word "login".
For meaning, use `search_context`.
