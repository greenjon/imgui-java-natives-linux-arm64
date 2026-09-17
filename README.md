# imgui-java-natives-linux-arm64

Prebuilt `libimgui-java64.so` binaries for Linux `aarch64` (ARM64), for use with
[`io.github.spair:imgui-java`](https://github.com/SpaiR/imgui-java).

## Why this exists

Upstream `imgui-java` does not publish Linux ARM64 natives — the
`imgui-java-natives-linux` artifact only ever contains `x86_64` ELF binaries.
Upstream issue [SpaiR/imgui-java#105](https://github.com/SpaiR/imgui-java/issues/105)
("Make it work in arm64 linux") has been open since 2022.

Rather than cross-compiling this on every downstream release (Dear ImGui/imgui-java
bindings change far less often than most apps release), this repo builds a native
ARM64 `.so` **once per `imgui-java` version** on GitHub's free native `ubuntu-24.04-arm`
hosted runners, and publishes it as a GitHub Release asset — so any project (not just
the one this was built for) can just download it instead of building it themselves.

## Usage

1. Find the [Release](../../releases) matching the `imgui-java` version you depend on
   (tag name matches the upstream tag, e.g. `v1.92.7.1`).
2. Download `libimgui-java64-linux-arm64-<version>.so` from that release.
3. Bundle it in your application (e.g. as a resource under
   `natives/linux-arm64/libimgui-java64.so`) and point `imgui.ImGui` at it before
   any `ImGui` calls, either via:
   - `System.setProperty("imgui.library.path", "<dir containing the .so>")`, or
   - extracting it to a temp file and calling `System.load(path)`.

If the version you need isn't published yet, trigger a build yourself (see below) —
it takes a few minutes on the free ARM64 runner, no local cross-compilation toolchain
needed.

## Building a new version

Run the **Build & Release Linux ARM64 Natives** workflow manually
(Actions tab → *Run workflow*), passing the upstream `imgui-java` tag to build,
e.g. `v1.92.7.1`. The workflow:

1. Checks out `SpaiR/imgui-java` at that tag (with submodules).
2. Compiles the Java bindings and generates JNI headers.
3. Runs `imgui-binding`'s `generateLibs` task, which invokes the host C++ toolchain —
   because the runner itself is `aarch64`, the output is naturally an ARM64 ELF shared
   object, no cross-compiler needed.
4. Verifies the output is actually an ARM64 binary (`file` check).
5. Publishes it as a GitHub Release asset tagged with the `imgui-java` version.

## License

The build workflow in this repo is MIT licensed. The binaries it produces are compiled
from [`SpaiR/imgui-java`](https://github.com/SpaiR/imgui-java) source (also MIT) —
see that project for the license covering the compiled code itself.
