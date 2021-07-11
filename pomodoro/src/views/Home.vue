<template lang="pug">
#home
  b-container
    b-row
      b-col(cols='12')
        //- 目前顯示的東西
        h1 {{ currentText }}
        //- 目前倒數剩餘時間
        h2 {{ timeText }}
        //- 若下面 computed 的 status() 呼叫 VUEX 的 this.$store.state.status !== 1, 顯示開始按鈕
        b-btn(variant='primary' v-if='status !== 1' @click='start')
          font-awesome-icon(:icon='["fas", "play"]')
        //- 若下面 computed 的 status() 呼叫 VUEX 的 this.$store.state.status !== 1, 顯示暫停按鈕
        b-btn(variant='primary' v-if='status === 1' @click='pause')
          font-awesome-icon(:icon='["fas", "pause"]')
        //- 若下面 computed 的 status() 呼叫 VUEX 的 this.$store.state.current > 0, 顯示跳過按鈕
        //- 代一個參數 true 或 false 進去判斷是時間到的 finish 還是跳過的 finish
        b-btn(variant='primary' v-if='current.length > 0' @click='finish(true)')
          font-awesome-icon(:icon='["fas", "step-forward"]')
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      // setIterval
      timer: 0
    }
  },
  computed: {
    status () {
      return this.$store.state.status
    },
    list () {
      return this.$store.state.list
    },
    current () {
      return this.$store.state.current
    },
    // 目前倒數的東西
    currentText () {
      let text = this.current
      // 如果目前沒有顯示東西
      if (text.length === 0) {
        // 如果目前沒有顯示東西, 如果目前待辦清單也是空的
        if (this.list.length === 0) {
          // 目前顯示沒有事項
          text = '沒有事項'
          // 如果目前沒有顯示東西, 如果待辦清單有東西
        } else {
          text = '點擊開始'
        }
      }
      return text
    },
    // 倒數計時剩幾秒的資料存在 VUEX, 呼叫 VUEX => state => timeleft
    timeleft () {
      return this.$store.state.timeleft
    },
    // 顯示倒數時間
    timeText () {
      // Math.floor() 無條件捨去
      // 目前剩幾分鐘
      const m = Math.floor(this.timeleft / 60)
      // 除以 60 取餘數, 目前剩幾秒
      const s = Math.floor(this.timeleft % 60)
      // 把分秒組成字串顯示
      return `${m} : ${s}`
    }
  },
  methods: {
    // 暫停
    pause () {
      // 暫停倒數
      clearInterval(this.timer)
      // 將 VUEX 的 this.$store.state.status 改成 2 (暫停)
      this.$store.commit('changeStatus', 2)
    },
    // 開始倒數
    start () {
      // 若 computed => status() 呼叫 VUEX 的 this.$store.state.status !== 2 (暫停), 且清單裡面有東西
      if (this.status !== 2 && this.list.length > 0) {
        // 呼叫 VUEX => start
        this.$store.commit('start')
      }
      // 若 current 目前顯示有東西
      if (this.current.length > 0) {
        // 將 VUEX => changeStatus (state, data) {state.status = data} 的 data 改為 1 (倒數中)
        this.$store.commit('changeStatus', 1)
        // 倒數
        this.timer = setInterval(() => {
          // 呼叫 VUEX => countdown (state) {state.timeleft--}, 每過一秒 VUEX state.timeleft -1
          this.$store.commit('countdown')
          if (this.timeleft <= -1) {
            // 自然倒數結束, 非跳過
            this.finish(false)
          }
        }, 1000)
      }
    },
    // 跳過, 非自然倒數結束
    finish (skip) {
      // 暫停倒數
      clearInterval(this.timer)
      // 將 VUEX => changeStatus (state, data) {state.status = data} 的 data 改為 0
      this.$store.commit('changeStatus', 0)
      this.$store.commit('addFinish')

      // 若自然倒數結束
      if (!skip) {
        // 播放音效
        const audio = new Audio()
        audio.src = require('../assets/' + this.$store.state.sound)
        audio.play()
      }

      // 若結束後清單還有東西
      if (this.list.length > 0) {
        // 繼續倒數
        this.start()
        // 若結束後清單沒有東西
      } else {
        // 顯示結束
        this.$swal({
          icon: 'success',
          title: '結束'
        })
      }
    }
  }
}
</script>
