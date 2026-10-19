# -nodexp
NodeXP — Node.js + Python, un seul processus, une seule boucle d'événements. Appels croisés sans processus enfant ni sérialisation : pont natif, tampons partagés sans copie, async intact des deux côtés.
README.md
# NodeXP

**Node.js and Python in one process, one event loop, one heap.**

Call Python from JavaScript and JavaScript from Python without spawning a
child process, without serializing through stdin/stdout, and without giving
up the async model on either side.

<!-- TODO: replace with your real badge URLs once CI is green -->
[![CI](https://github.com/USER/nodexp/actions/workflows/ci.yml/badge.svg)](https://github.com/USER/nodexp/actions/workflows/ci.yml)
[![npm](https://img.shields.io/npm/v/nodexp.svg)](https://www.npmjs.com/package/nodexp)
[![PyPI](https://img.shields.io/pypi/v/nodexp.svg)](https://pypi.org/project/nodexp/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## The problem

Every project that needs both ecosystems ends up building the same glue:
a Node process spawns a Python child, they talk over pipes or a socket, and
every call pays serialization cost plus a process hop. Async is lost at the
boundary. Errors arrive as strings. Debugging means reading two logs.

NodeXP removes the boundary. Both interpreters live in the same process,
share typed buffers, and cooperate on a single event loop.

## Quick start

```bash
npm install nodexp        # JavaScript side
pip install nodexp        # Python side

Hello from Python, called from JavaScript:
const xp = require('nodexp');

const stats = xp.import('statistics');
const mean = stats.mean([2, 4, 6, 8]);
console.log(mean);        // 5.0

Hello from JavaScript, called from Python:
import nodexp

math = nodexp.js("Math")
print(math.max(3, 7, 2))   # 7
Architecture
┌───────────────────────────────────────────────────────────────┐
│                        NodeXP process                         │
│                                                               │
│   ┌──────────────────┐              ┌─────────────────────┐   │
│   │   V8 + libuv     │              │      CPython        │   │
│   │   JS event loop  │◄── bridge ──►│   interpreter       │   │
│   │                  │              │   (GIL, refcounts)  │   │
│   └──────────────────┘              └─────────────────────┘   │
│            │                                   │              │
│            └──────────────┬────────────────────┘              │
│                           ▼                                   │
│              shared heap · typed buffers · refs               │
└───────────────────────────────────────────────────────────────┘
<!-- TODO: declare which integration strategy you actually implemented. The three realistic options are: (a) native addon embedding CPython via the C API (node-gyp) (b) WASM-compiled CPython (Pyodide) running inside V8 (c) out-of-process worker with a fast binary IPC channel Readers will ask. State yours here in one sentence. -->

Integration strategy: TODO — one sentence describing how the two interpreters are wired together.
API
<!-- TODO: this is a PROPOSED surface. Replace every name below with the names your implementation actually exports. -->
From JavaScript
Call	Purpose
xp.import(name)	Import a Python module, returns a proxy
xp.call(fn, ...args)	Call a Python callable
xp.eval(source)	Evaluate a Python expression
xp.await_(thenable)	Await a Python coroutine from JS
From Python
Call	Purpose
nodexp.js(expr)	Evaluate a JavaScript expression
nodexp.export(name, fn)	Expose a Python function to JS
nodexp.require(module)	Require a Node module
nodexp.await_(promise)	Await a JS promise from Python
Examples
Async across the boundary
const xp = require('nodexp');
const httpx = xp.import('httpx');

// A Python coroutine awaited from JavaScript
const response = await xp.await_(httpx.get('https://example.com'));
console.log(response.status_code);
Shared buffers, no copy
const xp = require('nodexp');
const numpy = xp.import('numpy');

const buffer = new Float64Array([1, 2, 3, 4]);
const doubled = numpy.multiply(buffer, 2);   // zero-copy view
console.log(doubled[3]);                     // 8
<!-- TODO: add one example that shows error propagation across the boundary. It is the single most common question on projects like this. -->
Error propagation
try {
  xp.eval("1 / 0");
} catch (err) {
  console.error(err.name);      // ZeroDivisionError
  console.error(err.pythonTraceback);
}
Compatibility
<!-- TODO: fill in the versions you actually test against. Do not guess — a wrong matrix generates bug reports you cannot reproduce. -->
Component	Supported
Node.js	TODO
Python	TODO
Operating systems	TODO
Architectures	TODO
Building from source
git clone https://github.com/USER/nodexp.git
cd nodexp
npm install
npm run build          # compiles the native bridge
npm test

Requirements: TODO — compiler, Python dev headers, node-gyp toolchain.
Benchmarks
<!-- TODO: run these on your own hardware and paste real numbers. Leave the table empty until you have measured. Published benchmarks that nobody can reproduce damage a project faster than having none. -->
Operation	NodeXP	Subprocess + JSON IPC	Speedup
10k scalar calls	TODO	TODO	TODO
1 MB buffer transfer	TODO	TODO	TODO
Cold start	TODO	TODO	TODO

Reproduce with: npm run bench
Limitations

Honest constraints, stated up front:

    The GIL still exists. Python bytecode does not run concurrently with itself. Parallelism comes from releasing the GIL during I/O and native calls, not from threads executing Python simultaneously.
    TODO — list your real constraints here. Every project of this shape has at least three. Naming them builds more trust than hiding them.
    TODO
    TODO

Project status

TODO — pick one and be truthful: experimental / alpha / beta / stable. Include the date of the last verified end-to-end run.
Roadmap

    TODO — nearest milestone
    TODO
    TODO

Contributing

See CONTRIBUTING.md. Bug reports are most useful when they include the exact command you ran and its raw output.
License

MIT. See LICENSE.

Third-party component licenses are listed in THIRD-PARTY.md.

---

## Les fichiers d'accompagnement

**`package.json`** — le point d'entrée npm :

```json
{
  "name": "nodexp",
  "version": "0.1.0",
  "description": "Node.js and Python in one process, one event loop, one heap.",
  "main": "lib/index.js",
  "types": "lib/index.d.ts",
  "engines": { "node": ">=20" },
  "scripts": {
    "build": "node-gyp rebuild",
    "test": "node --test test/",
    "bench": "node bench/index.js",
    "lint": "eslint lib/ test/"
  },
  "keywords": ["python", "cpython", "interop", "runtime", "ffi", "polyglot"],
  "license": "MIT",
  "files": ["lib/", "binding.gyp", "README.md", "LICENSE"]
}

