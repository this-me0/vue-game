<template>
  <div>
    <div>
      <input v-model="wh" name="wh" type="number" />
      <button @click="reset">重置</button>
    </div>

    <canvas
      id="checkerboard"
      :width="wh"
      :height="wh"
      @click="handlerClick"
    ></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      wh: 500, // 宽高
      interval: 0, // 间隔
      init_X_Y: 10, // 初始点
      r: 12, // 半径
      points: [], // 所有点位
      exist_point: [], // 存在点位（弃用）
      operate: 'user', // 用户或者电脑 computer
      direction: [
        // 方位
        'up',
        'down',
        'left',
        'right',
        'left_up',
        'right_down',
        'left_down',
        'right_up'
      ],
      state: 0 // 0初始1用户赢2电脑赢
    }
  },
  computed: {
    // 未使用棋子位置
    pointDiff() {
      return this.points.filter((e) => !e.exist)
    },
    // 用户点位
    userPoint() {
      console.log(this.points.filter((e) => (e.operate == 'user')), '用户点位')
      return this.points.filter((e) => (e.operate == 'user'))
    },
    computerPoint() {
      return this.points.filter((e) => (e.operate == 'computer'))
    }
  },
  watch: {
    operate: {
      handler(newVal, oldVal) {
        if (newVal == 'computer') {
          setTimeout(() => {}, 200)
        }
      },
      immediate: true
    },
    state(newVal) {
      if (newVal != 0) newVal == 1 ? alert('用户胜利') : alert('电脑胜利')
    },
    wh: {
      handler(newVal) {
        console.log(newVal, '------')
        this.interval = (newVal - this.init_X_Y * 2) / 15
        console.log(this.interval)
        setTimeout(() => {
          this.reset()
        }, 100)
      },
      immediate: true
    }
  },
  mounted() {
    this.mspaint()
    this.getAllPoint()
  },
  methods: {
    handlerClick(e) {
      // 用户点击棋盘
      if (this.state != 0) return
      if (this.operate == 'computer') return

      for (const item of this.points) {
        // 点击可下区域周围生成棋子
        if (
          Math.abs(e.offsetX - item.x) < this.r &&
            Math.abs(e.offsetY - item.y) < this.r &&
            !item.exist
        ) {
          const canvas = document.getElementById('checkerboard')
          const g2 = canvas.getContext('2d')
          g2.beginPath()
          g2.fillStyle = 'white' //
          g2.arc(item.x, item.y, this.r, 0, 2 * Math.PI)
          g2.fill()
          this.operate = 'computer'
          item.exist = true
          item.operate = 'user'
          console.log(item)
          this.win(item, 'user')
          if (this.state == 0) this.computerMspaint()
          break
        }
      }
    },
    // 电脑下
    computerClick() {
      this.exist_point // 已经存在的棋子
      this.points // 棋盘点位
    },
    // 绘制棋盘
    mspaint() {
      const interval = this.interval // 间隔
      const init_X_Y = this.init_X_Y // 初始xy
      const r = this.r // 半径
      const canvas = document.getElementById('checkerboard')
      const g2 = canvas.getContext('2d')

      for (let i = 0; i < 16; i++) {
        const line_X_Y = init_X_Y + i * interval // 线条XY
        // 竖线
        g2.beginPath()
        g2.strokeStyle = 'black'
        g2.lineWidth = 2
        g2.moveTo(line_X_Y, 10)
        g2.lineTo(line_X_Y, this.wh - 10)
        g2.closePath()
        g2.stroke()
        // 横线
        g2.beginPath()
        g2.strokeStyle = 'black'
        g2.lineWidth = 2
        g2.moveTo(10, line_X_Y)
        g2.lineTo(this.wh - 10, line_X_Y)
        g2.closePath()
        g2.stroke()
      }
    },
    // 获取所有点位
    getAllPoint() {
      this.points = []
      for (let i = 0; i < 16; i++) {
        // 遍历X轴
        const x = i * this.interval + this.init_X_Y
        for (let j = 0; j < 16; j++) {
          // 遍历Y轴
          const y = j * this.interval + this.init_X_Y
          this.points.push({ x, y, exist: false })
        }
      }
      console.log(this.points)
    },
    // 计算点位权重
    checkPoint() {
      // 检查点位权重
      let maxWeight = 0 // 连续三个+50分 连续四个+99 低于三个各方位每个加1
      let final = {}
      let lr, ud, lu_rd, ld_ru
      for (const item of this.pointDiff) {
        const userWeight = this.direction.map((e) => {
          const arr = this.bearingFor(item, e, 'user')
          let decrement = 0
          // 用户有两个棋子相连
          if (arr >= 2) {
            // 有电脑方棋子挡路权重-20
            const arrLeft = this.bearingFor(arr[0], e, 'computer').length
            const arrRight = this.bearingFor(
              arr[arr.length - 1],
              e,
              'computer'
            ).length
            if (arrLeft > 0) decrement += 20
            if (arrRight > 0) decrement += 20
          }
          return this.getCount(arr.length, decrement)
        })
        // 上下权重值
        ud = userWeight[0] + userWeight[1]
        // 左右权重值
        lr = userWeight[2] + userWeight[3]
        // 左上到右下
        lu_rd = userWeight[4] + userWeight[5]
        // 左下到右上
        ld_ru = userWeight[6] + userWeight[7]

        if (ud > maxWeight) {
          maxWeight = ud
          final = item
        } else if (lr > maxWeight) {
          maxWeight = lr
          final = item
        } else if (lu_rd > maxWeight) {
          maxWeight = lu_rd
          final = item
        } else if (ld_ru > maxWeight) {
          maxWeight = ld_ru
          final = item
        }
      }

      for (const item of this.pointDiff) {
        const userWeight = this.direction.map((e) => {
          const arr = this.bearingFor(item, e, 'computer')
          let decrement = 0
          if (arr >= 2) {
            const arrLeft = this.bearingFor(arr[0], e, 'user').length
            const arrRight = this.bearingFor(
              arr[arr.length - 1],
              e,
              'user'
            ).length
            if (arrLeft > 0) decrement += 20
            if (arrRight > 0) decrement += 20
          }
          return this.getCount(arr.length, decrement)
        })
        ud = userWeight[0] + userWeight[1]
        lr = userWeight[2] + userWeight[3]
        lu_rd = userWeight[4] + userWeight[5]
        ld_ru = userWeight[6] + userWeight[7]
        if (ud >= maxWeight) {
          maxWeight = ud
          final = item
        } else if (lr >= maxWeight) {
          maxWeight = lr
          final = item
        } else if (lu_rd >= maxWeight) {
          maxWeight = lu_rd
          final = item
        } else if (ld_ru >= maxWeight) {
          maxWeight = ld_ru
          final = item
        }
      }
      return final
    },
    // 电脑绘制棋子
    computerMspaint() {
      const final = this.checkPoint()
      const canvas = document.getElementById('checkerboard')
      const g2 = canvas.getContext('2d')
      g2.beginPath()
      g2.fillStyle = 'black' //
      g2.arc(final.x, final.y, this.r, 0, 2 * Math.PI)
      g2.fill()
      this.points.forEach((e) => {
        if (e.x == final.x && e.y == final.y) {
          e.exist = true
          e.operate = 'computer'
        }
      })
      this.operate = 'user'
      this.win(final, 'computer')
    },

    /** 用户附近可下点位**/
    userNearbyPoint() {
      const userPoint = this.points.filter((e) => {
        return e.operate == 'user'
      })
      const arr = []
      for (const item of userPoint) {
        const right = this.points.find((e) => {
          return (
            e.x == item.x + this.interval && e.y == item.y && e.exist == false
          )
        })
        if (right) arr.push(right)

        const right_up = this.points.find((e) => {
          return (
            e.x == item.x + this.interval &&
              e.y == item.y - this.interval &&
              !e.exist
          )
        })
        if (right_up) arr.push(right_up)

        const right_down = this.points.find((e) => {
          return (
            e.x == item.x + this.interval &&
              e.y == item.y + this.interval &&
              !e.exist
          )
        })
        if (right_down) arr.push(right_down)

        const left = this.points.find((e) => {
          return e.x == item.x - this.interval && e.y == item.y && !e.exist
        })
        if (left) arr.push(left)

        const left_up = this.points.find((e) => {
          return (
            e.x == item.x - this.interval &&
              e.y == item.y - this.interval &&
              !e.exist
          )
        })
        if (left_up) arr.push(left_up)

        const left_down = this.points.find((e) => {
          return (
            e.x == item.x - this.interval &&
              e.y == item.y + this.interval &&
              !e.exist
          )
        })
        if (left_down) arr.push(left_down)

        const up = this.points.find((e) => {
          return e.x == item.x && e.y == item.y - this.interval && !e.exist
        })
        if (up) arr.push(up)

        const down = this.points.find((e) => {
          return e.x == item.x && e.y == item.y + this.interval && !e.exist
        })
        if (down) arr.push(down)
      }
      return arr
    },
    // 地柜八个方向棋子
    bearingFor(item, direction, operate) {
      // 寻找八个方位棋子
      let obj = []
      let dirPoint = {}
      // 方位寻找
      switch (direction) {
        case 'up':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x &&
                e.y == item.y - this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'up', operate)]
          }
          break
        case 'down':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x &&
                e.y == item.y + this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'down', operate)]
          }
          break
        case 'left':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x - this.interval &&
                e.y == item.y &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'left', operate)]
            if (operate == 'user') { console.log(obj, 'obj') }
          }
          break
        case 'right':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x + this.interval &&
                e.y == item.y &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'right', operate)]
          }
          break
        case 'right_up':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x + this.interval &&
                e.y == item.y - this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'right_up', operate)]
          }
          break
        case 'right_down':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x + this.interval &&
                e.y == item.y + this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [
              dirPoint,
              ...this.bearingFor(dirPoint, 'right_down', operate)
            ]
          }
          break
        case 'left_up':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x - this.interval &&
                e.y == item.y - this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [dirPoint, ...this.bearingFor(dirPoint, 'left_up', operate)]
          }
          break
        case 'left_down':
          dirPoint = this.points.find((e) => {
            return (
              e.x == item.x - this.interval &&
                e.y == item.y + this.interval &&
                e.exist &&
                e.operate == operate
            )
          })
          if (dirPoint != null) {
            obj = [
              dirPoint,
              ...this.bearingFor(dirPoint, 'left_down', operate)
            ]
          }
          break
      }
      return obj
    },
    // 返回权重
    getCount(len, decrement = 0) {
      if (len == 3) return 50 - decrement
      if (len == 4) return 70 - decrement
      if (len >= 5) return 999 - decrement
      return len
    },
    // 胜利判断
    win(item, flag) {
      this.direction.forEach((e) => {
        if (flag == 'user') {
          console.log(this.userPoint)
          this.points.filter((e) => (e.operate == 'user')).forEach((user) => {
            const arr = this.bearingFor(user, e, flag)
            console.log(e, arr)
            if (arr.length == 4) {
              if (flag == 'user') this.state = 1
              else this.state = 2
              const canvas = document.getElementById('checkerboard')
              const g2 = canvas.getContext('2d')

              g2.beginPath()
              g2.strokeStyle = '#fc011a'
              g2.lineCap = 'round'
              g2.lineWidth = 8
              g2.moveTo(item.x, item.y)
              g2.lineTo(arr[arr.length - 1].x, arr[arr.length - 1].y)
              g2.closePath()
              g2.stroke()
            }
          })
        } else {
          this.points.filter((e) => (e.operate == 'computer')).forEach((computer) => {
            const arr = this.bearingFor(computer, e, flag)
            if (arr.length == 4) {
              if (flag == 'user') this.state = 1
              else this.state = 2
              const canvas = document.getElementById('checkerboard')
              const g2 = canvas.getContext('2d')

              g2.beginPath()
              g2.strokeStyle = '#fc011a'
              g2.lineCap = 'round'
              g2.lineWidth = 8
              g2.moveTo(item.x, item.y)
              g2.lineTo(arr[arr.length - 1].x, arr[arr.length - 1].y)
              g2.closePath()
              g2.stroke()
            }
          })
        }
      })
    },
    // 重置
    reset() {
      const canvas = document.getElementById('checkerboard')
      const g2 = canvas.getContext('2d')
      g2.clearRect(0, 0, this.wh, this.wh)
      this.mspaint()
      this.getAllPoint()
      this.state = 0
      this.operate = 'user'
    }
  }
}
</script>

  <style>
  #checkerboard {
    background-color: #e4caa6;
  }
  </style>

