# obsidian-system-theme

On Linux, Obsidian's "Adapt to system" theme setting requires that Obsidian be restarted to pick up changes to the system theme.

This plugin uses Electron's [nativeTheme api](https://www.electronjs.org/docs/latest/api/native-theme) to workaround this limitation. When enabled, changing the system theme between light/dark should automatically be applied to Obsidian.

This code is more-or-less the exact same as [kepano's obsidian-system-dark-mode](https://github.com/kepano/obsidian-system-dark-mode) just with the underlying theme API changed.


#### Installation:
requires npm

```
# 1. cd to plugins directory
cd my-vault/.obsidian/plugins

# 2. clone repo
git clone git@github.com:jgauth/obsidian-system-theme.git && cd obsidian-system-theme

# 3. install deps
npm install

# 4. build
npm run build
```
Then restart obsidian and enable the plugin.

#### NOTE:
This plugin is currently sort of broken. On unload there's this error:
```
Plugin failure: obsidian-system-theme TypeError: Cannot read properties of undefined (reading 'offref')
```
so doing a full unload requires restarting obsidian. However it works fine when loaded, so I'm not going to spend any time trying to fix this atm. Its probably something to due with using remote: `require('electron').remote;`

Because of this I won't submit it to the official community plugins for now.

Tested on Ubuntu 20.04

GNOME Shell 3.36.9

----

#### Additional context:

Obsidian forum user [ecchina_ko](https://forum.obsidian.md/u/ecchina_ko) found the theme issue with Obsidian is due to a bug with Chromium:
```
I did some research, and I think I now know where the problem lies.

The Bug

While I obviously don’t have the original source code for Obsidian, I played with the development tools and the transpiled JavaScript code, and it would seem that Obsidian indeed uses the same method as the System Dark Mode plugin; specifically, it calls window.matchMedia() to get a MediaQueryList, to which it attaches an event listener for the prefers-color-scheme media feature, with a callback to a function that sets the theme depending on if the value is dark or not. Now, this would normally be all fine and dandy, except that the prefers-color-scheme feature has been broken on Chromium on Linux since 2019 1, specifically in the way that it does not react to the selected GTK theme – which is what GNOME and Fedora (among other Linux distributions and desktop environments) use. Since Obsidian is dependent on Electron, and Electron is dependent on Chromium, the bug also affects Obsidian.

The Fix

But wait – how does VS Code, which is also written in TypeScript and Electron, get around this on Linux? Their theme service uses Electron’s nativeTheme module 1, which provides an API to read and react to the system’s theme changes. It presents a general interface while performing the actual system-specific theme logic under the hood so that Electron users don’t have to do all the OS-specific handling themselves. Because of the previously mentioned Chromium bug, Electron developers have implemented a workaround for GTK themes for the module. The workaround seems like a good solution for now since Chromium developers don’t seem to be in any hurry to resolve the issue on their side.
```
Full thread: https://forum.obsidian.md/t/color-scheme-adapt-to-system-not-working-on-linux-fedora-gnome/38743/13
