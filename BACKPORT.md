# Wallpaper Engine KDE — v1.5 backport for Plasma 6 / Qt 6.12

This branch is a **community backport of the upstream crash fixes onto tag
`v1.5`**, for users who are still running the 1.5 package.

Upstream `main` already contains these fixes. This branch exists so a 1.5
installation can be rebuilt or upgraded without pulling every post-1.5 change.

## Fixes

1. **Qt 6.12 `AnimatedImage` / `QMovie::finished` recursion.**
   A multi-frame `preview.gif` whose first frame fails to decode, or an
   `AnimatedImage` with a zero decode width, makes `QMovie::start()` finish
   immediately. Qt 6.12 restarts the movie from inside its own `finished`
   signal, which recurses until the stack overflows and takes down
   `plasmashell` / System Settings.
   The new `PreviewAnimatedImage.qml` decodes the first frame with a plain
   `Image` and only creates the `AnimatedImage` when the first frame decoded
   and the decode width is at least one pixel.

2. **Plasma 6 `mouseHooker` targeting the settings `QQuickGridView`.**
   The Plasma 5 `MouseEventListener → QQuickGridView` lookup is disabled on
   Plasma 6; the hook now uses `AppletsLayout` / `FolderViewDropArea`.

## Commits

- `Animate a wallpaper preview only once its first frame decodes`
- `Keep a wallpaper preview static until it is at least a pixel wide`
- `fix(plasma6): prevent mouseHooker from targeting settings QQuickGridView`
- `tests: adapt broken-preview regression to v1.5 read-only cache_path`

The first two and the mouse-hook fix are upstream commits by their original
authors, cherry-picked onto `v1.5`.

## Arch package

Release assets include:

- `wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst`
- `wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst.sha256`
- `qt6.12-preview-crash-fix.patch`

The binary was built on Arch Linux x86_64 against Plasma 6.8.1 / Qt 6.12.0.
It is unsigned; verify the checksum before installing.

```bash
sha256sum -c wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst.sha256
sudo pacman -U wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst
systemctl --user restart plasma-plasmashell.service
```

The package keeps the upstream version `1.5-1`, so `pacman -U` performs a
reinstall. It can be removed with `sudo pacman -Rns
wallpaper-engine-kde-plugin`.

## Build from source

```bash
git clone -b fix/v1.5-qt6.12-preview-crash \
  https://github.com/xeyej123/wallpaper-engine-kde-plugin.git
cd wallpaper-engine-kde-plugin/arch
makepkg -f
```

The upstream `PKGBUILD` sets `CC=clang`; install `clang` or build with
`CC=gcc CXX=g++` if you prefer GCC.

## Tests

```bash
export PATH="/usr/lib/qt6/bin:$PATH"
QT_QPA_PLATFORM=offscreen qmltestrunner \
  -input tests/qml/tst_preview_animated_image.qml -maxwarnings 0
QT_QPA_PLATFORM=offscreen qmltestrunner \
  -input tests/qml/tst_wallpapergrid_broken_preview.qml -maxwarnings 0
```

The preview wrapper test covers: a valid GIF, a GIF whose first frame fails,
a source change to a broken GIF, a zero-width preview, and widening a preview
back to an animated size.
