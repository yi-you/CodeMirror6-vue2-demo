<template>
  <div id="app">
    <select v-model="langType" @change="updateLanguage">
      <option v-for="option in Object.keys(sqlDialectMap)" :key="option" :value="option">
        {{ option }}
      </option>
    </select>
    <span>Selected: {{ langType }}</span>
    <div ref="editor" class="sql-editor" :style="editorStyle" />
  </div>
</template>

<script>
import { getLanguage, sqlDialectMap, themesMap } from './editor-helper'
import { EditorView } from '@codemirror/view'
import { Compartment, EditorState } from '@codemirror/state'
import { basicSetup } from 'codemirror'

export default {
  name: 'SqlEditor',
  props: {
    doc: { type: String, default: '' },
    themeName: { type: String, default: 'light' },
    dbInfo: {
      type: Object, default: () => ({
        schema: {},
        defaultSchema: undefined,
        defaultTable: undefined
      })
    },
    editorStyle: {
      type: Object, default: () => ({ '--width': '100%', '--height': '200px' })
    }
  },
  data() {
    return {
      editor: null,
      useIndent: true,
      languageConf: new Compartment(),
      view: undefined,
      langType: 'hivesql',
      sqlDialectMap: sqlDialectMap
    }
  },
  mounted() {
    const _language = this.languageConf.of(
      getLanguage(this.langType, this.dbInfo.schema, this.dbInfo.defaultSchema, this.dbInfo.defaultTable))
    const _state = this.createState(this.doc, _language, this.themeName)
    this.view = new EditorView({
      parent: this.$refs.editor,
      state: _state
    })
  },
  beforeDestroy() {
    // console.log('关闭sql-editor窗口')
    this.view.destroy()
  },
  methods: {

    /**
     * 动态更新language
     *
     * @returns {boolean}
     */
    updateLanguage() {
      const language = getLanguage(this.langType, this.dbInfo.schema, this.dbInfo.defaultSchema, this.dbInfo.defaultTable)
      this.view.dispatch({
        effects: this.languageConf.reconfigure(language)
      })
      return true
    },
    createState(value, language, themeName) {
      const _extensions = [language, themesMap[themeName], EditorView.lineWrapping, ...basicSetup]
      const listener = EditorView.updateListener.of((viewUpdate) => {
        // https://discuss.codemirror.net/t/codemirror-6-proper-way-to-listen-for-changes/2395/11
        if (viewUpdate.docChanged) {
          const newDoc = viewUpdate.state.doc.toString()
          this.$emit('input', newDoc)
        }
        if (viewUpdate.focusChanged) {
          viewUpdate.view.hasFocus ? this.$emit('focus') : this.$emit('blur')
        }
      })
      _extensions.push(listener)
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
/deep/ .sql-editor .cm-editor {
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
