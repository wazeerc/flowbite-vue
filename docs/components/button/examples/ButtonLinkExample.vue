<template>
  <div class="vp-raw inline-flex flex-wrap gap-2">
    <fwb-button
      color="default"
      href="https://google.com"
      target="_blank"
    >
      Default
    </fwb-button>
    <fwb-button
      href="/about"
      tag="router-link"
    >
      Router link
    </fwb-button>
  </div>
</template>

<script lang="ts" setup>
import { FwbButton } from '../../../../src/index'
</script>
