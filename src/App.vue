<template>
  <div class="editor-container">
    <button @click="changeDb">修改数据库</button>
    <sql-editor ref="sqlEditor" v-model="value" :db-info="dbInfo" />
    <json-editor ref="jsonEditor" v-model="jsonValue" />
  </div>
</template>

<script>
import SqlEditor from '@/components/SqlEditor'
import JsonEditor from '@/components/JsonEditor'

const jsonData = 'select 1;\n' +
    'select approx_count_distinct(t2.col2_1) from default.table2 as t2;'
const schema = {
  'hue.table1': [
    { label: 'col1_1', detail: '字段11' },
    { label: 'col1_2', detail: '字段12' }
  ],
  'default.table2': [
    { label: 'col2_1', detail: '字段21' },
    { label: 'col2_2', detail: '字段22' }
  ],
  'default.table3': [
    { label: 'col3_1', detail: '字段31' },
    { label: 'col3_2', detail: '字段32' }
  ]
}
const dbInfo1 = {
  schema: schema,
  defaultSchema: 'hue',
  defaultTable: 'table1'
}
export default {
  name: 'App',
  components: {
    SqlEditor, JsonEditor
  },
  data() {
    return {
      dbInfo: dbInfo1,
      value: jsonData.toString(),
      jsonValue: JSON.stringify(dbInfo1, null, '\t')
    }
  },
  created() {
    this.$nextTick(() => {
      this.$refs.sqlEditor.updateDoc(this.value)
      this.$refs.jsonEditor.updateDoc(this.jsonValue)
    })
  },
  methods: {
    changeDb() {
      const newSchema = {
        'oracle.new_table1': [
          { label: 'new_col1_1', detail: 'new_字段11' },
          { label: 'new_col1_2', detail: 'new_字段12' }
        ],
        'default.new_table2': [
          { label: 'new_col2_1', detail: 'new_字段21' },
          { label: 'new_col2_2', detail: 'new_字段22' }
        ],
        'default.new_table3': [
          { label: 'new_col3_1', detail: 'new_字段31' },
          { label: 'new_col3_2', detail: 'new_字段32' }
        ]
      }
      const dbInfo2 = {
        schema: newSchema,
        defaultSchema: 'oracle',
        defaultTable: 'new_table1'
      }
      debugger
      if (this.dbInfo !== dbInfo1) {
        this.dbInfo = dbInfo1
      } else {
        this.dbInfo = dbInfo2
      }
      this.jsonValue = JSON.stringify(this.dbInfo, null, '\t')
      this.$nextTick(() => {
        this.$refs.sqlEditor.updateLanguage()
        this.$refs.jsonEditor.updateDoc(this.jsonValue)
      })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
