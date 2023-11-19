<template>
  <div
    class="previewBox"
    :class="{ hide: hide, disabledEvents: disabledEvents }"
  >
    <iframe
      class="iframe"
      ref="iframeRef"
      :srcdoc="srcdoc"
      :style="iframeStyle"
    ></iframe>
  </div>
</template>

<script setup>
import {
  ref,
  defineProps,
  computed,
  onBeforeUnmount,
  getCurrentInstance,
  watch
} from 'vue'
import { useStore } from 'vuex'
import { assembleHtml, compile, compileVue } from '@/utils'
import { base } from '@/config'
import { defaultImportMapStr } from '@/config/constants'

const dev = process.env.NODE_ENV !== 'production'

// props
const props = defineProps({
  hide: {
    type: Boolean,
    default: false
  },
  scale: {
    type: Number,
    default: 1
  }
})

// hooks定义部分

// 初始化数据
const iframeRef = ref(null)
const useInitData = () => {
  const { proxy } = getCurrentInstance()
  // vuex
  const store = useStore()
  // 数据
  const editData = computed(() => store.state.editData)
  const isNewWindowPreview = ref(false)
  const newWindowPreviewData = ref(null)
  const htmlLanguage = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.HTML.language
      : editData.value.code.HTML.language
  })
  const jsLanguage = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.JS.language
      : editData.value.code.JS.language
  })
  const cssLanguage = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.CSS.language
      : editData.value.code.CSS.language
  })
  const htmlContent = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.HTML.content
      : editData.value.code.HTML.content
  })
  const jsContent = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.JS.content
      : editData.value.code.JS.content
  })
  const cssContent = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.CSS.content
      : editData.value.code.CSS.content
  })
  const cssResources = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.CSS.resources
      : editData.value.code.CSS.resources
  })
  const jsResources = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.JS.resources
      : editData.value.code.JS.resources
  })
  const importMap = computed(() => {
    return JSON.parse(
      (isNewWindowPreview.value
        ? newWindowPreviewData.value.code.JS.importMap
        : editData.value.code.JS.importMap) || defaultImportMapStr
    )
  })
  const vueLanguage = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.VUE.language
      : editData.value.code.VUE.language
  })
  const vueContent = computed(() => {
    return isNewWindowPreview.value
      ? newWindowPreviewData.value.code.VUE.content
      : editData.value.code.VUE.content
  })

  return {
    proxy,
    store,
    editData,
    isNewWindowPreview,
    newWindowPreviewData,
    htmlLanguage,
    jsLanguage,
    cssLanguage,
    htmlContent,
    jsContent,
    cssContent,
    cssResources,
    jsResources,
    importMap,
    vueLanguage,
    vueContent
  }
}

// 处理日志打印
const useLog = ({ proxy }) => {
  // 打印日志
  const log = (type, data) => {
    iframeRef.value.contentWindow.postMessage({
      type,
      data
    })
  }

  proxy.$eventEmitter.on('log', log)

  return {
    log
  }
}

// 处理生成html结构
// 2.TODOS

// 处理运行
// 1.TODOS

// 新开窗口预览模式处理
// 2.TODOS

// 处理拖动
// 3.TODOS

// 处理动态执行js
// 4.TODOS

// 缩放
// 5.TODOS

// created部分
const {
  proxy,
  store,
  editData,
  isNewWindowPreview,
  newWindowPreviewData,
  htmlLanguage,
  jsLanguage,
  cssLanguage,
  htmlContent,
  jsContent,
  cssContent,
  cssResources,
  jsResources,
  importMap,
  vueLanguage,
  vueContent
} = useInitData()
const { log } = useLog({ proxy })
const { createHtml } = useCreateHtml()
const { srcdoc, run, runStartTime } = useRun({
  store,
  isNewWindowPreview,
  newWindowPreviewData,
  editData,
  proxy,
  vueLanguage,
  vueContent,
  htmlLanguage,
  jsLanguage,
  cssLanguage,
  htmlContent,
  jsContent,
  cssContent,
  cssResources,
  jsResources,
  importMap,
  createHtml,
  log
})
useNewWindowPreview({
  newWindowPreviewData,
  isNewWindowPreview,
  run,
  runStartTime
})
const { disabledEvents } = useDrag({ proxy })
useDynamicRunJs({ proxy })
const { iframeStyle } = useScale()
</script>

<style scoped lang="less">
.previewBox {
  width: 100%;
  height: 100%;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background-color: #fff;

  &.hide {
    display: none;
  }

  &.disabledEvents {
    pointer-events: none;
  }

  .iframe {
    width: 100%;
    height: 100%;
    border: none;
    transform-origin: 0 0;
  }
}
</style>
