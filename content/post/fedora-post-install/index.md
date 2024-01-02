---
title: "Fedora Post Install"
subtitle: ""
description: "После установки Fedora важно настроить систему в соответствии с личными предпочтениями и задачами."
date: 2024-01-02T15:31:19+03:00
author: "Георгий Кузора"
image: "img/bkg3.png"
tags: ["linux", "fedora", "OS", "gnome"]
categories: ["Tech"]
draft: false
---

## Что делать после установки Fedora

После [установки Fedora](https://georgiykuzora.ru/post/fedora-install/) важно настроить систему в соответствии с личными предпочтениями и задачами.

## Обязательные действия

### Настройка swappiness

Современные компьютеры имеют большой объем оперативной памяти, поэтому обычно не возникает проблем с нехваткой памяти и необходимостью использовать своп-память.

Fedora по умолчанию создает виртуальный своп-раздел в формате ZRAM. Поэтому при установке системы не нужно создавать отдельный своп-раздел на диске. Однако, даже если у нас есть ZRAM, мы не хотим, чтобы система использовала его, пока не будет исчерпана вся доступная память. Чтобы задать такое поведение, нужно:

1. Открыть файл `sysctl.conf` в редакторе vi командой `sudo vi /etc/sysctl.conf`.
2. Добавить строку `vm.swappiness = 0` в файл `sysctl.conf` - это означает, что swap начнет создаваться только после полного исчерпания оперативной памяти.

В некоторых ситуациях можно также указать `vm.overcommit_memory = 1`. Это позволит программам запрашивать память, вне зависимости от того сколько памяти доступно физически. Более подробно на эту тему [читать здесь](https://serverfault.com/questions/606185/how-does-vm-overcommit-memory-work).

### Настройка DNF

DNF - менеджер пакетов Fedora. Он известен своей медленной работой. Чтобы исправить это, можно настроить дополнительные параметры.

1. Откройте файл `/etc/dnf/dnf.conf` в редакторе vi командой `sudo vi /etc/dnf/dnf.conf`.
2. Добавьте следующие строки в конец файла:
   - `fastestmirror=True` - DNF будет использовать наиболее быстрое зеркало для скачивания пакетов.
   - `max_parallel_downloads=10` - DNF сможет производить до 10 загрузок параллельно.
   - `deltarpm=True` - DNF будет загружть гораздо меньшие дельта-файлы RPM и компилировать их в RPM локально.
   - `defaultyes=True` - По умолчанию DNF будет предлагать подтвердить Yes нажатием Enter. По умолчанию нажатие Enter подтверждает No.
   - `keepcache=True` - DNF будет сохранять кэш данных о версиях пакетов в течении некоторого времени.

### Включение RPM Fusion

В Fedora по умолчанию отключены репозитории для множества бесплатных и проприетарных пакетов .rpm например Steam, Discord, некоторые мультимедийные кодеки и так далее. Чтобы получить к ним доступ нужно подключить дополнительные [репозитории](https://docs.fedoraproject.org/en-US/quick-docs/rpmfusion-setup/).

```shell
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

Репозитории RPM Fusion также предоставляют метаданные Appstream, чтобы пользователи могли устанавливать пакеты с помощью Gnome Software/KDE Discover. Следующая команда установит необходимые пакеты:

```shell
sudo dnf groupupdate core
```

### Обновление системы

После настройки DNF и RPM Fusion стоит обновить систему:

1. `sudo dnf -y update`
2. `sudo dnf -y upgrade --refresh`
3. После обновления перезагрузите систему.

### Обновление прошивки устройств

Если ваша система поддерживает доставку обновлений микропрограммы через `lvfs`, обновите микропрограмму устройства:

```shell
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates
sudo fwupdmgr update
```

### Установка медиа-кодеков

Установите [медиа-кодеки](https://docs.fedoraproject.org/en-US/quick-docs/installing-plugins-for-playing-movies-and-music/), чтобы обеспечить нормальное воспроизведение мультимедиа.

```shell
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} gstreamer1-plugin-openh264 gstreamer1-plugin-libav --exclude=gstreamer1-plugins-bad-free-devel
sudo dnf install lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia
```

### Установка имени хоста

По умолчанию хост установлен на `fedora`. Смените имя хоста если необходимо.

```shell
hostnamectl set-hostname <выбраное-имя-хоста>
```

### Дальнейшие настройки

Для дополнительной тонкой настройки системы вы можете использовать этот [гайд](https://github.com/devangshekhawat/Fedora-39-Post-Install-Guide).

## Установка и настройка Timeshit

Timeshift - это программа для создания резервных копий и восстановления версий системы. Если тома и подтома файловой системы Btrfs настроены правильно, Timeshift может использовать встроенные возможности этой файловой системы для создания снимков системы. Для настройки Timeshift рекомендуется следовать этому [гайду](https://github.com/linuxmint/timeshift).

## Установка и настройка рабочей среды

Теперь можно установить и настроить приложения и утилиты, необходимые для работы.

### Firefox

Несмотря на то что в будущем [Brave](https://brave.com/ru/) будет использоваться как основной браузер, стоит настроить [Firefox](https://www.mozilla.org/ru/firefox/new/) для более удобного использования в качестве запасного браузера.

Для синхронизации настроек браузера нужно подключиться к своему [аккаунту firefox](https://accounts.firefox.com/).

Для удобства использования Firefox можно установить следующие расширения:

- [AdGuard](https://addons.mozilla.org/en-US/firefox/addon/adguard-adblocker/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
- [Bitwarden](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/)
- [DeepL Translate](https://addons.mozilla.org/en-US/firefox/addon/deepl-translate/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
- [GNOME Shell integration](https://addons.mozilla.org/en-US/firefox/addon/gnome-shell-integration/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
- [Simple Tab Groups](https://addons.mozilla.org/en-US/firefox/addon/simple-tab-groups/)
- [Yandex Music Player](https://addons.mozilla.org/en-US/firefox/addon/yandexmusicplayer/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
- [Yandex search](https://addons.mozilla.org/en-US/firefox/addon/yandex-search-english/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
- [YouTube NonStop](https://addons.mozilla.org/en-US/firefox/addon/youtube-nonstop/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)

### Obsidian и Syncthing

[Obsidian](https://obsidian.md/) - это программа для создания и редактирования заметок, которую следует использовать вместе с [Syncthing](https://syncthing.net/), программой для синхронизации данных между устройствами.

Для настройки Syncthing следует воспользоваться этим [гайдом](https://docs.syncthing.net/intro/getting-started.html#configuring).

Я синхронизирую следующие директории между компьютером и телефоном:

- `~/Downloads/1_Phone`
- `~/Pictures/1_Phone`
- `~/Documents/Books`
- `~/Documents/Notes`

### Установка Git и SSH сертификатов для GitHub

После установки Git следует добавить SSH-ключи на GitHub. Для этого рекомендуется использовать [официальный гайд](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

### Установка шрифтов

На Linux шрифты обычно скачиваются в каталог `~/.local/share/fonts/`.

Для моноширинных шрифтов я использую [Nerd Fonts](https://www.nerdfonts.com/).

Мои любимые шрифты:

- [JetBrainsMono](https://www.jetbrains.com/lp/mono/)
- [VictorMono](https://rubjo.github.io/victor-mono/)

### Установка zsh

Поскольку моя конфигурация Zsh предполагает использование дополнительных утилит, перед настройкой Zsh необходимо установить следующие программы:

#### Pyenv

Руководство по установке [тут](https://github.com/pyenv/pyenv?tab=readme-ov-file#installation).

Нужно не забыть установить [зависимости для компиляции интерпретатора python](https://github.com/pyenv/pyenv/wiki#suggested-build-environment).

#### Poetry

[Poetry](https://python-poetry.org/) устанавливается официальным скриптом: `curl -sSL https://install.python-poetry.org | python3 -`.

#### Starship

Руководство по установке [тут](https://starship.rs/guide/#%F0%9F%9A%80-installation).

#### Современные утилиты

- [ripgrep](https://github.com/BurntSushi/ripgrep)
- [bat](https://github.com/sharkdp/bat)
- [fd](https://github.com/sharkdp/fd)
- [exa](https://github.com/ogham/exa)
- [zoxide](https://github.com/ajeetdsouza/zoxide)
- [xh](https://github.com/ducaale/xh)
- ...q tools

  - [xq](https://github.com/sibprogrammer/xq)
  - [jq](https://jqlang.github.io/jq/)
  - [yq](https://github.com/mikefarah/yq)
  - [jqp](https://github.com/noahgorstein/jqp)

#### Установка Oh-my-zsh и плагинов

Для настройки Zsh я выбрал расширение Oh My Zsh. Руководство по его установке можно найти [здесь](https://ohmyz.sh/#install).

Также я использую несколько дополнительных плагинов, которых нет в базовой конфигурации Oh-my-zsh:

- git
- zsh-autosuggestions
- zsh-syntax-highlighting
- history-substring-search
- auto-notify
- vi-mode
- zsh-you-should-use
- poetry

### Установка Nvim

Для установки Nvim я подключился к репозиторию Copr, который содержит ночные сборки редактора. Это можно сделать с помощью следующих команд:

```shell
dnf copr enable agriffis/neovim-nightly
dnf install -y neovim python3-neovim
```

После установки [Neovim](https://neovim.io/) необходимо установить зависимости, необходимые для компиляции [Treesitter](https://github.com/nvim-treesitter/nvim-treesitter) и загрузки [LSP](https://github.com/neovim/nvim-lspconfig):

```shell
sudo dnf install nodejs gcc g++
```

### Установка golang и hugo

Для работы с Hugo нужно установить зависимости [golang](https://go.dev/) и [DartSaas](https://sass-lang.com/install/):

Для установки DartSaas рекомендую скачать [бинарный файл](https://github.com/sass/dart-sass/releases).

Hugo и golang можно установить через dnf.

```shell
sudo dnf install golang hugo
```

Также hugo можно скачать в виде [бинарного файла](https://github.com/gohugoio/hugo/releases).

### Установка rclone и настройка YandexDisk

Для работы с Yandex Disk можно использовать официальный [cli-tool](https://yandex.ru/support/disk-desktop-linux/). Но я предпочитаю использовать [rclone](https://rclone.org/).

Для настройки rclone для YandexDisk используйте [официальное руководство](https://rclone.org/yandex/).

### Установка Docker

Docker устанавливается согласно [официальному руководству](https://docs.docker.com/engine/install/fedora/).

### Скачивание Dotfiles

Для финальной конфигурации системы и утилит, необходимо скачать свои dotfiles с GitHub.

Мои личные [dotfiles](https://github.com/GeorgeKuzora/dotfiles).

### Добавление alternatives

Для более удобного запуска программ из командной строки, стоит настроить [alternatives](https://www.redhat.com/sysadmin/alternatives-command).

## Настройка окружения рабочего стола

В качестве окружения рабочего стола я выбрал [Gnome](https://www.gnome.org/).

### Настройка Settings

Выбираем settings по вкусу.

### Установка Tweaks

Устанавливаем Tweaks и выставляем настройки по своему вкусу.

```shell
sudo dnf install gnome-tweaks.noarch
```

### Установка расширений Gnome

Предпочитаемые мной расширения Gnome:

- [GSConnect](https://extensions.gnome.org/extension/1319/gsconnect/)
- [Caffeine](https://extensions.gnome.org/extension/517/caffeine/)
- [Go To Last Workspace](https://extensions.gnome.org/extension/1089/go-to-last-workspace/)
- [Focus Follows Workspace](https://extensions.gnome.org/extension/4688/focus-follows-workspace/)
- [VIM Alt-Tab](https://extensions.gnome.org/extension/2212/vim-alt-tab/)
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
- [Gnome 4x UI Improvements](https://extensions.gnome.org/extension/4158/gnome-40-ui-improvements/)
- [Blur my Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/)
- [Vitals](https://extensions.gnome.org/extension/1460/vitals/)
- [Unite](https://extensions.gnome.org/extension/1287/unite/)
- [Alphabetical App Grid](https://extensions.gnome.org/extension/4269/alphabetical-app-grid/)

### Установка dconf-editor и настройка горячих клавиш

Для настройки дополнительных горячих клавиш в Gnome, нужно использовать dconf-editor.

```shell
sudo dnf install dconf-editor
```

Настройки горячих клавиш располагаются по следующему пути- `dconf-editor > org > gnome > desktop > wm > keybindings`.

### Установка темной темы для legacy приложений

Для того чтобы приложения gtk-2 и gtk-3 использовали темную тему, нужно добавить строку `gtk-application-prefer-dark-theme=1` в файл `.config/(gtk-3.0,gtk-4.0)/settings.ini`.

## Установка приложений

Для установки приложений можно использовать dnf или flatpak.

Для приложений которые не хотят использовать темную тему, во `flatseal` нужно выставить значение поля - `GTK_THEME=Adwaita:dark`

### Приложения устанавливаемые через flatpak

- [Flatseal](https://flathub.org/apps/com.github.tchx84.Flatseal) - приложение для управления разрешениями приложений flatpak.
- [Pomodoro](https://flathub.org/apps/io.gitlab.idevecore.Pomodoro) - помодоро таймер.
- [Telegram](https://flathub.org/apps/org.telegram.desktop) - клиент Телеграм.
- [Obsidian](https://flathub.org/apps/md.obsidian.Obsidian) - приложение для ведения заметок.
- [Monophony](https://flathub.org/apps/io.gitlab.zehkira.Monophony) - приложение для прослушивания музыки YouTube.
- [Yoga optimizer](https://flathub.org/apps/org.flozz.yoga-image-optimizer) / [Curtail](https://flathub.org/apps/com.github.huluti.Curtail) - приложения для оптимизации изображений.
- [Calibre](https://flathub.org/apps/com.calibre_ebook.calibre) - приложение для управления ebook библиотекой.
- [Shortwave](https://flathub.org/apps/de.haeckerfelix.Shortwave) - приложение для прослушивания интернет радиостанций.
- [Bitwarden](https://flathub.org/apps/com.bitwarden.desktop) - приложение менеджер паролей.
- [DBeaverCommunity](https://flathub.org/apps/io.dbeaver.DBeaverCommunity) - приложение для управления базами данных.
- [Simplenote](https://flathub.org/apps/com.simplenote.Simplenote) - приложение для ведения заметок.
- [Blanket](https://flathub.org/apps/com.rafaelmardojai.Blanket) - приложение для прослушивания звуков природы.
- [Parabolic](https://flathub.org/apps/org.nickvision.tubeconverter) - приложение для скачивания видео с YouTube.
- [Amberol](https://flathub.org/apps/io.bassi.Amberol) - приложение музыкальный проигрыватель.
- [Eartag](https://flathub.org/apps/app.drey.EarTag) - приложение для изменения метаданных музыкальных файлов.
- [Pinta](https://flathub.org/apps/com.github.PintaProject.Pinta) - приложение редактор изображений.
- [Fragments](https://flathub.org/apps/de.haeckerfelix.Fragments) - приложение торрент клиент.
- [Drawio](https://flathub.org/apps/com.jgraph.drawio.desktop) - приложение для создания блок-схем.
- [Eyedropper](https://flathub.org/apps/com.github.finefindus.eyedropper) - приложение пипетка.
- [Goldendict](https://flathub.org/apps/org.goldendict.GoldenDict) - приложение электронный словарь.

### Приложения устанавливаемые через DNF

- [mc](https://midnight-commander.org/) - консольный файловый менеджер.
- [mpv](https://mpv.io/) - видео проигрыватель.
- [tldr](https://tldr.sh/) - облегченный man
- [VSCode](https://code.visualstudio.com/) - редактор кода.
- [wezterm](https://wezfurlong.org/wezterm/) - современный эмулятор терминала
- [kitty](https://sw.kovidgoyal.net/kitty/) - современный эмулятор терминала
- [brave](https://brave.com/) - приватный браузер.
- `@virtualisation` - пакет программ для работы с виртуальными машинами.
- [delta](https://github.com/dandavison/delta) - утилита для просмотра `git diff`.
- [btop](https://github.com/aristocratos/btop) - консольный системный монитор.
- [lazygit](https://github.com/jesseduffield/lazygit) - консольный UI для git.
- [lazydocker](https://github.com/jesseduffield/lazydocker) - консольный UI для docker.
