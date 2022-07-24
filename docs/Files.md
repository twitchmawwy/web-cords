# Source tree
WebCord contains many sources and assets for each of its features – this may
lead to the confusion to the developers, who don't know where they should
place their code and where to find the part for the specific feature.

## Top-level directories and configuration files

WebCord's local repositories usually has the following top-level folders:

- `sources` – this is the most important part of the WebCord's repository, as
   it contains all the source code files and assets (including compiled
   code to JavaScript).

- `docs` – this is where this documentation is placed. It may contain the
   sub-folders with the translations of the documentation to different languages.

- `node_modules` – this is the folder generated by NPM to store WebCord's
   dependencies. It shouldn't be pushed to the remote repository, but if it is
   there for some reason, please create a new issue as quickly as it is
   possible.

- `out` – this is the folder containing the packaged version of the WebCord and
  its *distributables* (i.e `*.deb` for Debian, `*.rpm` for Fedora, `*.AppImage`
  for Linux in general and `*.zip` for Windows and MacOS).

Moreover, you may find these top-level files at your copy of WebCord's
repository as well:

- `package.json` – this file contains the important metadata used by `npm` and
  `electron` like the WebCord's dependencies, path to `main` script, path to
  forge configuration or even that *WebCord is WebCord*.

- `package-lock.json` – a file generated by `npm` and is used to lock
  dependencies to the various versions. As WebCord's release model in master
  tends to always use as much up-to-date dependency versions as it is possible,
  most dependencies aren't locked and there's no lock file published into the
  repository for that reason (see `.gitignore` if you want to change that).

- `yarn.lock` – a lock file similar to `package-lock.json`, except it is used by
  the `yarn` package manager.

- `tsconfig.json` – a JSON with Comments configuration file that contains the
  metadata.

- `.eslintrc.json` – a linter configuration (ESLint).

## Directories and files inside the `sources` folder

To make code browsing much easier, the code and assets are grouped by their type
in following folders:

- `code` – contains the code of the application in TypeScript format. As WebCord
  code is quite large nowadays, the code is separated into different files in
  following folders:

  - `main` – has the part of the code that is WebCord-specific and cannot be
     reused by the any other software without code modifications.

  - `modules` – includes the scripts that are made in such way they can
     be reused by different software without any code modifications. In the
     future, I will host them on the Node.js registry to allow others using them
     in their own software as modules.

  - `build` – folder that includes scripts associated with the build
    configuration; currently it is used for the Electron Forge only (`forge.ts`).

- `assets` – contains all assets being part of the WebCord as a software. Assets
  in this folder are grouped in the following subfolders:
  
  - `icons` – contains all icons used by WebCord (including app icon and tray
    icons).

  - `translations` – a directory that includes official translations made to
    WebCord.

  - `web` – a folder for HTML internal web pages and their content (CSS/JSON
    files) used by WebCord.

- `app` – includes compiled application code to JavaScript language, with
  comments removed to save some space. I don't recommend messing with that files
  as they're overwritten by `tsc` anyway, unless you want to find if TypeScript
  did messed anything during the compilation (which is rather not going to
  happen, at least with the stable features). It has the same structure as
  `code` folder.

## Source files containing WebCord's code

Since initial release, WebCord has drastically changed that it's code became
too large for developing it comfortably in single file. This also needed the
naming scheme. So, in general, WebCord's source code files are named and put in
the folders under following name scheme:
```
{processOrBuild}/[type]/{functionality}.ts
```

- `{processOrBuild}` indicates to which process this script belongs to. WebCord
  currently contains three folders that represents that: `main`, `renderer` and
  `global` (for scripts that belongs to multiple processes). The exception to
  that rule is the `build` directory that contains the WebCord's build
  configuration for the Electron Forge.

- `[type]` subfolders groups code by its purpose. It can mean either a
  `BrowserWindows`-associated scripts (`windows`), window's `preload`s or
  `modules` on which other scripts depends on. There are also scripts that
  doesn't belong to these categories – they're put then directly inside the
  proper `{process}` folder.

- `{functionality}` shows directly what functionality this script implements.
  For instance, `about.ts` indicates the scripts that implements the *About*
  window functionality.

Here's more detailed description of what's been already discussed above:

---

### Cross-process scripts (`global/`):

  - `main.ts` – A script that is mainly responsible for loading another parts of
    the code from the other files and putting them together into the one program.
    It also handles the command-line flags (due to some flags work as a separate
    part of the application) and implements the default properties for the
    created `webContents` (mainly to harden the app security).
  
  - `global.ts` – A script that has simple modules declarations used between
    multiple processes (i.e. both main and renderer process scripts).

---

### Windows-related scripts:

#### Windows declarations (`main/windows/`):

  - `about.ts` – defines the *About* window, which shows the information about
    the application itself (versions, contributors, licenses etc.)

  - `docs.ts` – defines the WebCord's documentation browser,
    using the [`marked`](https://www.npmjs.com/package/marked) engine for the
    markdown rendering.

  - `main.ts` – contains the declaration of WebCord's main window, i.e. a such
    window that contains the Discord interface.

  - `settings.ts` – a window definition used for WebCord's new HTML-based
    configuration panel.

#### Window-specific `preload` scripts (`renderer/preload/`):

  - `main.ts` – a preload script for `mainWindow`,
    basically loads `capturer.ts` and `cosmetic.ts` scripts described in
    the next sections.

  - `settings.ts` – a front-end script for the WebCord's settings manager; it
    communicates via the IPC to load and fetch the configuration from the
    website.

  - `docs.ts` – a script providing the functionality for WebCord's markdown
    documentation reader.

  - `about.ts` – it is responsible for loading the *About* window content based
    on the translation files and pre-defined HTML file structure.

  - `capturer.ts` – communicates with screen capture BrowserView to show window
    list and send chosen source back to the Discord using IPC.

---

### Application modules (`*/modules/`):

#### Cross-process modules (`global/modules/`):

  - `user.ts` – a module that fakes the WebCord's user agent,
    by generating it using Chromium engine version in current Electron release.

  - `l10n.ts` – defines the localization support within WebCord.

  - `package.ts` – Contains a class and functions to load and validate the
    `package.json` file. It might be published in the future outside the
    WebCord's sources (as a standalone NPM package) once it will be designed to
    support all `package.json` properties according to the [NPM docs](https://docs.npmjs.com/cli/v8/configuring-npm/package-json).

#### Renderer process modules (`renderer/modules/`)

  - `capturer.ts` – sends the request to `main.ts` using IPC and waits for it
    to respond with chosen source.

  - `cosmetic.ts` – does some cosmetic changes to the Discord
    website, removing from it some unnecessary content like the application
    download popups.

#### Main process modules (`main/modules/`):

  - `bug.ts` – a module which generates an URL to the new
    GitHub issue, which is an URL to a `bug` template pre-filled with some OS
    details that are available to the Electron. Current (as of WebCord ) implementation

  - `error.ts` – handles `uncaughtException`s, allowing
    for prettier output in some cases as well as `dialog` GUI window displaying
    error message and simplified error stack.

  - `menus.ts` – includes modules that declares various (OS-native)
    menus, as well as some functions implementing various features called via
    it.

  - `config.ts` – contains classes and types used for reading and
    modifying various configuration files in WebCord.

  - `csp.ts` – defines the Content Security Policy information,
    that will be used by WebCord to overwrite the original CSP headers, allowing
    to block some third-party websites not being necessary for Discord to work.
    
  - `client.ts` – contains some globally-used properties within WebCord's main
    process. It contains some metadata not available in `package.json`, like the
    URL to the Discord website.

  - `update.ts` – a module used to implement the updates
    notification functionality in WebCord.

---

### Build configuration (i.e. for Electron Forge):

  - `build/forge.ts` – contains app's Electron Forge configurations.
  
  - `build/forge.d.ts` – includes types definitions for the Electron
    Forge configuration.