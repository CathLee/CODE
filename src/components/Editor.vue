<!--
 * @Author: cathylee 447932704@qq.com
 * @Date: 2023-01-11 20:10:04
 * @LastEditors: cathylee 447932704@qq.com
 * @LastEditTime: 2023-02-02 20:43:20
 * @FilePath: /codeRun/src/components/Editor.vue
 * @Description: 
 * 
 * Copyright (c) 2023 by cathylee 447932704@qq.com, All Rights Reserved. 
-->
<template>
  <div class="editorBox" :class="{ hide: hide }">
    <Drag v-if="show" :number="editorItemList.length" :dir="dir">
      <DragItem
        v-for="(item, index) in editorItemList"
        :key="item.title"
        :index="index"
        :disabled="item.disableDrag"
        :showTouchBar="item.showTouchBar"
        :title="item.showTitle ? item.title : ''"
      >
        <EditorItem
          :title="item.title"
          :language="item.language"
          :codeTheme="codeTheme"
          :codeFontSize="codeFontSize"
          :content="item.content"
          :preprocessorList="preprocessorListMap[item.title]"
          :showAddBtn="item.showAddBtn"
          :dir="dir"
          :showAllAddResourcesBtn="['vue2', 'vue3'].includes(item.language)"
          :showHeader="showHeader"
          :readOnly="readOnly"
          @code-change="
            code => {
              codeChange(item, code)
            }
          "
          @preprocessor-change="
            p => {
              preprocessorChange(item, p)
            }
          "
          @add-resource="
            languageType => {
              addResource(languageType || item.title)
            }
          "
          @add-importmap="addImportmap(item)"
          @space-change="
            noSpace => {
              item.showTitle = noSpace
            }
          "
          @create-code-img="
            editor => {
              showCreateCodeImg(editor, item)
            }
          "
        ></EditorItem>
      </DragItem>
    </Drag>
    <!-- 资源管理弹窗 -->
    <EditAssets ref="EditAssetsComp"></EditAssets>
    <!-- 生成代码图片 -->
    <CodeToImg
      ref="CodeToImgComp"
      :getThemeData="getThemeData"
      :codeTheme="codeTheme"
      :codeFontSize="codeFontSize"
    ></CodeToImg>
    <!-- importmap编辑 -->
    <EditImportMap
      :codeTheme="codeTheme"
      :codeFontSize="codeFontSize"
    ></EditImportMap>
  </div>
</template>

<script setup>
import {
  ref,
  defineProps,
  onMounted,
  computed,
  getCurrentInstance,
  watch,
  onUnmounted
} from 'vue'
import { useStore } from 'vuex'
const props = defineProps({
  // 是否隐藏编辑器
  hide: {
    type: Boolean,
    default: false
  },
  // 排布方向
  dir: {
    type: String,
    default: 'h' // v（垂直）、h（水平）
  },
  // 要显示的编辑器列表
  showList: {
    type: Array,
    default() {
      return ['HTML', 'CSS', 'JS']
    } // 目前共有四种编辑器：'HTML'、 'CSS'、 'JS'、 'VUE'
  },
  // 是否要显示编辑器的头部
  showHeader: {
    type: Boolean,
    default: true
  },
  // 不要触发代码运行
  notRunCode: {
    type: Boolean,
    default: false
  },
  // 编辑器只读
  readOnly: {
    type: Boolean,
    default: false
  }
})

// hook定義部分
// 初始化數據
const useInit = () => {
  const store = useStore()
  const proxy = getCurrentInstance().proxy
  const editData = computed(() => store.state.editData)
  const codeTheme = computed(() => store.state.editData.config.codeTheme)
  const codeFontSize = computed(() => store.state.editData.config.codeFontSize)
  return {
    store,
    proxy,
    editData,
    codeTheme,
    codeFontSize
  }
}

// creatd部分
const { store, proxy, editData, codeTheme, codeFontSize } = useInit()
</script>

<style lang="less" scoped>
.editorBox {
  width: 100%;
  height: 100%;
  display: flex;

  &.hide {
    display: none;
  }
}

/deep/ .el-dialog__body {
  padding: 20px;
}
</style>
