# 10d-game

10d-game is a small C game engine for AI-agent experiments. An agent observes a
square field of view and chooses one action per tick. The procedurally generated
world contains four bean flavors that affect satiety, hydration, or reward.
Given the same seed, configuration, and actions, the engine produces the same
world and game evolution.

## Play immediately

Open [`build/10d_game.html`](build/10d_game.html) in a modern browser. It includes
the WebAssembly engine, JavaScript, and styles and works directly from disk
without a server, network connection, or installation.

Use the arrow keys or `W`, `A`, `S`, and `D` to move, `E` to eat, `F` to pick up
a bean, `R` to drop it, and `Space` to wait. A seed can be supplied in the URL:

```text
file:///path/to/10d-game/build/10d_game.html?seed=1234
```

## Included files

- `10d_game.c` and `10d_game.h`: engine source and documented public C API.
- `build/10d_game.wasm`: compiled engine for the browser and Python bindings.
- `build/10d_game.html`: ready-to-play, self-contained browser interface.
- `bindings/browser/10d_game.html`: browser source template.
- `bindings/python`: Python bindings, example configuration, dependency lock,
  tests, and documentation sources.
- `docs`: C API documentation sources and version helper.
- `Makefile` and formatting configurations: build and source-check tooling.
- `LICENSE`: MIT license, with the copyright holder anonymized for review.

Git history, hosting and CI configuration, downloaded SDKs, local environments,
caches, and unrelated generated files are omitted.

## Use from Python

Python 3.14 or newer is required. From this directory, install the bindings and
their runtime dependencies:

```bash
python3.14 -m pip install ./bindings/python
```

Alternatively, use the included dependency lock with uv:

```bash
cd bindings/python
uv sync --locked
cd ../..
```

Example (run from this directory with Python, or with
`uv run --project bindings/python python`):

```python
import xdgame

with xdgame.Game("build/10d_game.wasm", seed=42) as game:
    reward = game.tick(xdgame.Action.NOP)
    print(reward, game.fov.shape)
```

See the [Python binding README](bindings/python/README.md) for the Gym-style
environment, map rendering, graphical interface, configuration, and API docs.
Installing Python dependencies requires an internet connection or local copies
of the dependencies; the bundled browser game runs entirely offline.

## Compile the native C engine

The native engine needs a C23 compiler, the C standard library, and the math
library. Compile it and link it into an application, for example:

```bash
cc -std=c23 -O3 -c 10d_game.c
cc your_program.c 10d_game.o -lm
```

The public API and the game rules are documented in `10d_game.h`.

## Rebuild WebAssembly and the browser game

Rebuilding requires Emscripten. A copy of the SDK is omitted from this archive;
an existing installation can be used with `emcc` on `PATH`:

```bash
make wasm
make html
```

If needed, install the official SDK outside this source directory:

```bash
git clone https://github.com/emscripten-core/emsdk.git /path/to/emsdk
/path/to/emsdk/emsdk install latest
/path/to/emsdk/emsdk activate latest
source /path/to/emsdk/emsdk_env.sh
```

An explicit compiler path can also be supplied:

```bash
make wasm html EMCC=/path/to/emsdk/upstream/emscripten/emcc
```

The bundled build artifacts can be used without installing Emscripten.

## Tests and documentation

Run the included Python tests against the bundled WebAssembly module:

```bash
cd bindings/python
uv sync --locked --group dev
uv run --locked --group dev pytest
```

From the archive root, `make check` checks the C sources with `clang-format`
and `clang-tidy`. `make docs` builds the C API documentation with Doxygen and a
C23 compiler, writing `docs/build/html/index.html`.

To build the Python API documentation:

```bash
cd bindings/python
uv sync --locked --group docs
uv run --locked --group docs sphinx-build -W -b html docs docs/_build/html
```
