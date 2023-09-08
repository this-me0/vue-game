<template>
  <div class="hello">
    <div>
      请选择你的模式 ：
      <button :disabled="!isShow" @click="chooseType(1)">系统随机</button>
      <button :disabled="!isShow" @click="chooseType(2)">玩家自选</button>
      <div>
        请输入人机出手速度
        <input v-model="stepNum" min="500" type="number" placeholder="默认100毫秒" @blur="setNum">
      </div>
    </div>
    <transition name="fade">
      <div v-if="whoWin!==''" class="whoWin">
        {{ whoWin }}
      </div>
    </transition>

    <div v-if="chooseMode" class="body">
      <div class="name">
        <div>对手</div>
        <div>自己</div>
      </div>
      <div class="box">
        <div class="left">
          {{ leftChoose.name }}
        </div>
        <div class="center">
          <button :disabled="isStart" @click="start">{{ status }}</button>
        </div>
        <div v-if="chooseMode === 2" class="right">
          <span v-if="num">{{ num == 1?'石头':num == 2?'剪刀':'布' }}</span>
          <div v-if="!num">
            <button @click="choose(1)">石头</button>
            <button @click="choose(2)">剪刀</button>
            <button @click="choose(3)">布</button>
          </div>
        </div>
        <div v-else class="right">
          <span>{{ rightChoose.name }}</span>
        </div>
      </div>
    </div>
    <div>
      <div>总获胜次数:{{ winNum }}</div>
      <div>连胜次数:{{ wins }}</div>
      <div>最高连胜次数:{{ mostWins }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data() {
    return {
      leftData: [
        { name: '石头', value: 1 },
        { name: '剪刀', value: 2 },
        { name: '布', value: 3 }
      ],
      leftChoose: { name: '', value: null },
      rightChoose: { name: '', value: null },
      status: '开始',
      timer: null,
      whoWin: '',
      chooseMode: null,
      isShow: true,
      num: null,
      winNum: 0,
      price: 0,
      isStart: false,
      wins: 0,
      mostWins: 0,
      stepNum: 100
    }
  },
  mounted() {

  },
  methods: {
    left() {
      this.leftChoose = this.leftData[Math.floor(Math.random() * this.leftData.length)]
    },

    right() {
      if (this.chooseMode == 1) {
        this.rightChoose = this.leftData[Math.floor(Math.random() * this.leftData.length)]
      }
    },
    start(num) {
      if (this.status == '开始') {
        this.status = '结束'
        this.isShow = false
        this.whoWin = ''
        this.num = null
        if (this.chooseMode == 2) {
          this.isStart = true
          this.rightChoose.value = null
        }
        this.timer = setInterval(() => {
          this.left()
          this.right()
        }, this.stepNum)
      } else {
        this.status = '开始'
        this.isShow = true
        clearInterval(this.timer)
        this.timer = null
      }

      if (!this.timer) {
        const { value: leftValue } = this.leftChoose
        const { value: rightValue = num } = this.rightChoose

        if (leftValue === rightValue) {
          this.whoWin = '平局'
          this.wins = 0
        } else if ((leftValue === 1 && rightValue === 2) || (leftValue === 2 && rightValue === 3) || (leftValue === 3 && rightValue === 1)) {
          this.whoWin = '你输了'
          this.wins = 0
        } else {
          this.whoWin = '你赢了'
          this.winNum++
          this.wins += 1
          this.mostWins = this.wins
        }
        setTimeout(() => {
          this.whoWin = ''
        }, 2000)
      }
    },
    choose(num) {
      if (this.status == '结束') {
        this.start(num)
        this.num = num
        this.isStart = true
        setTimeout(() => {
          this.isStart = false
        }, 2000)
      } else {
        this.whoWin = '请先开始游戏'
      }
    },
    setNum() {
      if (this.stepNum < 100) {
        this.stepNum = 100
        this.whoWin = '最小输入100毫秒'
      } else {
        this.whoWin = ''
      }
    },
    chooseType(num) {
      this.chooseMode = num
      this.leftChoose = { name: '', value: null }
      this.rightChoose = { name: '', value: null }
    }
  }
}
</script>
<style scoped>
.hello {
  position: relative;
  width: 400px;
  height: 300px;
  background-color: pink;
  padding: 10px;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.box {
  line-height: 100px;
  display: flex;
  align-items: center;
}
.left {
  flex: 1;
}
.center {
  flex: 1;
}
.right {
  flex: 1;
}
.name {
  display: flex;
  justify-content: space-between;
  padding:  0 30px;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.whoWin {
  position: absolute;
  top: calc(20%);
  color: #fff;
  font-size: 30px;
  width: 100%;
  text-align: center;
}
.body {
  flex: 1;
  margin-top: 30px;
}
</style>
