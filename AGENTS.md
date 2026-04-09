## Cursor Cloud specific instructions

### Codebase overview

SkyMP is an open-source multiplayer mod for Skyrim SE. This is a C++20/TypeScript monorepo built with CMake + Ninja + vcpkg. The key services that can be built and tested on Linux are the **skymp5-server** (C++ native addon + Node.js/TypeScript) and the **unit tests** (Catch2).

### System prerequisites (Ubuntu 24.04)

The VM image already has build-essential, clang-18, cmake 3.28, Node.js 22, and yarn pre-installed. The update script installs additional packages: `ninja-build`, `automake`, `libtool`, `pkgconf`, `flex`, `bison`, `autoconf`, `autoconf-archive`, `libicu-dev`, `libstdc++-14-dev`.

Key gotcha: Clang 18 on Ubuntu 24.04 selects GCC 14 headers but only GCC 13 dev headers are installed by default. You **must** have `libstdc++-14-dev` installed or C++ compilation will fail with `cannot find -lstdc++`.

### Build & test commands

See `CLAUDE.md` and `CONTRIBUTING.md` for the standard build/test workflow. Summary:

```bash
# Configure (from repo root)
./build.sh --system-compiler --configure -DCMAKE_BUILD_TYPE=Debug -DDOWNLOAD_SKYRIM_DATA=ON

# Build (from repo root)
cd build && cmake --build .

# Run all tests
cd build && ctest --verbose

# Run specific unit test by tag
cd build && ./unit/unit [TagName]
```

The `--system-compiler` flag uses the system clang (v18) instead of requiring clang-15 or clang-20. The `-DDOWNLOAD_SKYRIM_DATA=ON` flag downloads Skyrim ESM data files needed by many tests. Without it, ESPM-related tests are skipped.

### Running the server

```bash
cd build/dist/server
yarn install
node dist_back/skymp5-server.js
```

The server listens on port 7777 (game) and 3000 (HTTP resources). Server settings are in `build/dist/server/server-settings.json`. The default config uses `offlineMode: true` and file-based storage.

### Lint

There is no single lint command. C++ formatting uses `.clang-format` with `clang-format` (the build installs `clang-format-15` or `clang-format-20` depending on distro; on Ubuntu 24.04 you can use the system `clang-format`). TypeScript formatting uses Prettier (configured in CI via `.github/workflows/prettier.yml`).

### Notes

- The `vcpkg` directory is a git submodule; always run `git submodule init && git submodule update` after cloning or pulling.
- The build directory must be exactly `<repo_root>/build` (enforced by CMakeLists.txt).
- Client-side components (skyrim-platform, skymp5-client, skymp5-front) target Windows/Skyrim and cannot be fully built or tested on Linux.
- MongoDB is optional at runtime; the default storage driver is file-based.
