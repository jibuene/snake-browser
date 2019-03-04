<template>
  <div id="container">
    <h1>JIBsnake</h1>
    <h1>Score: {{score}} Highscore: {{highscore}}</h1>
    <button v-if="gameRunning == false" v-on:click="gameLoop()" > Start Game </button>
    <button v-on:click="stopLoop()">Stop Game </button>
    <v-stage ref="stage"
      :config="configKonva">
      <v-layer ref="layer">
        <v-rect :config="backgroundConfig"/>
        <v-rect :config="{
          x: foodX,
          y: foodY,
          width: 20,
          height: 20,
          fill: 'blue'
        }"/>
        <v-text ref="endText" v-on:click="snakeReset()" :config="canvasText"/>
        <v-rect v-for="location in snakeBody" :key="location.id"  :config="{
            x: location.x,
            y: location.y,
            width: 20,
            height: 20,
            fill: 'red'
          }"
        />
      </v-layer>
      <v-layer ref="dragLayer"></v-layer>
    </v-stage>
  </div>
</template>

<script>
/*eslint no-console: "off"*/
let vm = {};
const width = 1493;
const height = 700;

export default {
  name: 'HelloWorld',
  data() {
    return {
      score: 1,
      highscore: 0,
      endGame: false,
      gameRunning: false,
      currentlyMoving: false,
      foodY: this.calcFood(height),
      foodX: this.calcFood(width),
      snakeBody: [
        {x: 225, y: 150} /* TESTING
        {x: 200, y: 150},
        {x: 175, y: 150},
        {x: 150, y: 150},
        {x: 125, y: 150},
        {x: 100, y: 150},
        {x: 75, y: 150},
        {x: 50, y: 150},
        {x: 25, y: 150} */
      ],
      headOff: [],
      colors: ['red', 'blue', 'green'],
      xAxis: 25,
      yAxis: 0,
      backgroundConfig: {
        strokeWidth: 7,
        width: width,
        height: height,
        fill: 'black'
      },
      canvasText: {
        text: '',
        fontSize: 32,
        x: 300,
        y: 250,
        fill: 'gray'
      },
      configKonva: {
        width: width,
        height: height,
        fill: 'black'
      }
    }
  },
  methods: {
    gameLoop(){
      this.gameRunning = true
      const stage = vm.$refs.stage.getNode();
        this.intervall = setInterval(() => {
          this.snakePush(),
          stage.draw();
          this.currentlyMoving = false
        }, 80)
    },
    stopLoop(){
      clearInterval(this.intervall)
      this.gameRunning = false
    },
    checkSnakeCollide(item){
      return this.snakeBody.filter(obj => obj.x === item.x && obj.y === item.y)
    },
    findIndex(head){
      return this.snakeBody.findIndex(obj => obj.x === head.x && obj.y === head.y)
    },
    snakePush(){
      const head = {x: this.snakeBody[0].x + this.xAxis, y: this.snakeBody[0].y + this.yAxis}
      this.snakeBody.unshift(head)
      this.headOff = this.snakeBody.shift()
      console.log(this.findIndex(this.headOff))
      if (this.checkSnakeCollide(this.headOff).length) {
        this.snakeBody.splice(this.findIndex(this.headOff), this.snakeBody.length + 1)
      }
      this.snakeBody.unshift(this.headOff)
      if (this.snakeBody[0].x === this.foodX && this.snakeBody[0].y === this.foodY) {
        this.checkNewFood()
        this.score += 1
      } else {
        this.snakeBody.pop()
      }
      if (this.snakeBody[0].x < 0 || this.snakeBody[0].y < 0 || this.snakeBody[0].y > height - 25 || this.snakeBody[0].x > width - 15) {
        this.endGame = true
        const text = vm.$refs.endText.getNode();
        text.setAttrs({
          text: 'Le ePiC FaIl Press here to reset. Your score was: ' + this.score,
        })
        this.stopLoop()
      }
    },
    calcFood(max){
      return Math.round((Math.random() * (max - 0 - 25) + 0) / 25) * 25
    },
    checkNewFood(){
      this.foodY = this.calcFood(height)
      this.foodX = this.calcFood(width)
      const food = {x: this.foodX, y: this.foodY}
      if (this.checkSnakeCollide(food).length) {
        this.checkNewFood()
      }
    },
    snakeReset(){
      const text = vm.$refs.endText.getNode()
      text.setAttrs({
        text: ''
      })
      this.gameRunning = false
      this.xAxis = 25
      this.yAxis = 0
      this.checkNewFood()
      this.snakeBody = [
        {x: 225, y: 150},
        {x: 200, y: 150},
        {x: 175, y: 150},
        {x: 150, y: 150},
        {x: 125, y: 150},
        {x: 100, y: 150},
        {x: 75, y: 150},
        {x: 50, y: 150},
        {x: 25, y: 150}
      ],
      this.score > this.highscore ? this.highscore = this.score : false
      this.score = 0
    },
    updateCanvas(){
      const stage = vm.$refs.stage.getNode();
      stage.draw();
    },
  },
  created() {
    window.addEventListener('keydown', (e) => {
      if (this.currentlyMoving) return
      if (e.key == 'ArrowRight' && this.xAxis != -25) {
        this.xAxis = 25
        this.yAxis = 0
      }else if (e.key == 'ArrowLeft' && this.xAxis != 25) {
        this.xAxis = -25
        this.yAxis = 0
      }else if (e.key == 'ArrowUp' && this.yAxis != 25) {
        this.yAxis = -25
        this.xAxis = 0
      }else if (e.key == 'ArrowDown' && this.yAxis != -25) {
        this.yAxis = 25
        this.xAxis = 0
      }
      this.currentlyMoving = true
    });
  },
  mounted() {
    vm = this
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url('https://fonts.googleapis.com/css?family=Roboto');

#container {
  font-family: 'Roboto', sans-serif;
  margin: auto;
  width: 1500px;
  border-style: solid;
  border-color: #10ac84;
  background-color: #353b48;
  color: #227093;
}

div {
  border-style: solid;
  color: #10ac84;

}

.konvajs-content {
  padding-left: 0;
  padding-right: 0;
  margin-left: auto;
  margin-right: auto;
  display: block;
  width: 800px;
}

</style>
