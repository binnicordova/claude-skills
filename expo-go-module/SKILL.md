---
name: expo-go-module
description: Create, test and prepare for npm an Expo module with no native code (pure TypeScript) that runs in Expo Go and ships over expo-updates on iOS, Android and Web. Use when asked to build an Expo module or library that must work in Expo Go or over the air, or to port a native React Native library to pure TypeScript.
---

# Expo Go–compatible module (pure TypeScript)

Use this when the user wants a new npm library for Expo that works in **Expo Go**, can be delivered **over the air with expo-updates / EAS Update**, and targets **iOS, Android and Web**, often "like react-native-X but without native code". The rule that makes that possible: **no Swift/Kotlin/ObjC/Java, and depend only on native modules that Expo Go already bundles**.

Reference implementation: [`expo-logs`](https://github.com/binnicordova/expo-logs) (local: `~/github/expo-logs`), a file logger ported from react-native-file-logger. Re-read its `src/`, `example/metro.config.js`, `package.json` and `README.md` when in doubt.

## Defaults for this author

- Author: `BinniCordova <binni.2000.cordova@gmail.com> (https://github.com/binnicordova)`; repo `https://github.com/binnicordova/<name>`; license MIT with `Copyright (c) <year>-present BinniCordova` (the template ships Expo's copyright, so replace it).
- Package manager: **bun**. Scaffold with `bun create expo-module`.
- Local files go in `~/github/<module-name>/` (a new directory). Test on the iOS Simulator or the connected Android device, in Expo Go.
- README must credit the author: BinniCordova.com, GitHub `binnicordova`, LinkedIn `in/binnicordova`, npm `~binnizenobiocordovaleandro`, Google Play `developer?id=BinniCordova.com`. Never put the phone number in a README.

## 1. Research before code

1. Current SDK: `npm view expo dist-tags --json`. If the user asks for an SDK that is only on `next` (preview), use `@next` for every Expo package and say so. Read `node_modules/expo/bundledNativeModules.json` for matching versions.
2. Confirm the npm name is free: `npm view <name>` should 404.
3. If there's a reference repo, read its public TS API (`src/index.ts`) and README, and keep the API compatible where reasonable (same method names, enum values, defaults). Document every deviation in a "Migrating from …" section.
4. List the native capabilities needed and map each one to a module **bundled in Expo Go** (expo-file-system, expo-sharing, expo-mail-composer, expo-constants, expo-crypto, expo-secure-store, expo-haptics, …). If something needs a module Expo Go doesn't include, stop and tell the user it can't be Expo Go–compatible.
5. **Never trust memory for Expo APIs.** Install the exact version and read its typings (`node_modules/<pkg>/build/*.d.ts`). APIs change between SDKs; for example, expo-file-system uses the `File`/`Directory`/`Paths` classes with `write(content, { append: true })`.

## 2. Scaffold

```sh
export PATH=$HOME/.npmg/bin:$PATH   # if bun had to be installed via: npm config set prefix ~/.npmg && npm i -g bun
CI=1 bun create expo-module@next <name> --name <PascalName> --description "..." \
  --package expo.modules.<short> --author-name BinniCordova --author-email binni.2000.cordova@gmail.com \
  --author-url https://github.com/binnicordova --repo https://github.com/binnicordova/<name> \
  --license MIT --module-version 0.1.0 --platform apple android web --features Function \
  --with-readme --with-changelog --package-manager bun
```

- Use **long flags only**: `bun create` swallows short ones (`-p` gives "Invalid Argument").
- Drop `@next` when the target SDK is `latest`.
- The template may fall back to an older root `expo` devDependency; bump the root devDependencies to the target SDK (`bun add -d expo@<v> jest-expo babel-preset-expo react react-native @types/react` plus the Expo modules you use).

Then strip native code:

```sh
rm -rf ios android expo-module.config.json example/ios example/android example/webpack.config.js example/LICENSE .npmignore src/*
```

Remove the `open:ios` / `open:android` scripts and `internal/module_scripts/open-*.js`, and remove `expo.autolinking` from `example/package.json`. With no `expo-module.config.json`, autolinking ignores the package, which is correct for pure JS.

## 3. Architecture patterns

- **Platform split with files, not `Platform.OS` branches:** `src/storage/index.ts` + `src/storage/index.web.ts`, imported as `'./storage'`. Metro resolves `index.web.ts` for web, and tsc emits both into `build/`.
- **Dependency injection for testability:** a small interface (such as `LogStorage`) with native, web and `MemoryStorage` implementations. The main class takes the adapter in its constructor; export a ready-made singleton plus the class.
- **Optional peer dependencies** (features only some apps need), loaded lazily:
  ```ts
  function loadSharing(): typeof import('expo-sharing') | null {
    try { return require('expo-sharing'); } catch { return null; }
  }
  ```
  Expo's Metro config sets `allowOptionalDependencies: true`, so this bundles even when the package isn't installed. Declare the package in `peerDependencies` plus `peerDependenciesMeta: { x: { optional: true } }`, and throw a typed error with a `code` (`ERR_X_MISSING`) that tells the user the `npx expo install` command.
- **Web:** expo-file-system and similar modules don't work on web. Provide a web implementation (localStorage/IndexedDB, Blob downloads, the Web Share API, `mailto:`), guard `typeof window` / `document` for SSR, and fall back to memory.
- **Never throw from background work.** Fire-and-forget promises go through `.catch()`; internal errors are reported through the *original* `console.warn`, and the code is guarded against re-entrancy if it patches `console`.
- Flush or persist on `AppState` changes to `background` / `inactive`.
- Avoid Node built-ins (`util`, `fs`, `path`, `Buffer`): write small helpers, or use pure-JS dependencies such as `fflate` for zip.
- Custom error class: `class ExpoXError extends Error { code: string }`.

## 4. Tests and verification (all must pass before calling it done)

```sh
npx tsc --noEmit && npx eslint src --fix && CI=1 npx jest
```

- Jest uses the `jest-expo` preset. Mock native modules at the top of tests: `jest.mock('expo-file-system', () => ({ Directory: jest.fn(), File: jest.fn(), Paths: {} }))`. Test the core with `MemoryStorage`, and the web adapter with a fake `Storage`, including the quota path.
- Use `jest.useFakeTimers({ now })` + `jest.setSystemTime` for date and timer logic.
- **Bundle check** in `example/`:
  ```sh
  CI=1 npx expo export --platform ios --platform android --platform web --no-bytecode --output-dir /tmp/dist
  ```
  `--no-bytecode` is needed on Linux arm, where hermesc fails. Grep the native bundles to confirm no web-only code leaked (for example `localStorage`), and that the optional modules are referenced.
- **Optional-dependency check:** temporarily move `expo-sharing` etc. out of both `node_modules` folders, export again, confirm it bundles, then move them back.
- `bun run prepare` + `npm pack --dry-run`: only `build/`, `src/` (without `__tests__`), README, CHANGELOG, LICENSE and package.json should ship.

## 5. Example app (Expo Go)

`example/package.json`: the dependencies at the SDK versions from `bundledNativeModules.json`, plus `react-native-safe-area-context`, `react-dom`, `react-native-web`, `@expo/metro-runtime`, `expo-status-bar`. Scripts: `"ios": "expo start --ios"`, `"android": "expo start --android"`, `"web": "expo start --web"` (Expo Go, not `expo run`).

`example/metro.config.js`, so the example uses `../src` directly and never duplicates react or expo:

```js
const { getDefaultConfig } = require('expo/metro-config');
const path = require('path');
const root = path.resolve(__dirname, '..');
const config = getDefaultConfig(__dirname);
config.watchFolders = [root];
config.resolver.disableHierarchicalLookup = true;
config.resolver.nodeModulesPaths = [path.resolve(__dirname, 'node_modules'), path.resolve(root, 'node_modules')];
const upstream = config.resolver.resolveRequest;
config.resolver.resolveRequest = (ctx, name, platform) =>
  name === '<pkg-name>'
    ? ctx.resolveRequest(ctx, path.join(root, 'src', 'index.ts'), platform)
    : (upstream ?? ctx.resolveRequest)(ctx, name, platform);
module.exports = config;
```

The app itself: one screen, a button for every public API, a live view of the result (files, state, output), a light/dark palette from `useColorScheme`, and text that wraps (no horizontal ScrollView) so screenshots read well. Configure the library in `index.ts` before `registerRootComponent`. Show API errors in an `Alert` with the error `code`.

## 6. package.json for publishing

- `main`/`types` → `build/index.js` / `build/index.d.ts`, `"sideEffects": false`, `"publishConfig": { "access": "public" }`.
- `"files": ["build", "src", "!src/**/__tests__", "!**/*.tsbuildinfo", "README.md", "CHANGELOG.md", "LICENSE"]`.
- `repository` as `{ type: 'git', url: 'git+https://github.com/binnicordova/<name>.git' }`, plus `bugs` and `homepage`.
- `peerDependencies`: `expo` (`>=<sdk>.0.0-0` if the SDK is a preview), each Expo module used, `react`, `react-native`; optional ones go in `peerDependenciesMeta`.
- Scripts: `typecheck`, and `"prepublishOnly": "bun run clean && CI=1 bun run test && bun run lint && bun run prepare"`.
- Keywords: `expo`, `expo-go`, `expo-updates`, `react-native`, `typescript`, `ios`, `android`, `web`, plus domain words and the reference library's name.

## 7. README structure

1. Hero image `docs/images/hero.png` + centered title + one-line pitch + badges (npm, license, platforms, Expo Go compatible, "native code: none").
2. "Created by **Binni Cordova**" line with BinniCordova.com · GitHub · LinkedIn · npm · Google Play.
3. **Get started in 3 steps** (`steps.png`): install (`npx expo install …`), configure, use.
4. **See it running**: a 3-column table of real Simulator screenshots with captions.
5. **Real use cases** (`use-cases.png`) + a bullet for each case naming the API it uses.
6. About / why Expo Go + OTA, Contents, Installation (required vs optional packages table), Quick start, Configuration table (option, type, default, description), API tables, error codes, internals, Platform notes (iOS, iOS Simulator has no Mail, Android, Web limits, **expo-updates: no fingerprint or runtime change**), Migrating from X, Example app, Contributing / publish, Author card, License.

Relative image paths are fine: npm rewrites them to GitHub raw URLs through the `repository` field. `docs/` stays out of the tarball.

## 8. Working on the user's Mac from a cloud session

- The device shell (`device_bash`) is a Linux VM that sees `~/github` under `$HOME/mnt/github`. Do installs and builds in a VM scratch copy (`$HOME/work/<name>`), then `rsync -a --exclude node_modules --exclude build --exclude .git --exclude .expo --exclude dist` into `~/github/<name>`. Never ship Linux `node_modules` into the Mac folder.
- **Request delete permission for the github folder before running git there**: without it, git leaves `.git/HEAD.lock` and `tmp_obj_*` files that break later git commands. Clean them up if that happens.
- Terminal and IDEs are click-only under computer use, so you can't type in them. Use AskUserQuestion to give the user one exact command to paste, e.g. `cd ~/github/<name> && bun install && cd example && bun install && bun run ios`, then wait.
- iOS Simulator (bundle `com.apple.iphonesimulator`, tier full): request full-screen control, since background clicks are unreliable. `cmd+s` saves a full-resolution screenshot to `~/Desktop` (request access to that folder to copy it), and `shift+cmd+a` toggles light/dark. Dismiss LogBox toasts before capturing.
- For README images, Gemini needs the user signed in to Google in whichever browser you drive. If the sign-in doesn't stick after one try, don't loop: render the images from HTML/CSS in the cloud container with Playwright (`executablePath: /opt/pw-browsers/chromium-*/chrome-linux/chrome`, `deviceScaleFactor: 2`, fonts from `@fontsource/inter` and `@fontsource/jetbrains-mono`, `font-variant-ligatures: none` for code). Composite the real screenshots in phone frames, look at every render and fix any overlap, then write the images into the repo with `device_commit_files`. Save the Gemini prompts in `docs/images/GEMINI_PROMPTS.md`.
- Commit with the user's name and email and the session's attribution lines, and don't push or publish unless asked. Finish by telling the user `npm login && npm publish`.

## Done checklist

- [ ] No `ios/`, `android/` or `expo-module.config.json`; only Expo Go–bundled native dependencies
- [ ] tsc, eslint and jest all green; iOS, Android and web bundles export; optional peer dependencies verified
- [ ] Ran in Expo Go on the iOS Simulator and/or Android; every example button exercised; say plainly anything not tested
- [ ] Screenshots + hero/steps/use-case images in `docs/images`, README complete, author section present
- [ ] LICENSE copyright updated, CHANGELOG entry, `npm pack --dry-run` contents correct, npm name free
- [ ] Git committed cleanly (no lock files); temporary files removed
