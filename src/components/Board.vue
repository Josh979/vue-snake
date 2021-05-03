<template>
  <div class="flex justify-center align-middle">
    <div v-if="active" class="board-wrapper">
      <div class="flex">
        <div v-for="(row) in config.rows" class="">
          <div v-for="(column) in config.columns" class="space">
            <div class="snake head" v-show="isSnakeSpace(row,column)">
<!--              {{row}}-->
            </div>
            <div class="snake trail" v-show="isSnakeTrailSpace(row,column)">
<!--              {{row}}-->
            </div>
            <div class="food" v-show="isFood(row,column)"></div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <h1 class="text-4xl my-10">Game Over</h1>
    </div>
  </div>
  <div>
    <div v-if="active" class="temp-controls">
      <button @click="moveLeft">Left</button>
      <button @click="moveUp">Up</button>
      <button @click="moveDown">Down</button>
      <button @click="moveRight">right</button>
      <button @click="spawnFood">Spawn Food</button>
    </div>
  </div>
  <div class="">
    SCORE: {{getScore}}
  </div>

</template>

<script>
export default {
  name: 'Board',
  props: {},
  data() {
    return {
      config:{
        rows: 30,
        columns: 30,
      },
      snake:{
        position:{
          x:1,
          y:1,
          trail:[],
        },
        speed:275,
        direction: 'right',
      },
      food:[],
      active: true,
      score: 0,
      movementInterval: null,
      oppositeDirectionMap:{
        left:'right',
        right:'left',
        up: 'down',
        down: 'up'
      }

    }
  },
  computed:{
    headPosition() {
      return {...this.snake.position};
    },
    trailPositions() {
      return [...this.snake.position.trail];
    },
    getScore(){
      return this.score;
    }
  },
  methods:{
    setMovementInterval(){
      if (this.active === false){
        return false;
      }
      if (this.movementInterval !== null){
        clearInterval(this.movementInterval)
      }
      this.movementInterval = setInterval(() => {
        this.moveDirection();
      }, this.snake.speed)
    },
    setSnakeSpeed(speed){
      if (speed >= 20){
        this.snake.speed = speed;
      }
    },
    move(x,y){
      const newX = this.headPosition.x + x;
      const newY = this.headPosition.y + y;

      if (this.isWithinBounds(newX,newY) && !(this.isSnakeTrailSpace(newX,newY))){
        this.snake.position.trail.unshift(this.createPositionNode(this.snake.position.x,this.snake.position.y))

        let food = (this.isFood(newX,newY,true));
        if (!food){
          this.snake.position.trail.pop();
        }
        this.snake.position.x += x;
        this.snake.position.y += y;
      } else {
        this.gameOver();
      }
    },
    setDirection(direction){
      if (direction !== this.oppositeDirectionMap[this.snake.direction]){
        this.snake.direction = direction
      }
    },
    moveDirection(){
      switch(this.snake.direction){
        case 'left':
          this.moveLeft();
          break;
        case 'right':
          this.moveRight();
          break;
        case 'up':
          this.moveUp();
          break;
        case 'down':
          this.moveDown();
          break;

      }
    },
    moveLeft(){
      this.move(-1,0)
    },
    moveRight(){
      this.move(1,0)
    },
    moveUp(){
      this.move(0,-1)
    },
    moveDown(){
      this.move(0,1)
    },
    spawnFood(){
      let xPos = Math.floor(Math.random() * this.config.columns);
      let yPos = Math.floor(Math.random() * this.config.rows);

      let spawnLocation = this.createPositionNode(xPos,yPos)
      spawnLocation.value = 100;
      this.food.push(spawnLocation);
    },
    addPoints(points){
      this.score += points;
    },
    gameOver() {
      this.active = false;
    },
    isWithinBounds(x,y){
      if (x < 1 || x > this.config.columns){
        return false;
      } else if (y < 1 || y > this.config.rows){
        return false;
      }
      return true;
    },
    isSnakeSpace(x,y){
      //snake head
      return ((this.snake.position.x === x && this.snake.position.y === y)) ;
    },
    isSnakeTrailSpace(x,y){
      return this.trailPositions.find((pos) => pos.x === x && pos.y === y );
    },
    isFood(x,y, remove = false){
      let index = this.food.findIndex((pos) => pos.x === x && pos.y === y );
      if (index !== -1){
        if (remove){
          this.addPoints(this.food[index].value);
          this.food.splice(index, 1);
        }
        return true;
      } else {
        return false
      }
    },
    createPositionNode(x,y){
      return {x,y}
    },
    handleKeydown(e) {
      switch (e.keyCode) {
        case 37:
          this.setDirection('left');
          break;
        case 38:
          this.setDirection('up');
          break;
        case 39:
          this.setDirection('right');
          break;
        case 40:
          this.setDirection('down');
          break;
      }
    }
  },
  created() {
    this.snake.position.x = Math.floor(this.config.rows / 4);
    this.snake.position.y = Math.floor(this.config.columns / 2);
    let startingPosition = this.snake.position;
    for (let i = -1; i > -3; --i){
      this.snake.position.trail.push(this.createPositionNode(startingPosition.x + i, startingPosition.y))
    }
    this.setMovementInterval();
    setInterval(() => {
      this.setSnakeSpeed(Math.ceil(this.snake.speed*0.95));
      this.setMovementInterval()
    }, 10000)
    setInterval(() => {
      this.spawnFood()
    }, 10000)


    window.addEventListener('keydown', this.handleKeydown, null);
  }
}
</script>

<style lang="scss">
.board-wrapper{
  border: 1px solid lime;
}
.space{
  position:relative;
  width: 25px;
  height: 25px;
  border:1px solid rgba(50, 205, 50, 0.25);
  display: flex;
  justify-content: center;
  align-content: center;
  align-items: center;

  .food{
    z-index: 1;
    background: lime;
    width:50%;
    height:50%;
    border-radius:100%;
    animation: pulse .75s infinite;
    box-shadow: 0 0 0 0 rgba(50, 205, 50, 0.5);

  }

}
.snake{
  &.head{
    background:limegreen;
    width:100%;
    height:100%;
    border-radius:100%;

  }
  &.trail{
    background: rgba(50, 205, 50, 0.57);
    width:100%;
    height:100%;
    border-radius:100%;

  }
}
.snake-body{
  color:black;
  background:limegreen;
}
.temp-controls{
  button{
    padding:.5rem 1rem;
    margin: 1rem;
    border:1px solid lime;
  }
}



</style>
