<script setup>
import { ref,onMounted, onUpdated  } from 'vue';
import imageUrl from '@/assets/flappy_dunk.png'
const score = ref(0)
const highScore = ref(0)
const canvas = document.getElementById('gameCanvas');

const imgLoaded = ref(false)


const gameCanvas = ref(null)
const endMenu = ref(null)
const isFromStart = ref(true)
const ctx = ref(null)

// bird constant
const BIRD_SPEED = -5
const BIRD_WIDTH = 40
const BIRD_HEIGHT = 25
const PIPE_WIDTH = 50
const PIPE_GAP = 125

// bird movement
const birdX = ref(50)
const birdY = ref(50)
const birdVelocity = ref(0)
const birdyAcceleration = ref(0.1)

// pipe variable
const pipeX = ref(400)
const pipeY = ref(0)

// all flag
const scored = ref(false)
const isShowEndMenu = ref(false)
// const isHideEndMenu = ref(true)

onMounted(() => {
    ctx.value = gameCanvas.value.getContext("2d");
    console.log('imgLoaded',imgLoaded.value + ' new 2')
    
    pipeY.value = gameCanvas.value.height -200
    if(!isFromStart.value){
      loop()
    }
  });

document.body.onkeyup = function(e) {
    if (e.code == 'Space') {
        birdVelocity.value = BIRD_SPEED;
    }
}

function loop(){
  ctx.value.clearRect(0, 0,  gameCanvas.value.width,  gameCanvas.value.height);

  // // draw bird
  var bird = new Image();
  bird.onload = function() {
  }
  bird.src = '/src/assets/bird.png';
  ctx.value.drawImage(bird, birdX.value, birdY.value,50,50);
  bird.id = 'bird'
  
 
  ctx.value.fillStyle ='#333'
  ctx.value.fillRect(pipeX.value, -100, PIPE_WIDTH, pipeY.value)
  ctx.value.fillRect(pipeX.value, (pipeY.value + PIPE_GAP), PIPE_WIDTH,( gameCanvas.value.height - pipeY.value))

  //check collistion
  if(collisionCheck()){
    endGame()
    return;
  }

  //pipe movement
  pipeX.value -= 1.5

  //re-generate pipe
  if(pipeX.value < -50){
    pipeX.value = 400
    pipeY.value = Math.random() * ( gameCanvas.value.height - PIPE_GAP) + PIPE_WIDTH
  }

  //bird movement
  birdVelocity.value += birdyAcceleration.value
  birdY.value += birdVelocity.value

  increaseScore()
  window.requestAnimationFrame(loop);
}

function resetGame(){
  isFromStart.value = false
  isShowEndMenu.value = false
    birdX.value = 50;
    birdY.value = 50;
    birdVelocity.value = 0;
    birdyAcceleration.value = 0.1;

    pipeX.value = 400;
    pipeY.value = gameCanvas.value.height - 200;

    score.value = 0;
  loop()
}

function endGame(){
  isShowEndMenu.value = true
  if(highScore.value < score.value){
    highScore.value = score.value
  }
}
function increaseScore(){
  if(birdX.value < PIPE_WIDTH + pipeX.value){
    scored.value = false
  }

  if(birdX.value > PIPE_WIDTH + pipeX.value &&
    (birdY.value < PIPE_GAP +pipeY.value || birdY.value + BIRD_HEIGHT > PIPE_GAP +pipeY.value ) &&
    !scored.value){
      score.value++
      scored.value = true
  }
}

function collisionCheck(){
  const birdBox = ref({
    x: birdX.value,
    y: birdY.value,
    width: BIRD_WIDTH,
    height: BIRD_HEIGHT
  })

  const topPipeBox = ref({
    x: pipeX.value,
    y: pipeY.value - PIPE_GAP + BIRD_HEIGHT,
    width: PIPE_WIDTH,
    height: pipeY.value
  })

  const bottomPipeBox = ref({
    x: pipeX.value,
    y: pipeY.value + PIPE_GAP + BIRD_HEIGHT,
    width: PIPE_WIDTH,
    height: gameCanvas.value.height -pipeY.value -PIPE_GAP
  })

  //collistion on top
  if((birdBox.value.x + birdBox.value.width > topPipeBox.value.x) &&
     (birdBox.value.x <topPipeBox.value.width +topPipeBox.value.x )&&
     (birdBox.value.y < topPipeBox.value.y)){
      return true;
  }

  //collistion on bottom
  if((birdBox.value.x + birdBox.value.width > bottomPipeBox.value.x) &&
     (birdBox.value.x <bottomPipeBox.value.width +bottomPipeBox.value.x )&&
     (birdBox.value.y +birdBox.value.height > bottomPipeBox.value.y)){
      return true;
  }

  //bird hit edge
  if(birdY.value <0 ||birdY.value + BIRD_HEIGHT >  gameCanvas.value.height ){
    return true
  }

  return false

}


</script>

<template>
  <div ref="startMenu" class="start-menu text-center" v-if="isFromStart" :class="!isFromStart ? 'fadeOut':''">
    <h1>Flappy Bird</h1>
    <img src="/src/assets/bird.png" class="logo">
   
    <br>
    <button class="start-btn mt-2" @click="resetGame()">Start</button>
  </div>
  <div ref="endMenu" class="end-menu text-center" :class="isShowEndMenu ? 'end-menu-active' :'end-menu-inactive fadeOut'">
    <h1>Game Over!</h1>
    <p> Score: <span class="end-score">{{ score }}</span></p>
    <p> Best: <span class="Best-score">{{ highScore }}</span></p>
    <button class="restart-btn" @click="resetGame()">Restart</button>
  </div>
   <div ref="gameContainer" class="game-container"  :class="isShowEndMenu ? ' backdrop-blur' :''">
    <canvas ref="gameCanvas" id="gameCanvas" class="game-canvas" width="400" height="600"></canvas>
    <img src="/src/assets/flappy_dunk.png" style="display: none;">
    <div class="score-display text-center">{{ score }}</div>
  </div>
</template>

<style scoped>
.logo{
  width: 60%;
}
.start-menu{
  display: block;
  width: 300px;
  padding: 20px;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  position: absolute;
  left: 50%;
  transform: translate(-50%,50%);
  z-index: 2;
}
.end-menu{
  margin: 50px auto;
  width: 300px;
  padding: 20px;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  position: absolute;
  left: 50%;
  transform: translate(-50%,50%);
  z-index: 2;
}
.end-menu-active{
  display: block !important;
}
.end-menu-inactive{
  display: none !important;
}

.score-display{
  color: #fff;
  font-size: 2em;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}
.restart-btn,.start-btn{
  background-color: #2e2e2e;
  border: none;
  border-radius: 5px;
  color: #fff;
  cursor: pointer;
  font-size: 1em;
  padding: 15px 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.25s ease;
}

.restart-btn:hover,.start-btn:hover{
  background-color: #333;
  transform: translateY(-2px);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}
</style>