# Wallpaper Engine для KDE Plasma 6 — исправленная сборка v1.5

Готовая сборка плагина **Wallpaper Engine для KDE Plasma 6** на Arch Linux.
Лечит падение `plasmashell` и System Settings при открытии библиотеки обоев
или поиске на **Qt 6.12**.

Это неофициальный backport: тег `v1.5` плюс исправления, которые уже есть в
upstream `main`. Сделан для тех, кто сидит на 1.5 и не хочет тянуть все
изменения после релиза.

## Что исправлено

- **Падение при открытии библиотеки обоев.** В Qt 6.12 `AnimatedImage`
  перезапускает завершившийся `QMovie` прямо из его сигнала `finished`.
  Если у `preview.gif` не читается первый кадр — или у превью нулевая
  ширина — это уходит в бесконечную рекурсию и роняет процесс.
  Теперь первый кадр сначала проверяется обычным `Image`, и только потом
  создаётся `AnimatedImage`.
- **Конфликт `mouseHooker` с настройками Plasma 6.** Старый поиск
  `QQuickGridView` из Plasma 5 отключён; используются `AppletsLayout` и
  `FolderViewDropArea`.

## Кому подходит

- Arch Linux, x86_64
- KDE Plasma 6
- Qt 6.12

Пакет собран против Plasma 6.8.1 / Qt 6.12.0. На других дистрибутивах
используйте сборку из исходников.

## Установка

Скачайте пакет и checksum со страницы релиза:

**https://github.com/xeyej123/wallpaper-engine-kde-plugin/releases/tag/v1.5-wekfix1**

Затем:

```bash
sha256sum -c wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst.sha256
sudo pacman -U wallpaper-engine-kde-plugin-1.5-1-x86_64.pkg.tar.zst
systemctl --user restart plasma-plasmashell.service
```

Версия пакета остаётся `1.5-1`, поэтому `pacman -U` просто переустанавливает
плагин. Ваши обои и настройки сохраняются.

## Проверка

После перезапуска откройте:

**Правый клик по рабочему столу → Настроить рабочий стол и обои → Wallpaper Engine → Wallpapers**

Библиотека и поиск должны открываться без падения `plasmashell`.

## Удаление / откат

```bash
sudo pacman -Rns wallpaper-engine-kde-plugin
systemctl --user restart plasma-plasmashell.service
```

После этого можно поставить официальную версию плагина.

## Сборка из исходников

```bash
git clone -b fix/v1.5-qt6.12-preview-crash \
  https://github.com/xeyej123/wallpaper-engine-kde-plugin.git
cd wallpaper-engine-kde-plugin/arch
makepkg -f
```

В upstream `PKGBUILD` используется `clang`. Если его нет, поставьте `clang`
или соберите с `CC=gcc CXX=g++`.

## Важно

- Пакет **не подписан**. Перед установкой проверьте SHA256.
- Это **не официальный релиз** upstream. В `CaptSilver/wallpaper-engine-kde-plugin:main`
  эти исправления уже есть.
- Исходники и патч: ветка
  [`fix/v1.5-qt6.12-preview-crash`](https://github.com/xeyej123/wallpaper-engine-kde-plugin/tree/fix/v1.5-qt6.12-preview-crash)
  и файл `qt6.12-preview-crash-fix.patch` в релизе.

## Ссылки

- Релиз: https://github.com/xeyej123/wallpaper-engine-kde-plugin/releases/tag/v1.5-wekfix1
- Ветка: https://github.com/xeyej123/wallpaper-engine-kde-plugin/tree/fix/v1.5-qt6.12-preview-crash
- Upstream: https://github.com/CaptSilver/wallpaper-engine-kde-plugin
