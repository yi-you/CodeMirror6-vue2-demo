<template>
  <div id="app">
    <div ref="editor" class="json-editor" :style="editorStyle" />
  </div>
</template>

<script>
import { EditorView } from '@codemirror/view'
import { EditorState } from '@codemirror/state'
import { basicSetup } from 'codemirror'
import { json } from '@codemirror/lang-json'
import { themesMap } from './editor-helper'
import { foldGutter } from '@codemirror/language'
export default {
  name: 'JsonEditor',
  props: {
    doc: { type: String, default: '' }, themeName: { type: String, default: 'dark' },
    editorStyle: {
      type: Object, default: () => ({ '--width': '100%', '--height': '400px' })
    }
  },
  data() {
    return {
      editor: null,
      view: undefined
    }
  },
  mounted() {
    const _state = this.createState(this.doc, json(), themesMap[this.themeName])
    const view = new EditorView({
      parent: this.$refs.editor,
      state: _state,
      dispatch: (tr) => {
        view.update([tr])
        if (tr.changes.empty) return
        const v = view.state.doc.toString()
        this.$emit('input', v)
      }
    })
    this.view = view
  },
  beforeDestroy() {
    this.view.destroy()
  },
  methods: {
    createState(value, language, theme) {
      const _foldGutter = foldGutter({ openText: '收起', closedText: '展开' })
      const _extensions = [language, theme, _foldGutter, EditorView.lineWrapping, ...basicSetup]
      return EditorState.create({
        doc: value,
        extensions: _extensions
      })
    },
    updateDoc(newDoc) {
      const transaction = this.view.state.update({ changes: { from: 0, to: this.view.state.doc.length, insert: newDoc }})
      this.view.dispatch(transaction)
    }
  }
}
</script>
<style scoped>
/deep/ .json-editor .cm-editor {
  border: solid;
  border-radius: .2rem;
  overflow: hidden;
  height: var(--height);
  width: var(--width);
  min-height: 30px;
  max-height: 600px;
  min-width: 100px;
  max-width: 100%;
  resize: both;
  font-size: 14px;
  font-variant-numeric: tabular-nums;
  font-family: 'Consolas, "Courier New", monospace'
}

</style>
