## Repo Shape
- This repo is a Unity package. The root `package.json` is the Unity package manifest (`com.gilzoide.sqlite-net`), not a Node workspace.
- Main package code lives in `Runtime/` (`Gilzoide.SqliteNet` asmdef, root namespace `SQLite`). Editor-only importers/build hooks live in `Editor/` (`Gilzoide.SqliteNet.Editor`, `autoReferenced: false`).
- The only repo-local tests are Editor tests in `Tests/Editor/`, behind `UNITY_INCLUDE_TESTS`.

## Generated And Vendored Code
- `Plugins/sqlite-net~` is a git submodule pointing at upstream `praeclarum/sqlite-net`.
- `Runtime/sqlite-net/AssemblyInfo.cs`, `SQLite.cs`, and `SQLiteAsync.cs` are generated from `Plugins/sqlite-net~/src/*` by `make -C Plugins source`.
- `Plugins/tools~/fix-library-path.sed` is the patch layer that rewrites upstream sqlite-net for this package (`LibraryPath`, WebGL/iOS `__Internal`, public `Quote`, `partial SQLite3`, WebGL async scheduler, stripping fixes, struct query fix). If a change would be overwritten by `make -C Plugins source`, update the submodule source and/or this sed script instead of only editing `Runtime/sqlite-net/*`.

## Native Libraries
- Native SQLite builds are driven from `Plugins/Makefile`; prebuilt outputs are committed under `Plugins/lib/...`.
- Key targets: `make -C Plugins source`; `make -C Plugins all-linux`; `make -C Plugins all-apple`; `make -C Plugins all-android` or per-ABI `android-arm64|android-arm32|android-x86|android-x86_64` (`ANDROID_NDK_ROOT` must be set); `make -C Plugins all-windows`, `all-windows-mingw`, or `all-windows-llvm-mingw`; `make -C Plugins docker-all-android|docker-all-linux|docker-all-windows|docker-all-windows-llvm`.
- CI only builds native libraries: `.github/workflows/build.yml` runs the Docker targets on Ubuntu and `make -C Plugins all-apple` on macOS. There is no repo CI for Unity tests.

## Verification
- For changes to generated sqlite-net mirror logic, run `make -C Plugins source` and inspect the regenerated files in `Runtime/sqlite-net/`.
- For changes to native bindings or SQLite amalgamation, run the narrowest relevant `make -C Plugins ...` target instead of rebuilding everything.
- For managed package behavior, the only verified test surface in-repo is the Unity Editor test assembly `Gilzoide.SqliteNet.Tests.Editor` (`Tests/Editor/TestSerialization.cs`). No headless Unity command is documented in this repo.

## Behavior Quirks Worth Remembering
- `SQLiteAsset` is strictly read-only: `OpenFlags` strips `ReadWrite` and `Create` both in setter logic and editor validation.
- Streaming-assets-backed `SQLiteAsset` builds are only handled for non-Android/non-WebGL targets. `Editor/SQLiteAssetBuildProcessor.cs` is compiled out on Android/WebGL, and `Runtime/SQLiteAsset.CreateConnection()` falls back to in-memory bytes on those platforms.

## Style / Edit Safety
- `.editorconfig` uses 4 spaces for `*.cs`/`*.asmdef`, tabs in `Makefile` and `*.sed~`.
