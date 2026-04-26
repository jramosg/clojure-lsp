---
name: Editor Bug report
about: Create a report for an issue with clojure-lsp running in your editor
title: '`require` / `:refer` suggestions ignore the current file language (clj/cljs/cljc)'
labels: [bug, editor]
assignees: ''

---

**Describe the bug**

When clojure-lsp suggests namespaces to add to an `ns` form (the "add missing require" code action, and the completion items offered for `:require` aliases / `:refer` symbols), the candidate list is built from every namespace found in the project analysis regardless of the source file each candidate comes from.

That means, when I'm editing a plain `.clj` file, I still get suggestions that are only defined in `.cljs` files (or vice versa). Picking one of those and letting clojure-lsp add the require produces a file that can't even be loaded — the namespace it points to doesn't exist on the JVM.

**To Reproduce**

Steps to reproduce the behavior:

1. Open a project that has both Clojure-only and ClojureScript-only namespaces, e.g.:

   ```
   src/my/app/server.clj      ; (ns my.app.server) — clj only
   src/my/app/browser.cljs    ; (ns my.app.browser) — cljs only
   src/my/app/shared.cljc     ; (ns my.app.shared) — clj + cljs
   ```

2. From `src/my/app/server.clj` (a `.clj` file), type a symbol that isn't required yet and trigger the completion / "add missing require" code action. For example, start typing `browser/…`.

3. clojure-lsp happily offers `my.app.browser` (which is a `.cljs` file) as a candidate and, on accept, writes:

   ```clojure
   (ns my.app.server
     (:require [my.app.browser :as browser]))
   ```

4. The resulting `.clj` file fails to compile — `my.app.browser` is not loadable on the JVM.

The same happens in the other direction: from a `.cljs` file I get `.clj`-only namespaces suggested.

**Expected behavior**

Suggestions for `:require` / `:refer` (both completion and the "add missing require" code action) should take into account the language of the current file:

- From a `.clj` file: only suggest namespaces defined in `.clj` or `.cljc` files.
- From a `.cljs` file: only suggest namespaces defined in `.cljs` or `.cljc` files.
- From a `.cljc` file: suggest namespaces reachable from `.clj`, `.cljs` or `.cljc` (i.e. the union, since `.cljc` can be consumed by both).

Namespaces that are unreachable from the current file's language shouldn't appear in the candidate list at all, so that users don't accidentally add broken requires.

**Screenshots**

N/A

<!-- Fill the template below with the json request/response logs between the LS client (your editor plugin, like Calva, lsp-mode, nvim) and clojure-lsp. -->

<details>
 <summary><b>Log - client <-> server</b></summary>
<pre>
ADD JSON HERE
</pre>
</details>

<!-- Fill the template below with the content of the clojure-lsp log if any exceptions/relevant logs, check https://clojure-lsp.io/troubleshooting/#getting-server-log how to get it -->

<details>
 <summary><b>Log - clojure-lsp</b></summary>
<pre>
ADD HERE
</pre>
</details>

**User details (please complete the following information):**
 - OS: <fill in>
 - Editor: Calva (VS Code)
 - Version: <output of `clojure-lsp --version`>

**Additional context**

Besides producing broken code, the current behavior also makes suggestion lists noisy in projects that target both platforms: every `.clj` buffer shows all the browser-only namespaces as candidates, and every `.cljs` buffer shows all the server-only namespaces. The analysis already knows which file each namespace was defined in, so this can be filtered without additional information.
