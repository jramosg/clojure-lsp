---
name: Editor Bug report
about: Create a report for an issue with clojure-lsp running in your editor
title: '`Create test` code action appends a duplicate deftest when one already exists'
labels: [bug, editor]
assignees: ''

---

**Describe the bug**
When invoking the **"Create test for '\<fn\>'"** code action on a function that already has a matching `deftest` in the test namespace, clojure-lsp appends a second identical `(deftest <fn>-test ...)` to the test file instead of navigating to the one that is already there.

**To Reproduce**
1. Open a source file, e.g. `src/my/ns.clj`, with a function:
   ```clojure
   (ns my.ns)

   (defn greet [name]
     (str "Hello, " name))
   ```
2. Place the cursor on `greet` and invoke **"Create test for 'greet'"**. A test file is created at `test/my/ns_test.clj` containing `(deftest greet-test ...)`.
3. Go back to `src/my/ns.clj`, place the cursor on `greet` again, and invoke **"Create test for 'greet'"** a second time.

**Expected behavior**
The editor navigates to the existing `(deftest greet-test ...)` in `test/my/ns_test.clj`. The file is not modified.

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
 - OS: [e.g. ArchLinux, MacOS, Windows 10]
 - Editor [e.g. emacs, nvim, VSCode (Calva)]
 - Version: (post the result of `clojure-lsp --version`)

**Additional context**
After step 3, `test/my/ns_test.clj` contains two `(deftest greet-test ...)` forms. Running the test suite produces a duplicate-var warning and only the second definition is exercised, silently discarding the first.
