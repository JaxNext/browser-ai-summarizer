<template>
  <div class="top-row flex items-center mb-[24px]">
    <div class="title text-4xl font-bold text-center text-black dark:text-slate-50">浏览器内置 AI 摘要</div>
    <div class="options-row ml-auto mr-[48px] flex items-center gap-4 justify-center">
      <Select v-model="format" @update:model-value="handleOptionsChange">
        <SelectLabel class="text-black dark:text-slate-50">输入格式</SelectLabel>
        <SelectTrigger class="w-[120px] text-black dark:text-slate-50">
          <SelectValue placeholder="选择格式" />
        </SelectTrigger>
        <SelectContent>
          <SelectGroup>
            <SelectItem
              v-for="option in formatOptions"
              :key="option.value"
              :value="option.value"
            >{{ option.label }}</SelectItem>
          </SelectGroup>
        </SelectContent>
      </Select>
      <Select v-model="type" @update:model-value="handleOptionsChange">
        <SelectLabel class="text-black dark:text-slate-50">摘要模式</SelectLabel>
        <SelectTrigger class="w-[100px] text-black dark:text-slate-50">
          <SelectValue placeholder="选择模式" />
        </SelectTrigger>
        <SelectContent>
          <SelectGroup>
            <SelectItem
              v-for="option in typeOptions"
              :key="option.value"
              :value="option.value"
            >{{ option.label }}</SelectItem>
          </SelectGroup>
        </SelectContent>
      </Select>
      <Select v-model="length" @update:model-value="handleOptionsChange">
        <SelectLabel class="text-black dark:text-slate-50">生成长度</SelectLabel>
        <SelectTrigger class="w-[80px] text-black dark:text-slate-50">
          <SelectValue placeholder="选择长度" />
        </SelectTrigger>
        <SelectContent>
          <SelectGroup>
            <SelectItem
              v-for="option in lengthOptions"
              :key="option.value"
              :value="option.value"
            >{{ option.label }}</SelectItem>
          </SelectGroup>
        </SelectContent>
      </Select>
      <Button class="bg-sky-500 dark:bg-sky-800 dark:text-slate-50 dark:hover:bg-sky-900" @click="copy(result)">复制结果</Button>
    </div>
  </div>
  <div class="panel-row flex h-[calc(100vh-124px)] text-black dark:text-slate-50">
    <div class="panel-item flex-1 mr-[6px]">
      <Textarea
        v-model="input"
        placeholder="输入文本（目前支持英文）"
        class="h-full overflow-auto whitespace-pre-line"
        @input="handleInput"
      />
    </div>
    <div class="sign h-full flex items-center justify-center">=></div>
    <div class="panel-item flex-1 ml-[6px]">
      <div class="result-text h-full overflow-auto border border-gray-200 rounded-md p-4 text-black dark:text-slate-50 whitespace-pre-line">
        {{ result }}
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Textarea } from '@/components/ui/textarea'
import {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectLabel,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'
import { Button } from '@/components/ui/button'
import { genSummarizer, summarizeStreaming } from '@rejax/browser-ai'
import { useClipboard } from '@vueuse/core'

const { copy } = useClipboard()

const format = ref('plain-text')
const formatOptions = [
  { label: '纯文本', value: 'plain-text' },
  { label: 'Markdown', value: 'markdown' },
]
const type = ref('tl;dr')
const typeOptions = [
  { label: '总结', value: 'tl;dr' },
  { label: '关键点', value: 'key-points' },
  { label: '预告', value: 'teaser' },
  { label: '标题', value: 'headline' },
]
const length = ref('short')
const lengthOptions = [
  { label: '短', value: 'short' },
  { label: '中', value: 'medium' },
  { label: '长', value: 'long' },
]

const input = ref('')
const result = ref('')

let timer: NodeJS.Timeout | null = null
const handleInput = () => {
  if (!input.value) {
    result.value = ''
    return
  }
  if (timer) clearTimeout(timer)
  timer = setTimeout(async () => {
    await getResult()
  }, 1000)
}

interface Summarizer {
  summarize: (text: string, options?: any) => Promise<string>;
  summarizeStreaming: (text: string, options?: any) => Promise<string>;
}

const summarizer = ref<Summarizer | null>(null)

async function handleOptionsChange() {
  result.value = ''
  if (!input.value) return
  await createSummarizer()
  await getResult()
}

async function createSummarizer() {
  summarizer.value = await genSummarizer({
    format: format.value,
    type: type.value,
    length: length.value,
  })
}

async function getResult() {
  const res = await summarizeStreaming(input.value)
  for await (const chunk of res) {
    result.value = chunk
  }
}

onMounted(async () => {
  if (!format.value || !type.value || !length.value) return
  await createSummarizer()
})
</script>
