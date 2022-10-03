---
title: Vue3使用tinymce富文本编辑器
date: 2022-9-30
tags: Vue
categories: Vue
---

### 简介

tinymce 富文本编辑器功能强大，效果如图

![8558467.jpg](https://szx-bucket1.fsh.bcebos.com/images2/Snipaste_2022-09-30_14-38-49.png)

### 下载依赖

首先安装 tinymce，这里安装指定版本，高版本会有兼容性问题

```sh
npm install tinymce@5.10.2
```

### 下载主题和汉化包

在 public 文件夹新建 resource 文件夹，在 resource 文件夹下再新建 langs 文件夹和 skins 文件夹

结构如图

![8558467.jpg](https://szx-bucket1.fsh.bcebos.com/images2/Snipaste_2022-09-30_14-42-37.png)

打开这个连接 https://gitee.com/shuiche/tinymce-vue3/blob/master/langs/zh_CN.js 下载出来 zh_CN.js 文件，放在 langs 文件夹下

然后再 node_modules 中找到 tinymce 文件夹，复制 skins 文件夹下的内容到上面新建的 skins 中

![8558467.jpg](https://szx-bucket1.fsh.bcebos.com/images2/Snipaste_2022-09-30_14-44-35.png)

### 新建Utils文件

在 src/utils 文件夹新建两个文件

- src/utils/tinymce.ts
- src/utils/onMountedOrActivated.ts

内容分别如下

```ts
// tinymce.ts

// ==== isNumber  函数====
const toString = Object.prototype.toString

export function is(val?: any, type?: any) {
    return toString.call(val) === `[object ${type}]`
}

export function isNumber(val?: any) {
    return is(val, 'Number')
}


// ==== buildShortUUID  函数====
export function buildShortUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
        var r = Math.random() * 16 | 0,
            v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

```ts
// ==== onMountedOrActivated  hook====
import {nextTick, onMounted, onActivated} from 'vue'

export function onMountedOrActivated(hook?: any) {
    let mounted: any
    onMounted(() => {
        hook()
        nextTick(() => {
            mounted = true
        })
    })
    onActivated(() => {
        if (mounted) {
            hook()
        }
    })
}
```

### 封装组件

在 src/components 文件夹下新建一个 tinymce 文件夹，里面新建三个文件：helper.js、tinymce.js、Tinymce.vue，文件内容分别如下

```js
// ==========helper.js==========
const validEvents = [
    'onActivate',
    'onAddUndo',
    'onBeforeAddUndo',
    'onBeforeExecCommand',
    'onBeforeGetContent',
    'onBeforeRenderUI',
    'onBeforeSetContent',
    'onBeforePaste',
    'onBlur',
    'onChange',
    'onClearUndos',
    'onClick',
    'onContextMenu',
    'onCopy',
    'onCut',
    'onDblclick',
    'onDeactivate',
    'onDirty',
    'onDrag',
    'onDragDrop',
    'onDragEnd',
    'onDragGesture',
    'onDragOver',
    'onDrop',
    'onExecCommand',
    'onFocus',
    'onFocusIn',
    'onFocusOut',
    'onGetContent',
    'onHide',
    'onInit',
    'onKeyDown',
    'onKeyPress',
    'onKeyUp',
    'onLoadContent',
    'onMouseDown',
    'onMouseEnter',
    'onMouseLeave',
    'onMouseMove',
    'onMouseOut',
    'onMouseOver',
    'onMouseUp',
    'onNodeChange',
    'onObjectResizeStart',
    'onObjectResized',
    'onObjectSelected',
    'onPaste',
    'onPostProcess',
    'onPostRender',
    'onPreProcess',
    'onProgressState',
    'onRedo',
    'onRemove',
    'onReset',
    'onSaveContent',
    'onSelectionChange',
    'onSetAttrib',
    'onSetContent',
    'onShow',
    'onSubmit',
    'onUndo',
    'onVisualAid'
]

const isValidKey = (key) => validEvents.indexOf(key) !== -1

export const bindHandlers = (initEvent, listeners, editor) => {
    Object.keys(listeners)
        .filter(isValidKey)
        .forEach((key) => {
            const handler = listeners[key]
            if (typeof handler === 'function') {
                if (key === 'onInit') {
                    handler(initEvent, editor)
                } else {
                    editor.on(key.substring(2), (e) => handler(e, editor))
                }
            }
        })
}
```

```js
// ==========tinymce.js==========
// Any plugins you want to setting has to be imported
// Detail plugins list see https://www.tinymce.com/docs/plugins/
// Custom builds see https://www.tinymce.com/download/custom-builds/
// colorpicker/contextmenu/textcolor plugin is now built in to the core editor, please remove it from your editor configuration

export const plugins = [
    'advlist anchor autolink autosave code codesample  directionality  fullscreen hr insertdatetime link lists media nonbreaking noneditable pagebreak paste preview print save searchreplace spellchecker tabfocus  template  textpattern visualblocks visualchars wordcount'
]

export const toolbar = [
    'fontsizeselect lineheight searchreplace bold italic underline strikethrough alignleft aligncenter alignright outdent indent  blockquote undo redo removeformat subscript superscript code codesample',
    'hr bullist numlist link  preview anchor pagebreak insertdatetime media  forecolor backcolor fullscreen'
]
```

```vue
<template>
  <div class="prefixCls" :style="{ width: containerWidth }">
        <textarea
            :id="tinymceId"
            ref="elRef"
            :style="{ visibility: 'hidden' }"
        ></textarea>
  </div>
</template>

<script setup>
import tinymce from 'tinymce/tinymce'
import 'tinymce/themes/silver'
import 'tinymce/icons/default/icons'
import 'tinymce/plugins/advlist'
import 'tinymce/plugins/anchor'
import 'tinymce/plugins/autolink'
import 'tinymce/plugins/autosave'
import 'tinymce/plugins/code'
import 'tinymce/plugins/codesample'
import 'tinymce/plugins/directionality'
import 'tinymce/plugins/fullscreen'
import 'tinymce/plugins/hr'
import 'tinymce/plugins/insertdatetime'
import 'tinymce/plugins/link'
import 'tinymce/plugins/lists'
import 'tinymce/plugins/media'
import 'tinymce/plugins/nonbreaking'
import 'tinymce/plugins/noneditable'
import 'tinymce/plugins/pagebreak'
import 'tinymce/plugins/paste'
import 'tinymce/plugins/preview'
import 'tinymce/plugins/print'
import 'tinymce/plugins/save'
import 'tinymce/plugins/searchreplace'
import 'tinymce/plugins/spellchecker'
import 'tinymce/plugins/tabfocus'
import 'tinymce/plugins/template'
import 'tinymce/plugins/textpattern'
import 'tinymce/plugins/visualblocks'
import 'tinymce/plugins/visualchars'
import 'tinymce/plugins/wordcount'
// import 'tinymce/plugins/table';

import { computed, nextTick, ref, unref, watch, onDeactivated, onBeforeUnmount, defineProps, defineEmits, getCurrentInstance } from 'vue'
import { toolbar, plugins } from './tinymce'
import { buildShortUUID,isNumber } from '/@/utils/tinymce.ts'
import { bindHandlers } from './helper'
import { onMountedOrActivated } from '/@/utils/onMountedOrActivated'


const props = defineProps({
  options: {
    type: Object,
    default: () => {}
  },
  value: {
    type: String
  },

  toolbar: {
    type: Array,
    default: toolbar
  },
  plugins: {
    type: Array,
    default: plugins
  },
  modelValue: {
    type: String
  },
  height: {
    type: [Number, String],
    required: false,
    default: 400
  },
  width: {
    type: [Number, String],
    required: false,
    default: 'auto'
  },
  showImageUpload: {
    type: Boolean,
    default: true
  }
})
const emits = defineEmits(['change', 'update:modelValue', 'inited', 'init-error'])
const { attrs } = getCurrentInstance()
const tinymceId = ref(buildShortUUID('tiny-vue'))
const containerWidth = computed(() => {
  const width = props.width
  if (isNumber(width)) {
    return `${width}px`
  }
  return width
})
const editorRef = ref(null)
const fullscreen = ref(false)
const elRef = ref(null)
const tinymceContent = computed(() => props.modelValue)

const initOptions = computed(() => {
  const { height, options, toolbar, plugins } = props
  const publicPath = '/'
  return {
    selector: `#${unref(tinymceId)}`,
    height,
    toolbar,
    menubar: 'file edit insert view format table',
    plugins,
    language_url: '/resource/tinymce/langs/zh_CN.js',
    language: 'zh_CN',
    branding: false,
    default_link_target: '_blank',
    link_title: false,
    object_resizing: false,
    auto_focus: true,
    skin: 'oxide',
    skin_url: '/resource/tinymce/skins/ui/oxide',
    content_css: '/resource/tinymce/skins/ui/oxide/content.min.css',
    ...options,
    setup: (editor) => {
      editorRef.value = editor
      editor.on('init', (e) => initSetup(e))
    }
  }
})

const disabled = computed(() => {
  const { options } = props
  const getdDisabled = options && Reflect.get(options, 'readonly')
  const editor = unref(editorRef)
  if (editor) {
    editor.setMode(getdDisabled ? 'readonly' : 'design')
  }
  return getdDisabled ?? false
})

watch(
    () => attrs.disabled,
    () => {
      const editor = unref(editorRef)
      if (!editor) {
        return
      }
      editor.setMode(attrs.disabled ? 'readonly' : 'design')
    }
)

onMountedOrActivated(() => {
  if (!initOptions.value.inline) {
    tinymceId.value = buildShortUUID('tiny-vue')
  }
  nextTick(() => {
    setTimeout(() => {
      initEditor()
    }, 30)
  })
})

onBeforeUnmount(() => {
  destory()
})

onDeactivated(() => {
  destory()
})

function destory () {
  if (tinymce !== null) {
    // tinymce?.remove?.(unref(initOptions).selector!);
  }
}

function initSetup (e) {
  const editor = unref(editorRef)
  if (!editor) {
    return
  }
  const value = props.modelValue || ''

  editor.setContent(value)
  bindModelHandlers(editor)
  bindHandlers(e, attrs, unref(editorRef))
}

function initEditor () {
  const el = unref(elRef)
  if (el) {
    el.style.visibility = ''
  }
  tinymce
      .init(unref(initOptions))
      .then((editor) => {
        emits('inited', editor)
      })
      .catch((err) => {
        emits('init-error', err)
      })
}

function setValue (editor, val, prevVal) {
  if (
      editor &&
      typeof val === 'string' &&
      val !== prevVal &&
      val !== editor.getContent({ format: attrs.outputFormat })
  ) {
    editor.setContent(val)
  }
}

function bindModelHandlers (editor) {
  const modelEvents = attrs.modelEvents ? attrs.modelEvents : null
  const normalizedEvents = Array.isArray(modelEvents) ? modelEvents.join(' ') : modelEvents

  watch(
      () => props.modelValue,
      (val, prevVal) => {
        setValue(editor, val, prevVal)
      }
  )

  watch(
      () => props.value,
      (val, prevVal) => {
        setValue(editor, val, prevVal)
      },
      {
        immediate: true
      }
  )

  editor.on(normalizedEvents || 'change keyup undo redo', () => {
    const content = editor.getContent({ format: attrs.outputFormat })
    emits('update:modelValue', content)
    emits('change', content)
  })

  editor.on('FullscreenStateChanged', (e) => {
    fullscreen.value = e.state
  })
}

function handleImageUploading (name) {
  const editor = unref(editorRef)
  if (!editor) {
    return
  }
  editor.execCommand('mceInsertContent', false, getUploadingImgName(name))
  const content = editor?.getContent() ?? ''
  setValue(editor, content)
}

function handleDone (name, url) {
  const editor = unref(editorRef)
  if (!editor) {
    return
  }
  const content = editor?.getContent() ?? ''
  const val = content?.replace(getUploadingImgName(name), `<img src="${url}"/>`) ?? ''
  setValue(editor, val)
}

function getUploadingImgName (name) {
  return `[uploading:${name}]`
}
</script>

<style lang="scss" scoped>
.prefixCls{
  position: relative;
  line-height: normal;
}
textarea {
  z-index: -1;
  visibility: hidden;
}
</style>
```

这里要注意下 Tinymce.vue 组件中的 tinymce.ts 和 onMountedOrActivated 导入的路径

### 使用 tinymce

```vue
<tinymce v-model="description" @change="onChagneThiymce" width="100%"/>
```

```js
import Tinymce from "/@/components/tinymce/Tinymce.vue"

const description = ref("")

// 富文本编辑器改变触发
const onChagneThiymce = (val) => {
  console.log(val)
  form.value.description = val
}
```
