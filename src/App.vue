<template>
  <div class="app">
    <Heading />
    <div class='console' :class='{unclickable: !isUnclick}' @click='e=>{const {tile} = e.target.dataset; if(tile) handleClick(tile)}'>
        <button class='button' data-tile='blue' id='blue'></button>
        <button class='button' data-tile='red' id='red'></button>
        <button class='button' data-tile='yellow' id='yellow'></button>
        <button class='button' data-tile='green' id='green'></button>
    </div>
    <div class="hidden">
      <audio src="https://s3.amazonaws.com/freecodecamp/simonSound3.mp3" data-sound="blue"></audio>
      <audio src="https://s3.amazonaws.com/freecodecamp/simonSound1.mp3" data-sound="red"></audio>
      <audio src="https://s3.amazonaws.com/freecodecamp/simonSound4.mp3" data-sound="yellow"></audio>
      <audio src="https://s3.amazonaws.com/freecodecamp/simonSound2.mp3" data-sound="green"></audio>
    </div>
    <div class='display'>
        <div class='round'>
            <h2>Уровень: {{level}}</h2>
        </div>
        <div class='start-button'>
            <button class='button-start' :class='{ hidden: isActive }' @click='startGame()' >Погнали!</button>
            <span class="info js-info" :class='{ hidden: !isActive }'>{{ info }}</span>
        </div>
    </div>
    <div class='options'>
        <h2>Уровень сложности:</h2>
        <input class='options-input' checked @input="setOptions($event.target.value)" type='radio' id='easy' name='options' value=1500>
        <label class='options-label' for='easy'>Легкий</label>
        <input class='options-input' @input="setOptions($event.target.value)" type='radio' id='normal' name='options' value=1000>
        <label class='options-label' for='normal'>Средний</label>
        <input class='options-input' @input="setOptions($event.target.value)" type='radio' id='hard' name='options' value=400>
        <label class='options-label' for='hard'>Сложный</label>
        <input class='options-input' @input="setOptions($event.target.value)" type='radio' id='extreme' name='options' value=200>
        <label class='options-label' for='extreme'>Экстримальный</label>
    </div>
  </div>
</template>

<script>
import Heading from './components/Heading.vue'

// logic:
// 1. there has to be generator that randomly chooses button and appends it to the array
// 2. player has to push on the same buttons, e.g. fill another array, so it is equevalent to the generated array
// 3. if player's and generator's arrays are equal, game is continued
// 4. if player's array differ from generator's game stops. 

export default {
  name: 'App',
  components: {
    Heading
  },
  data() {
    return {
      sequence: [],
      playerSequence: [],
      round: 0,
      isActive: false,
      level: 0,
      nextSequence: [],
      tiles: ['red','green','blue','yellow'],
      info: '',
      isUnclick: false,
      time: 1500
    }
  },
  methods: {
    setOptions(value) {this.time = value},
    activateTile(color) {
      const tile = document.querySelector(`[data-tile='${color}']`);
      const sound = document.querySelector(`[data-sound='${color}']`);
      tile.classList.add('activated');
      sound.play();
      setTimeout(()=>{
        tile.classList.remove('activated');
      }, 200);
    },
    playRound(nextSequence) {
      nextSequence.forEach((color,index) => {
        setTimeout(()=>{
          this.activateTile(color);
        }, (index +1) * this.time);
      });
    },
    nextStep() {
      const random = this.tiles[Math.floor(Math.random() * this.tiles.length)];
      return random;
    },
    nextRound() {
      this.level+=1;  
      this.isUnclick = false;
      this.info = 'следите за компьютером';  
      this.nextSequence = [...this.sequence];
      this.nextSequence.push(this.nextStep());
      this.playRound(this.nextSequence);
      this.sequence = [...this.nextSequence];
      setTimeout(() => {
        this.playerTurn(this.level);
      }, this.level * this.time + 500);
    },
    playerTurn() {
      this.isUnclick = !this.isUnclick;
      this.text = `ваш ход!`;
    },
    handleClick(tile) {
      const index = this.playerSequence.push(tile) - 1;
      const sound = document.querySelector(`[data-sound='${tile}']`);
      sound.play();
      const remainingTaps = this.sequence.length - this.playerSequence.length;
      if (this.playerSequence[index] !== this.sequence[index]) {
        this.resetGame(`Игра окончена! Вы дошли до ${this.level} уровня! Попробуйте снова!`);
        return;
      }
      if (this.playerSequence.length === this.sequence.length) {
        this.playerSequence = [];
        this.info = 'Супер, продолжаем!';
        setTimeout(() => {
          this.nextRound();
        }, 500);
        return;
      }
      this.info=`ваш ход: осталось ${remainingTaps} нажати${remainingTaps > 1 ? 'я' : 'е'}`
    },
    resetGame(text) {
      alert(text);
      this.sequence = [];
      this.level = 0;
      this.isActive = !this.isActive;
      this.info = '';
      this.isUnclick = true;
    },
    startGame() {
      this.isActive = !this.isActive;
      this.info = 'игра началась, следите за компьютером';
      this.sequence = [];
      this.playerSequence = [];
      this.nextSequence = [];
      this.nextRound();
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Langar&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@500&display=swap');
body{
  background-color: #ffc6ff;
}
h1{
  font-family: 'Langar', cursive;
  font-size: 50px;
}
h2{
  font-family: 'Roboto', sans-serif;
  font-size: 35px;
}
span, label{
  font-size: 20px;
}
.app{
  text-align: center;
}
.unclickable {
  pointer-events: none;
}
.hidden{
    display: none !important;
}
.console{
    width: 400px;
    display: inline-block;
}
.button{
    height: 200px;
    width: 200px;
    opacity: 0.5
}
.button:active{
    opacity: 1;
}
#red.activated{
  opacity: 1;
}
#blue.activated{
  opacity: 1;
}
#green.activated{
  opacity: 1;
}
#yellow.activated{
  opacity: 1;
}
#blue{
    background-color: #118ab2;
    border-radius: 50% 0 0 0;
    border: 5px solid black;
}
#red{
    background-color: #ef476f;
    border-radius: 0 50% 0 0;
    border: 5px solid black;
}
#yellow{
    background-color: #ffd166;
    border-radius: 0 0 0 50%;
    border: 5px solid black;
}
#green{
    background-color: #06d6a0;
    border-radius: 0 0 50% 0;
    border: 5px solid black;
}
</style>
