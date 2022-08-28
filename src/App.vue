<template>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minesweeper</title>
  </head>
  <body>
    <div 
      class="settings"
      :style="cssOpenSettings">
      <button @click='select_beginner'>
        Я новичок!
      </button>
      <button @click='select_intermediate'>
        Я уже смешарик!
      </button>
      <button @click='select_expert'>
        Я робоняня!
      </button>
    </div>
    <div class="setting_menu">
      <a class="button_new_game"
        @click="fillField()"
        >Новая игра</a>

      <a class="button_setting"
        @click="openSettings()"
      >Настройки</a>

      <a class="button_statistics">Статистика</a>
    </div>

    <div 
      class="timer"
      :class='{moved_to_left: active_setting}'>
      <span>Времени прошло: {{ count_timer }}</span>
    </div>

    <div 
      class="field"
      :style="cssFieldGrid"
      :class='{moved_to_left: active_setting}'
    >
      <button 
        v-for="(cell,index) in field_array"
        :key="index"
        :style="{'background-color': cell.color}"
        @click="bombCheck(cell); timerStart()">{{ cell.count }}</button>
    </div>
  </body>
</template>

<script>




export default {
  name: 'App',
  data() {
    return {
      width: {
        'beginner': 9,
        'intermediate': 16,
        'expert': 30
      },
      height: {
        'beginner': 9,
        'intermediate': 16,
        'expert': 16
      },
      cells_count: 0,
      field_array: [],
      bombs_count: {
        'beginner': 10,
        'intermediate': 40,
        'expert': 99
      },
      selected_mode: 'beginner',
      bombs_id: [],
      count_timer: 0,
      isRunning: false,
      timer: null,
      active_setting: false
    }},
    computed: {
      cssFieldGrid() { 
        return {
          'grid-template-columns': 'repeat(' + this.width[this.selected_mode] + ', 40px)'
        }
      },
      cssOpenSettings() {
        let res = '-28%';
        if(this.active_setting===false) {
          res = '-150%'
        }
        return {
          'top': res
        }
      }
    },
    created() {
      this.fillField();//запуск создания поля после отрисовки страницы
    },
    methods: {
      fillField() {
        if(this.count_timer!==0) {
          this.isRunning=false
          clearInterval(this.timer)
          this.count_timer=0
        }//проверка для плавного обнуления таймера
        let bombs_count = this.bombs_count[this.selected_mode]//создание переменной с количеством бомб
        this.cells_count = this.width[this.selected_mode] * this.height[this.selected_mode];//подсчёт количества ячеек
        /*while(bombs_count !== 0) {
          this.bombs_id[bombs_count-1] = Math.round(Math.random() * (this.cells_count + 1))
          bombs_count--;
        }*///генерация id поля бомб
        this.bombs_id = new Set()
        while( this.bombs_id.size < bombs_count) {
          this.bombs_id.add(Math.ceil(Math.random() * (this.cells_count + 1)));
        }
        for(let i=0; i<this.cells_count; i++) {
          let standart_cell = {
            id: i,
            count: null,
            bomb: false,
            color: '#A8DD90'
          }
          this.field_array[i] = standart_cell;
        }
        for(let l = 0; l < this.bombs_count[this.selected_mode]; l++) {
          if(this.field_array[this.getValueByIndex(l)-1]!==undefined) {
            this.field_array[this.getValueByIndex(l)-1].bomb = true;
            this.field_array[this.getValueByIndex(l)-1].color = 'black';
          }
        }
        for(let k=0; k<this.cells_count; k++) {
          this.getDistanceToBomb(k);
        }
      },
      bombCheck (cell) {
        if(this.field_array[cell.id].bomb === true) {
          this.field_array[cell.id].color = '#d3d0d0bd';
        }
      },
      timerStart() {
        if(!this.isRunning) {
          this.isRunning = !this.isRunning
          this.timer = setInterval( () => {
            if(this.isRunning === false) {
              clearInterval(this.timer)
            }
            this.count_timer=this.count_timer+1;
          }, 1000 )
        }
      },
      openSettings() {
        this.isRunning=false;
        this.active_setting=!this.active_setting;
        if(this.active_setting===false&&this.count_timer!==0) {
          this.timerStart();
        }
      },
      getValueByIndex(index) {
        let res;
        let ind = 0;
        for (let value of this.bombs_id) {
          if (ind === index) {
            res = value;
            break;
          }
          ind++;
        }
          return res
      },
      getDistanceToBomb(id_cell) {
        if(this.field_array[id_cell-1]!==undefined&&id_cell%this.width[this.selected_mode]!==0) {
          if(this.field_array[id_cell-1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
              this.field_array[id_cell].count++;
            }
          }
        }
        if(this.field_array[id_cell+1]!==undefined&&id_cell%this.width[this.selected_mode]!==0) {
          if(this.field_array[id_cell+1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }

        if(this.field_array[id_cell-this.width[this.selected_mode]-1]!==undefined&&id_cell%this.width[this.selected_mode]!==0) {
          if(this.field_array[id_cell-this.width[this.selected_mode]-1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }
        if(this.field_array[id_cell-this.width[this.selected_mode]]!==undefined) {
          if(this.field_array[id_cell-this.width[this.selected_mode]].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }
        if(this.field_array[id_cell-this.width[this.selected_mode]+1]!==undefined) {
          if(this.field_array[id_cell-this.width[this.selected_mode]+1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }

        if(this.field_array[id_cell+this.width[this.selected_mode]-1]!==undefined&&id_cell%this.width[this.selected_mode]!==0) {
          if(this.field_array[id_cell+this.width[this.selected_mode]-1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }
        if(this.field_array[id_cell+this.width[this.selected_mode]]!==undefined) {
          if(this.field_array[id_cell+this.width[this.selected_mode]].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }
        if(this.field_array[id_cell+this.width[this.selected_mode]+1]!==undefined) {
          if(this.field_array[id_cell+this.width[this.selected_mode]+1].bomb===true&&this.field_array[id_cell].bomb!==true){
            if(this.field_array[id_cell].count===null) {
              this.field_array[id_cell].count=0;
            }
            this.field_array[id_cell].count++;
          }
        }
      },
      select_beginner() {
        this.selected_mode = 'beginner';
        this.field_array = [];
        this.bombs_id = [];
        this.fillField();
      },
      select_intermediate() {
        this.selected_mode = 'intermediate';
        this.field_array = [];
        this.bombs_id = [];
        this.fillField();
      },
      select_expert() {
        this.selected_mode = 'expert';
        this.field_array = [];
        this.bombs_id = [];
        this.fillField();
      }
  }
}
</script>

<style lang="scss">
  @import url("./assets/null.css");
  * {
    -ms-user-select: none;
    -moz-user-select: none;
    -khtml-user-select: none;
    -webkit-user-select: none;
  }
  ::-webkit-scrollbar {
    width: 0;
  }
  .body {
      max-width: 1440px;
      background: #d1d1d1;
  }
  .settings {
    position: absolute;
    z-index: 10;
    width: 33%;
    height: 50vh;
    background-color: white;
    border: 2px solid #000000;
    
    transition: 0.4s;
    margin: auto;
    right: 0;
    bottom: 0;
    left: 0;

    display: grid;
    justify-content: center;
    align-items: center;
    grid-template-columns: repeat(1, 50%);
    button {
      height: 10vh;
      background-color: white;
      border: 2px solid #000000;
    }
  }
  .setting_menu {
    display: flex;
    width: 53.055%;
    justify-content: space-around;
    margin: 22px auto 0;
    a {
      height: 4.4vh;
      width: calc(100%/4);
      border: 2px solid #000000;

      background:-webkit-linear-gradient(top,#d1d1d1,#7b7c7b); /* для Chrome и Safari */  
      color: #fff!important;  
      text-decoration: none!important;

      display: flex;
      justify-content: center;
      align-items: center;

      font-size: calc(20px + 16 * (100% / 1440));
    }
    a:active {
        background:-webkit-linear-gradient(bottom, #d1d1d1 5%,#7b7c7b); /* для Chrome и Safari */ 
        text-decoration: none;} 
    }
  .timer {
    display: flex;
    width: calc(100%/4.403);
    justify-content: space-around;
    margin: 32px auto;
    span {
      height: 4.4vh;
      width: 100%;
      background: #d1d1d1;
      border: 2px solid #000000;

      display: flex;
      justify-content: center;
      align-items: center;

      font-size: calc(20px + 16 * (100% / 1440));
    }
  }
  .field {
    display: grid;
    justify-content: center;
    button {
      transition: 0.1s;
      height: 40px;
      font-size: 24px;
      border: 2px solid #000000;
    }
  }
  .moved_to_left {
    justify-content: flex-start;
    margin-left: 20px;
  }
</style>
