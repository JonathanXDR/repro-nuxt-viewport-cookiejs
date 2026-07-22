# repro-nuxt-viewport-cookiejs

Minimal reproduction: **nuxt-viewport breaks Nuxt dev mode under Nuxt 4.5 / Vite 8** because its runtime default-imports [`cookiejs`](https://github.com/jaywcjlove/cookie.js), whose `browser` field points at a UMD build with no ES module exports.

## Reproduction steps

```bash
npm install
npm run dev
```

Open the app in a browser. The page never hydrates and the console shows:

```
Uncaught (in promise) SyntaxError: The requested module
'/_nuxt/@fs/…/node_modules/cookiejs/dist/cookie.js' does not provide
an export named 'default' (at manager.js:1:8)
```

Or verify without a browser, straight from the Vite transform pipeline:

```bash
curl -s "http://localhost:3000/_nuxt/@fs$PWD/node_modules/nuxt-viewport/dist/runtime/manager.js" \
  | grep cookiejs
```

Bug present, the import points at the raw UMD file:

```
import cookie from "/_nuxt/@fs/…/node_modules/cookiejs/dist/cookie.js?v=…"
```

Fixed (with the workaround below), the import points at the prebundled interop wrapper:

```
import __vite__cjsImport0_cookiejs from "/_nuxt/@fs/…/node_modules/.cache/vite/deps/cookiejs.js?v=…"
```

## Root cause

- `nuxt-viewport/dist/runtime/manager.js` does `import cookie from "cookiejs"`.
- `cookiejs` has no `exports` map. Vite's client-side resolution picks its `browser` field, which is `dist/cookie.js`, a UMD wrapper with no ESM exports.
- The import lives in Nuxt module runtime code served via `@fs`, so Vite's dependency scanner never discovers `cookiejs` for prebundling.
- Vite 8 (bundled since Nuxt 4.5.0) serves the file raw without CJS interop, so the default import fails at parse time. The same setup worked under Nuxt 4.4 / Vite 7.

## Workaround

Uncomment in `nuxt.config.ts`:

```ts
vite: {
  optimizeDeps: {
    include: ['cookiejs'],
  },
},
```

## Suggested fix

nuxt-viewport can register the dependency for prebundling in its module setup:

```ts
nuxt.options.vite.optimizeDeps ||= {}
nuxt.options.vite.optimizeDeps.include ||= []
nuxt.options.vite.optimizeDeps.include.push('cookiejs')
```

Alternatively: import `cookiejs/dist/cookie.esm.js` in the runtime, or an upstream `exports` map in cookiejs.

## Environment

| Package       | Version |
| ------------- | ------- |
| nuxt          | 4.5.0   |
| nuxt-viewport | 2.4.0   |
| cookiejs      | 2.1.3   |

Production builds are unaffected. Dev mode only.
