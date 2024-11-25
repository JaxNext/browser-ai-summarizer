<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { checkSummarizerUsability, setOriginTrialToken } from '@rejax/browser-ai'
import DarkMode from '@/components/DarkMode.vue'
import NotSupported from '@/components/NotSupported.vue'
import Summarizer from '@/components/Summarizer.vue'
import { SUMMARIZER_ORIGIN_TRIAL_TOKEN } from '@/constants/trial'

const isSupported = ref(false)

onMounted(async () => {
  const res = await checkSummarizerUsability()
  isSupported.value = res.available
  if (res.available && SUMMARIZER_ORIGIN_TRIAL_TOKEN) {
    setOriginTrialToken(SUMMARIZER_ORIGIN_TRIAL_TOKEN)
  }
})
</script>

<template>
  <div class="dark:bg-[rgb(2,8,23)] min-h-screen box-border p-6">
    <DarkMode class="fixed right-2 top-6" />
    <NotSupported v-if="!isSupported" />
    <Summarizer v-else />
  </div>
</template>