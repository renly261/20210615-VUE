<template lang="pug">
#list
  b-container
    b-row
      b-col(cols='12')
        //- 新增功能
        //- :state='state' 驗證是 bootstrap-vue 的用法, 'state' 是下面 methods 裡的 state function, 有 true false null
        b-form-group(label='新增事項' invalid-feedback='請至少輸入兩個字' :state='state')
          b-form-input(v-model='newitem' :state='state' trim @keydown.enter='additem')
        b-btn(variant='primary' @click='additem') 新增
        hr
        h1 待辦
        //- 編輯功能
        b-table(:items='list' :fields='listfields')
          template(#cell(name)='data')
            b-form-input(
              v-if='data.item.edit'
              v-model='data.item.model'
              trim
              :state='data.item.state'
              @keydown.enter='changelist(data.index)'
              @keydown.esc='cancellist(data.index)'
            )
            span(v-else) {{ data.value }}
          template(#cell(action)='data')
            //- 如果不是編輯狀態
            span(v-if='!data.item.edit')
              //- 如果不是編輯狀態, 顯示編輯按鈕
              //- @click='editlist(data.index)', data.index 編輯陣列裡第幾筆資料, 點下去執行 methods => editlist
              b-btn(@click='editlist(data.index)' variant='success')
                font-awesome-icon(:icon='["fas", "pen"]')
              //- 如果不是編輯狀態, 顯示刪除按鈕
              b-btn(@click='dellist(data.index)' variant='danger')
                font-awesome-icon(:icon='["fas", "trash"]')
            //-  如果是編輯狀態
            span(v-else)
              //- 如果是編輯狀態, 顯示確認編輯按鈕
              b-btn(@click='changelist(data.index)' variant='success')
                font-awesome-icon(:icon='["fas", "check"]')
              //- 如果是編輯狀態, 顯示取消編輯按鈕
              b-btn(@click='cancellist(data.index)' variant='danger')
                font-awesome-icon(:icon='["fas", "undo"]')
        h1 已完成
        b-table-simple
          thead
            tr
              th 名稱
              th 操作
          tbody
            tr(v-for='(item, idx) in finished' :key='idx')
              td {{ item }}
              td
                b-btn(@click='delfinish(idx)' variant='danger')
                  font-awesome-icon(:icon='["fas", "trash"]')
</template>

<script>
export default {
  name: 'List',
  data () {
    return {
      newitem: '',
      listfields: [
        { key: 'name', label: '名稱' },
        { key: 'action', label: '操作' }
      ]
    }
  },
  computed: {
    state () {
      if (this.newitem.length === 0) {
        return null
      } else if (this.newitem.length < 2) {
        return false
      } else {
        return true
      }
    },
    list () {
      return this.$store.getters.list.map(item => {
        if (item.model.length < 2) {
          item.state = false
        } else {
          item.state = true
        }
        return item
      })
    },
    finished () {
      return this.$store.state.finished
    }
  },
  methods: {
    additem () {
      if (this.state) {
        // 呼叫 VUEX => mutations => addList (state, data), data 就是這邊的 this.newitem
        // 呼叫 VUEX 的 mutations 然後把資料傳進去
        this.$store.commit('addList', this.newitem)
        // 新增完後清空內容, 方便下一次新增
        this.newitem = ''
      } else {
        this.$swal({
          icon: 'error',
          title: '錯誤',
          text: '必須要兩個字以上'
        })
      }
    },
    // 呼叫 VUEX => mutations => editList (state, data) {state.list[data].edit = true}
    editlist (index) {
      this.$store.commit('editList', index)
    },
    changelist (index) {
      console.log(index)
      // 如果 state 過才做修改, 一樣是呼叫 VUEX => changeList
      if (this.list[index].state) {
        this.$store.commit('changeList', index)
      }
    },
    cancellist (index) {
      // 呼叫 VUEX => cancelList
      this.$store.commit('cancelList', index)
    },
    // 呼叫 VUEX => delList
    dellist (index) {
      this.$store.commit('delList', index)
    },
    delfinish (index) {
      this.$store.commit('delFinish', index)
    }
  }
}
</script>
