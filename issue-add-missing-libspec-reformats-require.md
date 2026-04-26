---
name: Editor Bug report
about: Create a report for an issue with clojure-lsp running in your editor
title: '"Add missing require" code action reformats the whole `:require` block'
labels: [bug, editor]
assignees: ''

---

**Describe the bug**

When I trigger the "add missing require" code action on an `ns` form where the first required namespace sits on the same line as `:require`, the whole `:require` block gets re-indented instead of just the new entry being added. The diff ends up touching every line, which makes code review noisy for a one-line logical change.

**To Reproduce**

Steps to reproduce the behavior:

1. Start with an `ns` form like this, where `:require` keeps the first libspec on the same line and the rest are aligned under it:

   ```clojure
   (ns my.ns
     (:require [ns.alpha :as a]
               [ns.beta :as b]
               [ns.delta :as d]
               [ns.epsilon :as e]))
   ```

2. Somewhere in the file, use a symbol from a namespace that isn't required yet, e.g. `ns.gamma/foo`.
3. Trigger the code action to add the missing require.
4. `:ns-inner-blocks-indentation` and `:keep-require-at-start?` are not set in `.lsp/config.edn` — just the defaults.

What I see:

```clojure
(ns my.ns
  (:require
   [ns.alpha :as a]
   [ns.beta :as b]
   [ns.delta :as d]
   [ns.epsilon :as e]
   [ns.gamma :as g]))
```

Every existing line of the `:require` block is changed: `:require` now has a newline after it, and every libspec is indented under it with 2 spaces instead of aligned with the first one.

**Expected behavior**

Only the newly added entry should appear in the diff; the existing alignment should be preserved:

```clojure
(ns my.ns
  (:require [ns.alpha :as a]
            [ns.beta :as b]
            [ns.delta :as d]
            [ns.epsilon :as e]
            [ns.gamma :as g]))
```

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

Git diffs for a single added require touch the whole block, hiding the actual change under formatting noise, and any team convention that keeps the first libspec on the same line as `:require` gets silently rewritten every time someone uses this code action.

Workaround — adding this to `.lsp/config.edn` avoids the reformat:

```clojure
{:clean {:ns-inner-blocks-indentation :keep}}
```

But I'd expect the default behavior to preserve whatever style the file already uses.
