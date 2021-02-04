<template>
  <span v-on:submit.prevent> 
      <button class="playlist-add-button" @click="setTime">Set Timer</button>
      <div v-if="enterPass" class="timer-Area">
          <input v-model="search_password_entered" @keyup.enter="checkPass" v-if="enterPass" type="password" placeholder="Enter Password To unlock">
      </div>
      <div class="timerOptionsList" v-if="allowSetTime">
            <ul v-for="timerOption in timerOptions" :key="timerOption.show">
                <li @click="selectTimer(timerOption.valueInMin)">{{ timerOption.show }}</li>
            </ul>
          </div>

        <div v-if="showTimer" class="playlist-add-button timerClock">
            <span>{{ houre }}</span>
            <span>:</span>
            <span>{{ minutes }}</span>
            <span>:</span>
            <span>{{ seconds }}</span>
        </div>
  </span>
</template>

<script>
    import axios  from 'axios'
    
export default {
name: 'setTime',
data: () => {
    return {
        search_password_entered: '',
        allowSetTime: false,
        enterPass: false,
        search_password: null,
        timerOptions: [{show: 'Clear Timer', valueInMin: 0},
                       {show: '15 min', valueInMin: 15},
                       {show: '30 min', valueInMin: 30},
                       {show: '45 min', valueInMin: 45},
                       {show: '1 hr', valueInMin: 60},
                       {show: '1 hr 15 min', valueInMin: 75},
                       {show: '1 hr 30 min', valueInMin: 90},
                       {show: '1 hr 45 min', valueInMin: 105},
                       {show: '2 hr', valueInMin: 120},
                       {show: '2 hr 15 min', valueInMin: 135},
                       {show: '2 hr 30 min', valueInMin: 150},
                       {show: '2 hr 45 min', valueInMin: 165}
                       ],
        timer: null,
        timerTime: null,
        showTimer: false
        }
},
mounted() {
    this.setTimerForVideoPayer()
	let self = this
    axios({
          method: 'post',
          url: `https://imasparkle.org:8443/getsp/`,
          data: '',
          headers:{
            "Authorization" : "Token "+ localStorage.Token
          }
        })
        .then(function(response){
            self.search_password = JSON.parse(response.data)
        })
        .catch(function(err){
            console.log("Login Required to save")
        });
},
methods: {
    setTimerForVideoPayer(time) {
            let date = new Date()
            let savedTimer = time ? time : localStorage.getItem('SetTimer') ? localStorage.getItem('SetTimer') : null
            if (savedTimer && savedTimer !== 'null') {
                this.timer = this.timer ? this.timer : setInterval(() => this.countdown(), 1000);
                this.timerTime = Math.ceil((new Date(savedTimer).getTime() - date.getTime()) / 1000)
                this.showTimer = true
            } else {
                this.$root.$emit('setTimeEnd', false)
            }
        },
        clearTimer() {
            this.$root.$emit('setTimeEnd', false)
            this.showTimer = false,
            clearInterval(this.timer);
            this.timer = null
        },
        padTime (time) {
            return (time < 10 ? '0' : '') + time;
        },
        countdown () {
            if(this.timerTime >= 1){
                this.timerTime--;
            } else{
                this.$root.$emit('setTimeEnd', true)
                this.$root.$emit('pauseVideo')
                clearInterval(this.timer);
            }
        },
    setTime() {
        this.enterPass = !this.enterPass
        this.allowSetTime = false
    },
    selectTimer (input) {
        let currentDate = new Date()
        let addedTime = null
        if (input > 0) {
            addedTime = new Date(currentDate.getTime() + (input * 60 * 1000));
            this.setTimerForVideoPayer(addedTime)
        } else if (input === 0){
            this.clearTimer()
        }
        localStorage.setItem('SetTimer', addedTime)
        this.allowSetTime = false
    },
    checkPass () {
				if(this.search_password_entered == this.search_password){
					this.search_password_entered = ''
                    this.allowSetTime = true
                    this.enterPass = false
				}
			}
},
    computed: {
        houre: function() {
            const houre = Math.floor(this.timerTime / (60 * 60));
            return this.timerTime >= 0 ? this.padTime(houre) : '00';
        },
        minutes: function() {
            const minutes = Math.floor(this.timerTime / 60) - (this.houre * 60);
            return this.timerTime >= 0 ? this.padTime(minutes) : '00';
        },
        seconds: function() {
            const seconds = this.timerTime - (this.minutes * 60) - (this.houre * 60 * 60);
            return this.timerTime >= 0 ? this.padTime(seconds) : '00';
        }
    }
}
</script>

<style scoped>
.playlist-add-button{
    padding: 4px 6px;
    margin-left: 5px;
    font-size: 0.8em;
    border-radius: 12px;
    border: 1px solid gray;
    cursor: pointer;
    background: white;
}
.timer-Area {
    position: absolute;
    background-color: #303030;
    padding: 10px 20px;
    border-radius: 7px;
    overflow-y: auto;
    z-index: 999;
}
.timerView{
    color: white;
    float: right;
    margin: 8px 0;
}
.timer-Area input {
    height: 25px;
    border-radius: 8px;
    background-color: #535353;
    border: none;
    padding: 2px 10px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    color: white;
}
.timerOptionsList li {
    list-style: none;
    padding: 5px;
    cursor: pointer;
    font-size: 15px;
}
.timerOptionsList {
   color: white;
    position: absolute;
    left: 0;
    height: 150px;
    background-color: black;
    z-index: 999;
    overflow-y: auto;
    width: 140px;
}
.playlist-add-button.timerClock {
    float: right;
}
</style>