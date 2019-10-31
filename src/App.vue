<template>
  <div id="app">
     <div id="stopwatch">
       {{displayTime}}
      </div>
      <div id="stopwatch-controls">
        <button id="reset-and-lap" @click="()=> resetAndLapHander()"> 
            {{ ( isStarted&& !isStopped) ? 'Lap' :'Reset'}}
         </button>
        <button id="start-and-stop" @click="()=> startAndStopHandler()"
          :class="{ red: isStarted && !isStopped }" > 

           {{ ( isStarted&& !isStopped) ? 'Stop' :'Start'}}
        </button>
      </div>
      <ul id="stopwatch-records">
          <li v-for="(lap,index) in lapsRecords" :key="index"
              :class="{ red: lap.isSlowest, green: lap.isFastest } "  >
              <span> Lap {{lapsRecords.length - index}} </span>
              <span> {{lap.display}} </span>
           </li> 
<!--            <li class="green">
              <span> Lap 1 </span>
              <span> 00:01.32 </span>
           </li> 
           <li class="red">
              <span> Lap 1 </span>
              <span> 00:01.33 </span>
           </li>  -->
      </ul>

  </div>
</template>

<script>


export default {
  name: 'app',
  components: {
    
  },
  computed: {
    lapsRecords() {
      let lapsClone = [...this.laps]

       if (lapsClone.length >2 ) {
        lapsClone = lapsClone.map( (oneLap) => {
            return {
              ...oneLap,
              isFastest: false,
              isSlowest: false,
            }  // clean
          });
        

        let fastestIndex = 0
        let slowestIndex = 0

        for ( let i=0; i < lapsClone.length - 1; i++) {
          if (lapsClone[i].time <lapsClone[fastestIndex].time) {
            fastestIndex = i
          }
          if (lapsClone[i].time >lapsClone[fastestIndex].time) {
            slowestIndex = i
          }
        }

        lapsClone[fastestIndex].isFastest = true;
        lapsClone[slowestIndex].isSlowest = true;
       }


      return lapsClone.reverse();
    }
  },
  watch: {
    displayTime() {
      let lapsClone = [...this.laps]
      let elapsedTime = this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap
      let lapTime = (elapsedTime/1000).toFixed(2);

      lapsClone[this.laps.length-1] = {
        time : parseFloat(lapTime),
        display: this.formatTime(lapTime),
        isFastest: false,
        isSlowest: false,
      }

      this.laps=lapsClone
    }
  },
  data() {
    return {
      timer:null,
      displayTime:'00:00:00',
      startTime:null,
      currentTime:null,
      isStarted: false,
      isStopped: false,
      stopTime: null,
      stoppedTimeOffset: 0 ,
      stoppedTimeOffsetForLap: 0 ,
      lastLapTime: null,
      laps:[],
    }
  },
  methods:{
    startAndStopHandler() {
      if (!this.isStarted) {
        this.startTime = Date.now();
        this.lastLapTime = Date.now();


        this.timer= setInterval(()=>{
          this.currentTime = Date.now();
          let elapsedTime = this.currentTime - this.startTime;
          this.displayTime = this.formatTime( (elapsedTime/1000).toFixed(2));
        },10);

        this.laps = [{ 
          time :0, 
          display: this.displayTime,
          isFastest: false,
          isSlowest: false,
        }]
        this.isStarted = true;
      }
      else if (this.isStarted && !this.isStopped) {
        clearInterval(this.timer);
        this.stopTime = Date.now();
        this.isStopped = true;
      }
      else if (this.isStarted && this.isStopped) {
        this.stoppedTimeOffset += Date.now() - this.stopTime

        this.timer= setInterval(()=>{
          this.currentTime = Date.now();
          let elapsedTime = this.currentTime - this.startTime - this.stoppedTimeOffset;
          this.displayTime = this.formatTime( (elapsedTime/1000).toFixed(2));
        },10);

        this.isStopped = false ;
      }
    },
    resetAndLapHander() {
      if (this.isStarted && this.isStopped) {

        this.timer = null;
        this.displayTime= '00:00:00';
        this.startTime = null;
        this.currentTime = null;
        this.isStarted = false;
        this.isStopped = false;
        this.stopTime = null;
        this.stoppedTimeOffset = 0;
        this.laps =[];
      } else if (this.isStarted && !this.isStopped ) {
        //Lap
        let elapsedTime = this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap
        let lapTime = (elapsedTime/1000).toFixed(2)


        this.laps.push({
          time: parseFloat(lapTime),
          display: this.formatTime(lapTime),
        });

        this.lastLapTime = this.currentTime
        this.stoppedTimeOffsetForLap = 0;
      }


    },
    formatTime(seconds) {
      let date = new Date(null)
      date.setSeconds(seconds)
      let result = date.toISOString().substr(14,5);

      
     
      return `${result}.${(seconds+'').split('.')[1]}`;
    }
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #000;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  color: #fff;
}

#stopwatch {
  flex : 0px 1 1;
  font-size: 5rem;
  display: flex;
  justify-content: center;
  align-items: center;

}

#stopwatch-records {
  flex: 0px 1 1;
  list-style: none;
  padding: 0;
  margin: 0 1.2rem;
  overflow: auto;
}

#stopwatch-records li {
  border-bottom: 1px solid #323234;
  display: flex;
  padding: .8rem 0;
}

#stopwatch-records li.green {
  color: #2ED158;
}

#stopwatch-records li.red {
  color: #FF453A;
}

#stopwatch-records li:first-child {
  border-top: 1px solid #323234;
  display: flex;
  padding: .8rem 0;
}

#stopwatch-records span {
  display: block;
  flex: 0px 1 1;
}

#stopwatch-records span:last-child {
  text-align: right;
}

#stopwatch-controls {
  overflow: hidden;
  margin: 0 1.25rem;
  flex: 100px 0 0;
}


button#reset-and-lap , button#start-and-stop {
  width: 20%;
  height: 80%;
  outline: none;
  background-color: #333;
  color: #fff;
  font-size: 1rem;
  border: 2px solid #000;
  box-shadow: 0px 0px 0px 2px #333;
  border-radius:100%;

}


button#start-and-stop {
  background-color: #082A12;
  box-shadow: 0px 0px 0px 2px #082A12;
  color: #2ED158;
  float: right;
}


button#start-and-stop.red {
  background-color: #320E0B;
  color: #FF453A;
  box-shadow: 0px 0px 0px 2px #320E0B;

}

</style>
