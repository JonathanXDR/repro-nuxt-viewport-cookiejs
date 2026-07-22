<script setup lang="ts">
const viewport = useViewport()

// Flips only when the client app actually mounts. When the cookiejs bug is
// present, the plugin import chain throws before Vue mounts, onMounted never
// runs, and the page keeps showing the server-rendered "NOT hydrated" state.
const hydrated = ref(false)
onMounted(() => {
  hydrated.value = true
})
</script>

<template>
  <div>
    <h1>nuxt-viewport cookiejs reproduction</h1>
    <p>
      This page is server-rendered, so you can always read this text.
      The bug affects client-side hydration only.
    </p>
    <p style="font-size: 1.25rem">
      Status:
      <strong v-if="hydrated" style="color: green">
        HYDRATED, the bug did not reproduce (workaround active?)
      </strong>
      <strong v-else style="color: red">
        NOT HYDRATED, bug reproduced. The browser console shows the cookiejs
        SyntaxError and this text never turns green.
      </strong>
    </p>
    <pre>SyntaxError: The requested module '/_nuxt/@fs/.../cookiejs/dist/cookie.js' does not provide an export named 'default'</pre>
    <p>Current breakpoint: {{ viewport.breakpoint }}</p>
  </div>
</template>
