<template>
  <div id="app">
    <h1>Snake Game</h1>
    <button @click="startGame">开始游戏</button>
    <div id="game-board">
      <div
        v-for="cell in cells"
        :key="cell.id"
        :class="['cell', { 'snake': cell.isSnake, 'food': cell.isFood }]"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      gridSize: 20,
      snake: [{ x: 10, y: 10 }],
      food: { x: 5, y: 5 },
      direction: 'right',
      gameOver: false
    }
  },
  computed: {
    cells() {
      const cells = []
      for (let x = 0; x < this.gridSize; x++) {
        for (let y = 0; y < this.gridSize; y++) {
          const isSnake = this.snake.some(cell => cell.x === x && cell.y === y)
          const isFood = this.food.x === x && this.food.y === y
          cells.push({
            id: `${x}-${y}`,
            isSnake,
            isFood
          })
        }
      }
      return cells
    }
  },
  mounted() {
    // this.startGame()
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeydown)
  },

  methods: {
    startGame() {
      setInterval(() => {
        if (!this.gameOver) {
          this.moveSnake()
          this.checkCollision()
        } else {
          this.snake = [{ x: 10, y: 10 }]
        }
      }, 200)
    },
    moveSnake() {
      const head = { ...this.snake[0] }
      switch (this.direction) {
        case 'up':
          head.y--
          break
        case 'down':
          head.y++
          break
        case 'left':
          head.x--
          break
        case 'right':
          head.x++
          break
      }
      this.snake.unshift(head)
      if (head.x === this.food.x && head.y === this.food.y) {
        this.generateFood()
      } else {
        this.snake.pop()
      }
    },
    generateFood() {
      const x = Math.floor(Math.random() * this.gridSize)
      const y = Math.floor(Math.random() * this.gridSize)
      this.food = { x, y }
    },
    handleKeydown(event) {
      switch (event.keyCode) {
        case 87:
          this.direction = 'left'
          break
        case 65:
          this.direction = 'up'
          break
        case 83:
          this.direction = 'right'
          break
        case 68:
          this.direction = 'down'
          break
      }
    },
    checkCollision() {
      const head = this.snake[0]
      for (let i = 1; i < this.snake.length; i++) {
        if (head.x === this.snake[i].x && head.y === this.snake[i].y) {
          this.gameOver = true
          break
        }
      }
      if (
        head.x < 0 ||
            head.x >= this.gridSize ||
            head.y < 0 ||
            head.y >= this.gridSize
      ) {
        this.gameOver = true
        alert('游戏结束')
      }
    }
  }
}
</script>

<style  scoped>
#game-board {
      display: grid;
      grid-template-columns: repeat(20, 1fr);
      grid-template-rows: repeat(20, 1fr);
      width: 400px;
      height: 400px;
      border: 1px solid black;
    }

    .cell {
      background-color: #eee;
      border: 1px solid #ccc;
    }

    .snake {
      background-color: #333;
    }

    .food {
      background-color: red;
    }
</style>
