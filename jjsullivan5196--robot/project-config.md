---
trigger: always_on
description: A simple calculator web app in Clojure. The server ([http-kit](https://github.com/http-kit/http-kit))
---

# AGENTS.md — robot

A simple calculator web app in Clojure. The server ([http-kit](https://github.com/http-kit/http-kit))
owns all state — the current expression and its result live in a server-side
atom. The frontend is a single HTML page driven by
[Datastar](https://data-star.dev) (v1.0.3): every key press is a `@post` to the
backend, and the backend answers with a Datastar SSE event that morphs the
display fragment. There is no client-side calculator logic and no JavaScript
written by us.

Built with the Clojure CLI (`deps.edn`), not Leiningen. Main namespace:
`robot.core`.

## How it works

```
browser                                   server (http-kit)
───────                                   ─────────────────
GET /            ───────────────────────▶ web/page-html @state
                 ◀─── text/html ────────  full page incl. <div id="display">

click "7"        ───────────────────────▶ (swap! state calc/press "7")
  data-on:click="@post('/press/7')"
                 ◀─── text/event-stream ─ event: datastar-patch-elements
                                          data: elements <div id="display">…</div>
Datastar morphs #display in place
```

- `robot.calc` — pure functions: `press` (state transition per key) and
  `evaluate` (hand-written recursive-descent parser for `+ - * / ( )` and
  decimals). **Never** evaluate user input with `read-string`/`eval`.
- `robot.web` — rendering (`page-html`, `display-html`), the SSE encoder
  (`patch-elements-sse`), and the ring handler (`make-handler`).
- `robot.core` — the `state` atom, `start!`/`stop!`, and `-main`.

Key names in URLs are ASCII words (`plus`, `times`, `equals`, `clear`,
`back`, …) mapped to characters by `calc/keys->chars`; see the `keypad`
table in `robot.web` for the full list.

## Layout

```
robot/
├── deps.edn
├── src/robot/
│   ├── core.clj        # (ns robot.core)  server lifecycle + -main
│   ├── calc.clj        # (ns robot.calc)  pure calculator logic
│   └── web.clj         # (ns robot.web)   HTML, SSE, routing
├── test/robot/
│   ├── calc_test.clj
│   └── web_test.clj
└── .gitignore
```

Namespace ↔ path mapping is standard: `robot.calc-test` →
`test/robot/calc_test.clj` (dashes become underscores in filenames).

## Scaffold from scratch

If the files above are missing, create them exactly as follows.

### `deps.edn`

```clojure
{:paths ["src"]
 :deps  {org.clojure/clojure {:mvn/version "1.12.3"}
         http-kit/http-kit     {:mvn/version "2.8.1"}}
 :aliases
 {:run  {:main-opts ["-m" "robot.core"]}
  :test {:extra-paths ["test"]
         :extra-deps  {io.github.cognitect-labs/test-runner
                       {:git/tag "v0.5.1" :git/sha "dfb30dd"}}
         :main-opts   ["-m" "cognitect.test-runner"]
         :exec-fn     cognitect.test-runner.api/test}
  :nrepl {:extra-deps {nrepl/nrepl {:mvn/version "1.3.1"}}
          :main-opts  ["-m" "nrepl.cmdline" "--interactive"]}}}
```

### `src/robot/calc.clj`

```clojure
(ns robot.calc
  "Pure calculator: state transitions and expression evaluation.
   No I/O, no atoms — everything here is a plain function."
  (:require [clojure.string :as str]))

(def initial
  "Fresh calculator state."
  {:expr "" :result nil})

;; ---------------------------------------------------------------------------
;; Evaluation: a small recursive-descent parser for + - * / ( ) and decimals.
;; Deliberately not `read-string`/`eval` — never evaluate user input as code.

(defn- tokenize [s]
  (let [toks (re-seq #"\d+(?:\.\d+)?|[-+*/()]" s)]
    (when (not= (apply str toks) (str/replace s #"\s" ""))
      (throw (ex-info "invalid characters" {:expr s})))
    toks))

(declare parse-expr)

(defn- parse-atom [[t & more]]
  (cond
    (nil? t)  (throw (ex-info "unexpected end of input" {}))
    (= t "(") (let [[v rest] (parse-expr more)]
                (when (not= (first rest) ")")
                  (throw (ex-info "expected )" {})))
                [v (next rest)])
    (= t "-") (let [[v rest] (parse-atom more)] [(- v) rest])
    (re-matches #"\d+(?:\.\d+)?" t) [(parse-double t) more]
    :else (throw (ex-info (str "unexpected " t) {:token t}))))

(defn- parse-binary
  "Left-assoc binary operators at one precedence level."
  [ops next-level toks]
  (loop [[v rest] (next-level toks)]
    (if-let [f (ops (first rest))]
      (let [[v2 rest2] (next-level (next rest))]
        (recur [(f v v2) rest2]))
      [v rest])))

(defn- parse-term [toks] (parse-binary {"*" * "/" /} parse-atom toks))
(defn- parse-expr [toks] (parse-binary {"+" + "-" -} parse-term toks))

(defn evaluate
  "Evaluate an arithmetic expression string to a double.
   Throws ex-info on malformed input or non-finite results."
  [s]
  (let [[v rest] (parse-expr (tokenize s))]
    (when (seq rest)
      (throw (ex-info "trailing input" {:rest rest})))
    (when-not (Double/isFinite v)
      (throw (ex-info "non-finite result" {:value v})))
    v))

(defn format-number
  "Render a double without a trailing .0 when it is integral."
  [^double v]
  (if (== v (Math/rint v))
    (str (long v))
    (str v)))

;; ---------------------------------------------------------------------------
;; Key presses

(def keys->chars
  "URL-safe key names (used in /press/<key>) to the character they append."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjsullivan5196/robot](https://github.com/jjsullivan5196/robot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
