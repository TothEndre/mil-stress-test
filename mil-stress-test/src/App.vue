<template>
  <div id="app">
    <div v-if="isRunning">
      <section>
        <button @click="nextRound">NEXT ROUND</button>
        <button @click="restart">RESTART</button>
        <button @click="stop">STOP</button>
        <hr>
      </section>
      <section v-if="queue.length != 0">
        <div id="counter">
          {{ fullTimer }}
        </div>
        <div>
          <h1>Feladat</h1>
          <div v-if="queue[queue.length - 2].type === 'shoot'">
            <p>{{ queue[queue.length - 2].name }} : {{ queue[queue.length - 2].detail }} . cél</p>
            <p>teljes idő: {{ queue[queue.length - 2].duration }} mp</p>
          </div>
          <div v-else>
            <p>{{ queue[queue.length - 2].name }} : {{ queue[queue.length - 2].detail }} ismétlés</p>
            <p>teljes idő: {{ queue[queue.length - 2].duration }} mp</p>
          </div>
        </div>
        <hr>
      </section>
      <section v-if="queue.length >= 2">
        <div>
          <h1>Következő</h1>
          <p>{{ queue[queue.length - 1].name }} ( {{ queue[queue.length - 1].duration }} mp )</p>
        </div>
      </section>
    </div>
    <div v-else>
      <p>Add meg a gyakorlatok összidejét</p>
      <select v-model="usrInMin">
        <option disabled value="">perc</option>
        <option v-for="n in 11" :key="n">{{ n - 1 }}</option>
      </select>
      <select v-model="usrInSec">
        <option disabled value="">mp</option>
        <option v-for="n in 6" :key="n">{{ (n - 1) * 10 }}</option>
      </select>
      <p>Add meg a célok számát</p>
      <select v-model="numberOfTgt">
        <option v-for="n in 5" :key="n">{{ n }}</option>
      </select>
    </div>
    <hr>
    <button v-if="!isRunning" @click="start">START TRAINING</button>
    <button v-if="!isRunning && queue.length !== 0" @click="resetQueue">RESET QUEUE</button>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      exercises: {
        'fekvő': {
          type: 'movement',
          difficulty: 1,
          timePerRep: {
            min: 0.5,
            max: 2
          },
          repCount: {
            min: 5,
            max: 15
          }
        },
        'felülés': {
          type: 'movement',
          difficulty: 1,
          timePerRep: {
            min: 1,
            max: 2.5
          },
          repCount: {
            min: 5,
            max: 10
          }
        },
        'célozz': {
          type: 'shoot',
          positions: ['állva', 'guggolva', 'fekve'],
          difficulty: 1,
          timePerRep: {
            min: 3,
            max: 6
          }
        }
      },
      queue: [],
      isRunning: false,
      numberOfTgt: 0,
      usrInMin: 0,
      usrInSec: 0,
      totalTimeTask: 0,
      timerID: 0,
      timeLeft: 0,
      calcDur: 0
    }
  },
  computed: {
    totalTimeUsr: function() {
      return parseInt(this.usrInMin * 60) + parseInt(this.usrInSec)
    },
    fullTimer: function() {
      const minute = Math.trunc(this.timeLeft / 60)
      const seconds = this.timeLeft % 60 < 10 ? `0${this.timeLeft % 60}` : this.timeLeft % 60
      return `${minute} : ${seconds}`
    }
  },
  methods: {
    start() {
      console.log('start()')
      if (this.totalTimeUsr === 0) {
        alert('Add meg az időt!')
        console.log('User input (total time) is missing')
        return
      }

      // Add tasks to the queue, for at least as long as user wanted
      this.isRunning = true
      while (this.totalTimeTask < this.totalTimeUsr) {
        this.addTask()
      }

      // Modify last exercise duration to match user input
      console.log('utolsó: ' + this.queue[this.queue.length -1].duration)
      console.log('totalTimetask: ' + this.totalTimeTask)
      console.log('totalTimeUsr: ' + this.totalTimeUsr)
      const modifier = (this.totalTimeTask - this.totalTimeUsr)
      console.log('modifier:' + modifier)
      this.queue[this.queue.length -1].duration -= modifier
      console.log(`modified duration: ${this.queue[this.queue.length -1].duration}`)

      // Set timeLeft and deductor
      this.timeLeft = this.totalTimeUsr
      const timerID = setInterval(() => {
        this.timeLeft -= 1
      }, 1000)
      this.timerID = timerID
      
      this.queue.forEach(element => {
        console.log(element.duration)
        this.calcDur += element.duration
      });

    },
    stop() {
      console.log('stop()')
      this.isRunning = false
      clearInterval(this.timerID)
    },
    restart() {
      console.log('restart()')
      this.start()
    },
    nextRound() {
      console.log('nextRound()')
      // Skip to next exercise
      
    },
    createTask() {
      console.log('createTask()')
      // Create a task that can be added to the queue, returns the task object
      var newTask = {};
      const getRndChoice = (canShoot) => {
        const keys = Object.keys(this.exercises)
        if (!canShoot) {
          keys.splice(keys.indexOf('célozz', 1))
        }
        return keys[Math.floor(Math.random() * keys.length)]
      }
      
      // Check queue tasks' type and generate a choice
      if (this.queue.length >= 2) {
        if (this.queue[this.queue.length - 2].type === 'movement' && this.queue[this.queue.length - 1].type === 'movement') {
          newTask.name = 'célozz'
        } else if (this.queue[this.queue.length - 2].type === 'shoot' && this.queue[this.queue.length - 1].type === 'shoot') {
          newTask.name = getRndChoice(false)
        } else {
          newTask.name = getRndChoice(true)
        }
      } else {
        newTask.name = getRndChoice(true)
      }
      // Create details for the task
      console.log("newTask.name is: " + newTask.name)
      newTask.type = this.exercises[newTask.name].type

      if (newTask.type === 'shoot') {
        newTask.detail = Math.ceil(Math.random() * this.numberOfTgt)
        const min = this.exercises.célozz.timePerRep.min
        const max = this.exercises.célozz.timePerRep.max
        newTask.duration = Math.ceil(Math.random() * (max - min) + min)
        newTask.position = this.exercises.célozz.positions[Math.floor(Math.random() * this.exercises.célozz.positions.length)]
      } else {
        const repMin = this.exercises[newTask.name].repCount.min
        const repMax = this.exercises[newTask.name].repCount.max
        newTask.detail = Math.ceil(Math.random() * (repMax - repMin)) + repMin
        const durMin = Math.ceil(newTask.detail * this.exercises[newTask.name].timePerRep.min)
        const durMax = Math.ceil(newTask.detail * this.exercises[newTask.name].timePerRep.max)
        newTask.duration = Math.ceil(Math.random() * (durMax - durMin)) + durMin
      }

      console.log(`Random choice was: ${JSON.stringify(newTask, undefined, 2)}`)

      return newTask
    },
    addTask() {
      console.log('addTask()')
      // Invoke createTask() then add to queue
      const task = this.createTask()
      this.queue.push(task)
      this.totalTimeTask += task.duration
    },
    resetQueue() {
      this.queue = []
      this.totalTimeTask = 0
      this.timeLeft = 0
      console.log('queue is reset')
    }
  },
  updated() {
    console.log('updated()')
    console.log(JSON.stringify(this.queue))
  }
}
</script>

<style>
body {
  font-family: 'Play', sans-serif;
}

button:hover {
  cursor: pointer;
}
</style>