<script setup lang="ts">
// @ts-nocheck
import About from "./about/about.md"

useHead({ title: "关于 | Pak" })
</script>

<template>
  <h1 class="mt-4 mb-8">
    关于
  </h1>
  <section id="md">
    <About />
  </section>
</template>

<route lang="yaml">
meta:
  layout: default
</route>
