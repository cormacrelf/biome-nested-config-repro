# Biome js bug

biome @ version 2.3.5.

## Structure

- biome.jsonc (includes: `**/*.js`)
- app.js (incorrectly formatted)
- subdirectory 
    - biome.jsonc (extends: `//`, includes: `*.ts`)
    - lib.js (incorrectly formatted)
    - lib.ts (incorrectly formatted)

## The control: `biome format --stdin-file-path=app.js < ...`

This works and outputs formatted javascript. Clearly stdin mode can read the
root config and determine it should format a stream representing `app.js` at
the root.

```
biome format --stdin-file-path=app.js < app.js

let x = 5;

...
```

## The failure: `biome format --stdin-file-path=subdirectory/...`

### Formatting `subdirectory/lib.js`

Stdin mode does not format, it just prints the input unchanged.

```
biome format --stdin-file-path=subdirectory/lib.ts < subdirectory/lib.ts

let y              = 65;
...
```

### Formatting `subdirectory/lib.js`

Because the root config includes `**/*.js`, formatting the javascript file
half-works, but ignores the subdirectory's 2-spaces config. As you can see it
did reformat the let binding, but changed the spacing to 4 spaces:

```sh
# using moreutils' sponge
biome format --stdin-file-path=subdirectory/lib.js < subdirectory/lib.js | sponge subdirectory/lib.js
```

```diff
-let y              = 65;
+let y = 65;

 function indent() {
-  // `biome format .` respects the formatter/indentWidth property from the root config (2 spaces)
-  return y;
+    // `biome format .` respects the formatter/indentWidth property from the root config (2 spaces)
+    return y;
 }
```

### Running from within the subdirectory?

If you `cd subdirectory`, it fails as well:

```
cd subdirectory
biome format --stdin-file-path=lib.ts < lib.ts

let y              = 65;
...
```

## `biome format .`: works for all files including in the subdirectory

This behaves as if `subdirectory/biome.jsonc` adds items the root's `includes`
list, which is a justifiable and sane behaviour of "extends". It also correctly
reads the subdirectory's formatter.indentWidth property (2).

```
; biome format .

app.js format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ Formatter would have printed the following content:

    1   │ - let····x·=·5;
      1 │ + let·x·=·5;
    2 2 │
    3 3 │   function indent() {


subdirectory/lib.js format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ Formatter would have printed the following content:

    1   │ - let·y··············=·65;
      1 │ + let·y·=·65;
    2 2 │
    3 3 │   function indent() {


subdirectory/lib.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ Formatter would have printed the following content:

    1   │ - let·y··············=·65;
      1 │ + let·y·=·65;
    2 2 │
    3 3 │   function indent() {


Checked 4 files in 1752µs. No fixes applied.
Found 3 errors.
format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✖ Some errors were emitted while running checks.
```
