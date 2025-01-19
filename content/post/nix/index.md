---
title:       "Конфигурация окружения разработчика при помощи Nix"
subtitle:    ""
description: "В данной статье подробно разбирается процесс настройки окружения пользователя на Linux с использованием Nix package manager и Nix Home Manager."
date:        2025-01-19T14:25:16+03:00
author:      "Георгий Кузора"
image:       "img/fog_forest.jpg"
tags:        ["Nix", "Linux", "Coding"]
categories:  ["Tech"]
draft:       false
---

## Введение

В последние годы я активно использую Nix package manager, Nix flakes и Nix Home Manager как в профессиональной деятельности, так и в домашних условиях. Эти инструменты стали для меня незаменимыми помощниками в управлении программным окружением, особенно при работе с различными дистрибутивами Linux, такими как WSL и Fedora.

На работе, где я занимаюсь разработкой программного обеспечения, Nix package manager и Nix flakes позволяют мне быстро и эффективно создавать изолированные окружения для различных проектов. Это особенно полезно, когда необходимо переключаться между разными версиями библиотек или инструментов. Благодаря Nix, я могу быть уверен, что все зависимости проекта будут установлены корректно и не повлияют на другие рабочие окружения.

Дома, работая на Fedora Linux, я также оценил преимущества Nix Home Manager. Этот инструмент позволяет мне легко управлять конфигурацией моей рабочей среды, включая настройки оболочки, менеджеры окон и даже настройки системы. Это значительно упрощает процесс обновления и переустановки системы, если это вдруг потребуется.

В этой статье я хочу поделиться своим опытом использования Nix package manager, Nix flakes и Nix Home Manager. Я расскажу об их установке, настройке, обновлении и применении на практике. Надеюсь, что эта информация будет полезна как начинающим, так и опытным пользователям Linux, которые хотят улучшить управление своим программным окружением.

## Установка Nix Package Manager

Я не использую NixOS, так что для работы с Nix мне необходим [Nix Package Manager](https://nixos.org/).

Так как я не планирую, что в моей системе будут пользователи кроме меня, кто будет использовать Nix, то для установки я использую Single User Instalation script с [официального сайта NixOS](https://nixos.org/download/):

```shell
sudo sh <(curl -L https://nixos.org/nix/install) --no-daemon
```

После установки следует проверить что в конфигурацию shell была добавлена одна из следующих строк:

```shell
# fish 
if test -e /home/georgiy/.nix-profile/etc/profile.d/nix.fish; . /home/georgiy/.nix-profile/etc/profile.d/nix.fish; end # added by Nix installer

# bash, zsh
if [ -e /home/georgiy/.nix-profile/etc/profile.d/nix.sh ]; then . /home/georgiy/.nix-profile/etc/profile.d/nix.sh; fi # added by Nix installer
```

После установки нужно создать конфигурацию Nix и Nix Package Manager.

## Конфигурация Nix и Nix Package Manager

Я использую минимальную конфигурацию. Лишь для того чтобы иметь возможность использовать [Nix Flakes](https://nixos.wiki/wiki/Flakes) и UnFree пакеты.

Для конфигурации Nix нужно создать директорию `$XDG_CONFIG_HOME/nix`. В этой директории нужно создать файл `nix.conf` со следующим содержанием:

```nix
experimental-features = nix-command flakes
```

Это включит возможность использовать [nix-команды](https://nixos.wiki/wiki/Nix_command) и [Flakes](https://nixos.wiki/wiki/Flakes). Эти возможности необходимы, чтобы использовать Home Manager вместе с Flakes.

Для конфигурации Nix Package Manager нужно создать директорию `$XDG_CONFIG_HOME/nixpkgs`. В этой директории нужно создать файл `config.nix` со следующим содержанием:

```nix
{ allowUnfree = true; }
```

Это позволит установить пакеты которые не имеют OpenSource лицензии.

Кроме этого следует проверить какой [nix-channel](https://nixos.wiki/wiki/Nix_channels) используется. Список nix-channel в системе доступен командой:

```shell
nix-channel --list
```

Вывод команды будет примерно таким:

```
nixpkgs https://nixos.org/channels/nixpkgs-unstable
```

Если в выводе `nixpkgs` не не соответствует `nixpkgs-unstable` канал, то его следует заменить командой:

```shell
nix-channel --add https://nixos.org/channels/nixpkgs-unstable nixpkgs
```

Далее установим Home Manager.

## Установка Home Manager

[Home Manager](https://github.com/nix-community/home-manager) предоставляет базовую систему для управления пользовательской средой с использованием менеджера пакетов Nix вместе с библиотеками [Nix](https://nixos.org/explore.html), найденными в [Nixpkgs](https://github.com/NixOS/nixpkgs). Он позволяет декларативно настраивать пакеты и конфигурационные файлы, специфичные для конкретного пользователя (не глобальные).

Для установки используем [Standalone](https://nix-community.github.io/home-manager/index.xhtml#sec-install-standalone) метод из официальной документации. Выполним команды (первая команда требует ранее установленного `nixpkgs-unstable` канал):

```shell
nix-channel --add https://github.com/nix-community/home-manager/archive/master.tar.gz home-manager
nix-channel --update
nix-shell '<home-manager>' -A install
```

Далее можно приступить к конфигурации окружения при помощи Home Manager и Nix Flakes.

## Конфигурация окружения

Для конфигурации окружения воспользуемся возможностями Home Manager и Nix Flakes.

Пример конфигурации можно посмотреть в моих [dotfiles](https://github.com/GeorgeKuzora/dotfiles/tree/main/.nix/home-manager).

Основными файлами являются:

- `flake.nix`
- `home.nix`

Дополнительные варианты конфигурации для [WSL](https://learn.microsoft.com/ru-ru/windows/wsl/):

- `home-wsl.nix`
- `work-wsl.nix`

Автогенерируеммый lock файл:

- `flake.lock`

### Кофигурация flake.nix

```nix
{
  description = "Home Manager configuration of georgiy";

  inputs = {
    # Входящие пути дял  Home Manager и Nixpkgs.
    nixpkgs = {
      url = "github:nixos/nixpkgs/nixos-unstable";
    };
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };
  
  # Данные конфигурации которые будут на выходе из flake при его запуске
  outputs = { nixpkgs, home-manager, ... }@inputs:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations."georgiy" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home.nix ];
      };
      homeConfigurations."wsl-work" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./work-wsl.nix ];
      };
      homeConfigurations."wsl-home" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home-wsl.nix ];
      };
    };
}
```

#### Основные компоненты:

1. **Описание (description)**:
    - `description = "Home Manager configuration of georgiy";`
        - Это текстовое описание flake, которое помогает понять его предназначение.
2. **Входы (inputs)**:
    - Здесь указываются источники Home Manager и Nixpkgs.
    - `nixpkgs` указывает на репозиторий Nixpkgs на GitHub.
    - `home-manager` также указывает на репозиторий Home Manager и следует за изменениями в `nixpkgs`.
3. **Выходы (outputs)**:
    - Определяются выходы flake, которые включают конфигурации Home Manager.
    - Используются переменные `nixpkgs` и `home-manager`, загруженные из `inputs`.
    - Устанавливается переменная `system` для архитектуры x86_64-linux.
    - `pkgs` наследует legacy пакеты для указанной системы.

#### Конфигурации Home Manager:

- **georgiy**:
    - Эта конфигурация использует файл `home.nix`, который, вероятно, содержит настройки для пользовательского окружения Georgiy в Linux.
- **wsl-work**:
    - Конфигурация для рабочей среды в WSL, используя файл `work-wsl.nix`, который может содержать специфические для работы настройки.
- **wsl-home**:
    - Конфигурация для домашней среды в WSL, используя файл `home-wsl.nix`, который может включать настройки для домашнего использования.

#### Процесс работы:

1. **Загрузка исходных кодов**:
    - Flake загружает исходные коды Nixpkgs и Home Manager из указанных репозиториев.
2. **Определение системы**:
    - Устанавливается переменная `system` как x86_64-linux.
3. **Наследование пакетов**:
    - `pkgs` наследует legacy пакеты для указанной системы из `nixpkgs`.
4. **Конфигурации Home Manager**:
    - Создаются три различные конфигурации Home Manager:
        - **georgiy**: Использует файл `home.nix` для настроек.
        - **wsl-work**: Использует файл `work-wsl.nix` для рабочих настроек в WSL.
        - **wsl-home**: Использует файл `home-wsl.nix` для домашних настроек в WSL.

### Кофигурация home.nix

```nix
{ config, pkgs, ... }:
let
  username = "georgiy";
  home = "/home/georgiy";
in {
  nixpkgs.config.allowUnfree = true;
  news.display = "silent";

  home = {
    username = "${username}";
    homeDirectory = "${home}";
    stateVersion = "24.05";

    packages = [
      pkgs.lazydocker
      pkgs.obsidian
      # ...
    ];

    file = {
      ".config/atuin".source = config.lib.file.mkOutOfStoreSymlink "${home}/.dotfiles/.config/atuin";
      ".config/autostart".source = config.lib.file.mkOutOfStoreSymlink "${home}/.dotfiles/.config/autostart";
      # ...
    };
    sessionVariables = {
      # EDITOR = "emacs";
    };
  };
  # Let Home Manager install and manage itself.
  programs = {
    home-manager.enable = true;
    bash = {
      enable = true;
      bashrcExtra = ''
        fish -i
      '';
    };
  };
}
```

### Основная структура

Конфигурационный файл `home.nix` определяет, какие пакеты будут установлены из Nix package manager, а также устанавливает символические ссылки для конфигурации пакетов и приложений, используемых пользователем. Дополнительно включается конфигурация Bash при помощи Home Manager.

### Основные секции конфигурации:

1. **Определение переменных**:
    - `username` и `home` задают имя пользователя и путь к его домашнему каталогу.
2. **Настройки Nixpkgs**:
    - `nixpkgs.config.allowUnfree = true;` разрешает установку пакетов, которые могут содержать несвободное ПО.
    - `news.display = "silent";` управляет отображением новостей при обновлении пакетов.
3. **Настройки Home Manager для пользователя**:
    - `home` определяет параметры для домашнего каталога пользователя.
    - `username`, `homeDirectory`, и `stateVersion` задают имя пользователя, путь к домашнему каталогу и версию состояния.
4. **Пакеты**:
    - `packages` определяет список пакетов, которые будут установлены для этого пользователя. В данном случае это `lazydocker` и `obsidian`.
5. **Символические ссылки**:
    - `file` задает создание символических ссылок для конфигурационных файлов. Например, `.config/atuin` и `.config/autostart` будут символическими ссылками на файлы в каталоге `.dotfiles`.
6. **Переменные сессии**:
    - `sessionVariables` позволяет задать переменные сессии, например, `EDITOR` для редактора по умолчанию.
7. **Программы**:
    - `programs` включает Home Manager для управления собой и конфигурацию Bash.
    - `home-manager.enable = true;` позволяет Home Manager управлять собой.
    - `bash = { enable = true; }` включает конфигурацию Bash.
	- `bashrcExtra` позволяет добавлять дополнительные команды в файл `.bashrc`. В данном примере это команда для запуска оболочки Fish `fish -i` означает, что при входе в систему Bash будет запускать интерактивную оболочку Fish.
### Подробное описание:

#### 1. Определение переменных

```
let
  username = "georgiy";
  home = "/home/georgiy";
in {
  ...
}
```

Здесь задаются переменные `username` и `home`, которые будут использоваться в дальнейшем для определения пути к домашнему каталогу пользователя.

#### 2. Настройки Nixpkgs

```
nixpkgs.config.allowUnfree = true;
news.display = "silent";
```

Эти строки разрешают установку пакетов, содержащих несвободное ПО, и управляют отображением новостей при обновлении пакетов.

#### 3. Настройки Home Manager для пользователя

```
home = {
  username = "${username}";
  homeDirectory = "${home}";
  stateVersion = "24.05";
  ...
}
```

Секция `home` задает параметры для домашнего каталога пользователя. Здесь указываются имя пользователя, путь к домашнему каталогу и версия состояния.

#### 4. Пакеты

```
packages = [
  pkgs.lazydocker
  pkgs.obsidian
];
```

В этом разделе определяется список пакетов, которые будут установлены для пользователя. В данном случае это `lazydocker` и `obsidian`.

#### 5. Символические ссылки

```
file = {
  ".config/atuin".source = config.lib.file.mkOutOfStoreSymlink "${home}/.dotfiles/.config/atuin";
  ".config/autostart".source = config.lib.file.mkOutOfStoreSymlink "${home}/.dotfiles/.config/autostart";
};
```

Секция `file` задает создание символических ссылок для конфигурационных файлов. Например, `.config/atuin` и `.config/autostart` будут символическими ссылками на файлы в каталоге `.dotfiles`.

#### 6. Переменные сессии

```
sessionVariables = {
  # EDITOR = "emacs";
};
```

Здесь можно задать переменные сессии, например, `EDITOR` для редактора по умолчанию. В данном случае это закомментировано, но может быть полезно для настройки среды.

#### 7. Программы

```
programs = {
  home-manager.enable = true;
  bash = {
    enable = true;
    bashrcExtra = ''
      fish -i
    '';
  };
};
```

Секция `programs` включает Home Manager для управления собой и конфигурацию Bash. Здесь также можно добавить дополнительные настройки в `bashrcExtra`, например, для запуска других оболочек или дополнительных команд.

## Загрузка конфигурации окружения

Для загрузки конфигурации и создания соответствующего окружения нужно выполнить команду:

```shell
home-manager switch --flake .#georgiy
```

Здесь `#georgiy` символическое название конфигурации Home Manager заданное в файле `flake.nix`:

```nix
homeConfigurations."georgiy" = home-manager.lib.homeManagerConfiguration {
  inherit pkgs;
  modules = [ ./home.nix ];
};
```

После выполнения команды Home Manager создаст новую версию системы, установить пакеты из Nix Package Manager, создат символические ссылки по указанным путям.

## Обновление пакетов

При обновлении пакетов в данной конфигурации нужно помнить, что мы используем Nix Flakes для управления Nix Package Manager и Nix Home Manager.

При работе с Nix Flakes для обновления текущей версии пакетов на которые указывает наш файл `flake.nix` используем команду:

```shell
nix flake update
```

> Если Nix Flakes не используются то нужна команда `nix-channel --update`.

После этого нужно обновить локально установленные пакеты на новые версии, которые соответсвуют состоянию обновленного файла `flake.nix`. Для этого используем команду:

```shell
home-manager switch --flake .#georgiy
```

Home Manager обновит локальные программы на новую версию.

## Заключение

В данной статье мы подробно рассмотрели процесс настройки окружения пользователя на Linux с использованием Nix package manager и Nix Home Manager.
