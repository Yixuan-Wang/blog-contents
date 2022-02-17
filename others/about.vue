<script setup lang="ts">
// @ts-nocheck
const about = import.meta.globEager('./about/*.md')

const { t, locale } = useI18n()
const aboutComponent = computed(() => about[`./about/about-${locale.value}.md`].default)

const pageName = computed(() => t('about'))

useHead({ title: computed(() => `${pageName.value} | Pak`) })
</script>

<template>
  <h1 class="mt-4 mb-8">
    {{ pageName }}
  </h1>
  <section id="md">
    <component :is="aboutComponent" />
  </section>
</template>

<route lang="yaml">
meta:
  layout: default
</route>
