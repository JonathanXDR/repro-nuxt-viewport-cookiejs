<script setup lang="ts">
const viewport = useViewport()
</script>

<template>
  <div>
    <h1>nuxt-viewport cookiejs reproduction</h1>
    <p>
      If you can read this in the browser, the bug did NOT reproduce.
      When the bug is present, the app never hydrates and the console shows:
    </p>
    <pre>SyntaxError: The requested module '/_nuxt/@fs/.../cookiejs/dist/cookie.js' does not provide an export named 'default'</pre>
    <p>Current breakpoint: {{ viewport.breakpoint }}</p>
  </div>
</template>
