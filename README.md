# Bolt CEP

_A boilerplate for Adobe CEP Extensions built on Vite + React + TypeScript + Scss_

## What?

A Modern CEP Boilerplate.

Build your React CEP Panel app with HMR (Hot Module Reloading).

Adjust manifest settings in your cep.config.ts. Running in dev mode will hot-reload upon changes.

Build to create your consolidated production-ready CEP panel.

## Why?

While UXP is going to replace CEP in all Adobe apps at some point, we still need CEP panels in the meantime, also there should be a fair amount of overlap time between them in order for us to switch over.

For now, Vite development is one of the fastest available, and uses Rollup for build time to create super optmized bundle sizes.

## How?

Write in React and TypeScript start building your app like lightning!

---

## Quickstart

`yarn`

- Installs all dependencies

`yarn build`

- Runs initial build
- Creates cep folder structure
- Creates symlink to extensions folder

`yarn dev`

- Runs in dev mode with HMR Hot-reloading.
- Both JS and ExtendScript folders re-build on changes
- Viewable in browser via localhost:3000/panel/
  - (e.g. http://localhost:3000/main/, http://localhost:3000/settings/, etc.)

`yarn serve`

- Serve files after running `yarn build`
- Viewable in browser via localhost:5000/panel/
  - (e.g. http://localhost:5000/main/, http://localhost:5000/settings/, etc.)

`yarn debug && yarn tools`

- Adds snippet for debugging with React Dev Tools
- Launches standalone React Dev Tools

`yarn zxp`

- Builds and bundles your project into a zxp for publishing in the `dist/zxp` folder

---

## Config

Update your CEP build and package settings in `cep.config.ts` safely typed

Add your custom id in `src/js/shared.ts` as your scoped namepsace for ExtendScript

Start building your app in `src/js/main/index.tsx`

Write ExtendScript code in `src/jsx/main.ts`

---

## CEP Panel Structure

Each panel is treated as it's own page, with shared code for efficiency. The Boilerplate currently comes with 2 panels, `main` and `settings`. These are configured in the `cep.config.ts`.

Each panel can be edited in their respective folders:

```
src
 └─ js
    ├─ main
    │   ├─ index.html
    |   └─ index.tsx
    └─ settings
        ├─ index.html
        └─ index.tsx
```

To add panels, add an item to the panels object in `cep.config.ts`, and duplicate the folder structure and adjust as needed.

---

## ExtendScript

ExtendScript can be written in ES6 and will be compiled down to a single ES3 file for comaptibility.

JSON 2 is included by default, and any external libraries added with the include directive will be bundled as well:

```
// @include './lib/library.js'
```

App-speicific code is split into modules by the application's BridgeTalk name as seen in the `index.ts`.

```
aftereffects >> aeft.ts
illustrator >> ilst.ts
animate >> anim.ts
```

Write your app-speicifc functions in each of these separate modules, and they will be required per each application.

To add support for additional host apps:

- Add additional app module files (aeft.ts, anim.ts, etc).
- Extend the main `switch()` in `scr/jsx/index.ts` with your additional.
- Add the host to your `cep.config.ts` file.

---

## Calling ExtendScript from JS

As demonstrated in `main.tsx`, ExtendScript functions can be called with the `evalES()` function that will append your panel's namespace in the background to avoid namespace clashes.

```
console.log(await evalES(`helloWorld("${csi.getApplicationID()}")`));
```

If you wish to skip this scoping and call a global function directly, simply pass `true` to the second parameter:

```
evalES(`alert("Hello from ExtendScript :: " + app.appName + " " + app.version)`, true);
```

---

## GitHub Actions ZXP Releases

This repo comes with a configured GitHub Action workflow to build a ZXP and add to the releases each time a git tag is added.

```
git tag 1.0.0
git push origin --tags
```

Then your new build will be available under releases (e.g. https://github.com/hyperbrew/bolt-cep/releases)

---

## Troubleshooting Modules

Node.js Built-in modules can be imported from the `src/js/lib/node.ts` file.

```
import { os, path, fs } from "../lib/node";
```

To use 3rd party libraries, first attempt to use with the standard import syntax.

```
import { FaBolt } from "react-icons/fa";
```

If the import syntax fails, you can resort to the Node.js `require()` syntax,

```
const unzipper = require("unzipper");
```

but make sure to add those modules to the `installModules:[]` array inside your `cep.config.ts` file.

```
  installModules: ["unzipper"],
```

Also if they're Node.js-specific modules, it's best to place the requires inside functions so they are only required at runtime and don't break your panel when previewing in the browser.

---

## A Note on Routers

If you would like to set up a routing system like react-router, be aware that you'll have to make adjustments for CEP. React Router for instance bases the router path off of `window.location.pathname` which in the browser resolves to the page:

`/main/index.html`

yet in CEP context resolves to the full system path:

`file:///C:/Users/Username/AppData/Roaming/Adobe/CEP/extensions/com.bolt.cep/main/index.html`

To solve this, you'll need to adjust the router basename for each context, here is one way of accomplishing that with the panel named `main`:

```
const posix = (str: string) => str.replace(/\\/g, "/");

const cepBasename = window.cep_node
  ? `${posix(window.cep_node.global.__dirname)}/`
  : "/main/";

ReactDOM.render(
  <React.StrictMode>
    <Router basename={cepBasename}>
    [...]
    </Router>
  </React.StrictMode>,
  document.getElementById("root")
);
```

## Limitations

- Built for Adobe CC 2020 and up
