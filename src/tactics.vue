<template>
  <div id="shootout">
    <h2 v-if="this.roundLimit == -1" class="rounds">{{ this.currentround }}</h2>
    <h2 v-else-if="this.roundLimit != -1" class="rounds">{{ this.currentround }}/{{ this.roundLimit }}</h2>
    <div class="head">
      <table class="tacticstable">
        <thead>
          <tr>
            <td></td>
            <td v-for="(player, index) in players" v-bind:key="player.name" :class="{ active: turn === index, player }">
              {{ player.name }}
            </td>
          </tr>
        </thead>
        <tr v-for="t in targets" v-bind:key="t">
          <td class="targetD">{{ t }}</td>
          <td v-for="(player, index) in players" v-bind:key="player.name" :class="{ playerd: index + 1 < players.length }">
            <span v-if="player.score[t] === 0"></span>
            <span v-else-if="player.score[t] === 1">X</span>
            <span v-else-if="player.score[t] === 2">XX</span>
            <span v-else-if="player.score[t] === 3"><s>XXX</s></span>
          </td>
        </tr>
        <tr class="scorerow">
          <td class="targetD"></td>
          <td v-for="(player, index) in players" v-bind:key="player.name" :class="{ playerd: index + 1 < players.length }">
            {{ player.score.points }}
          </td>
        </tr>
      </table>
      <div class="record">
        <div>{{ record[0] | totext }}</div>
        <div>{{ record[1] | totext }}</div>
        <div>{{ record[2] | totext }}</div>
      </div>
    </div>
    <div v-if="playerchange" class="playerchange" @click="changeplayer">Player change</div>
    <div v-if="gameover" class="gameover">
      <div class="mainmenu" @click="goToMenu">Main menu</div>
      <div class="playagain" @click="init">Play again</div>
    </div>
    <Dartboard v-on:hit="handleDartHit" :disabled="playerchange || gameover"  game=4 />
  </div>
</template>

<script>
import Dartboard from './components/Dartboard.vue'

export default {
  name: 'tactics',
  components: {
    Dartboard
  },
  props: ['playernames', 'targets'],
  data: function() {
    return {
      turn: 0,
      roundLimit: -1,
      currentround: 1,
      players: [],
      record: [],
      playerchange: false,
      gamover: false,
    }
  },
  created() {
    this.init()
  },
  filters: {
    totext(score) {
      if (score === undefined) {
        return ''
      }
      if (score === null) {
        return 'miss'
      }
      if ('sio'.includes(score[0])) {
        return 'single ' + score.substr(1)
      }
      if (score.substring(0,6) === 'double') {
        return score
      }
      if (score.substring(0,6) === 'triple') {
        return score
      }
      if (score[0] === 'd') {
        return 'double ' + score
      }
      if (score[0] === 't') {
        return 'triple ' + score
      }
      return score
    }
  },
  methods: {
    init() {

      const targets = this.targets.reduce((all, next) => ({...all, [next]: 0 }), {})
      console.log(this.playernames)
      this.gameover = false
      this.playerchange = false
      this.record = []
      this.currentround = 1
      this.turn = 0
      this.players = this.playernames.map(name => ({
        name,
        finished: false,
        score: {
          points: 0,
          ...targets
        },
        throws: [],
      }))
      
      setTimeout(() => {
        this.highlight()
      }, 10)
    },
    makeTargets(target) {
      console.log(target);
      const range = [...Array(20).keys()].map(i => i+1)
      if (target === 'bull') {
        return '.bull';
      } else if (target === 'triple') {
        const triples = range.map(s => "#t"+s);
        return triples;
      } else if (target === 'double') {
        const doubles = range.map(s => "#d"+s).join(',');
        console.log(doubles);
        return doubles;
      } else {
        return '.a' + target;
      }
    },
    highlight() {
      console.log(this.targets);
      const player = this.players[this.turn]
      const otherPlayers = this.players.filter(p => p !== player);

      const openTargets = this.targets.filter(target => player.score[target] !== 3)
      console.log(openTargets);

      const scoreTargets = this.targets.filter(target => player.score[target] === 3)
      console.log(scoreTargets);

      const pointTargets = scoreTargets.filter(target => otherPlayers.some(p => p.score[target] !== 3))
      console.log(pointTargets);
      const closedTargets = scoreTargets.filter(t => !pointTargets.includes(t))
      console.log(closedTargets);

      const makeClassName = target => this.makeTargets(target);
      const getSelection = list => {
        const query = list.map(makeClassName).reduce((selector, item) => {
          if (selector !== "") selector += ","
          selector += item
          return selector
        }, "")

        return query == "" ? [] : document.querySelectorAll(query)
      }

      document.querySelectorAll('.bull,.a20,.a19,.a18,.a17,.a16,.a15,.a14,.a13,.a12,.a11,.a10,.a9,.a8,.a7,.a6,.a5,.a4,.a3,.a2,.a1')
        .forEach(item => item.style.fill = "inherit")

      getSelection(openTargets).forEach(t => t.style.fill = "blue")
      getSelection(pointTargets).forEach(t => t.style.fill = "green")
      getSelection(closedTargets).forEach(t => t.style.fill = "#999")

    },
    handleDartHit(hit) {
      console.log("handleDartHit " + hit)
      const { score, target, multiplier } = this.calculateValue(hit)
      const player = this.players[this.turn]

      player.throws.push(target)

      const otherPlayers = this.players.filter(p => p !== player && p.finished === false);
      
      if (this.targets.includes(target)) {

        let leftover = 0
        if (player.score[target] === 3) {
          leftover = multiplier
        } else if (player.score[target] + multiplier >= 3) {
          leftover = multiplier - (3 - player.score[target])
          player.score[target] = 3
        } else {
          player.score[target] += multiplier
        }
        
        if (leftover > 0) {
          let scoring = 0
          this.players.forEach(p => {
            if (p.score[target] == 3) {
                scoring = scoring + 1
            }
          })
          console.log("scoring " + scoring + " length " + this.players.length)
          if (scoring > 0 && scoring < this.players.length) {
            player.score.points += leftover * score
            otherPlayers.forEach(other => {
              if (other.score[target] === 3) {
                other.score.points += leftover * score
              }
            })
          }
        }
      }

      this.record.push(hit)

      const mostpoints = this.players.reduce((most, next) => Math.max(next.score.points, most), Number.MIN_SAFE_INTEGER)
      const finished = this.targets.every(target => player.score[target] === 3) && player.score.points >= mostpoints

      console.log("most points " + mostpoints)
      console.log("finished " + finished )

      if (finished) {
        player.finished = true
        this.gameover = true
        this.playerchange = false
        return
      }

      if (player.throws.length % 3 == 0) {
        this.playerchange = true
      }

      this.highlight()
    },
    calculateValue(target) {
      console.log("calculateValue " + target)

      if (target === null) {
        return { score: 0, target: null, multiplier: null }
      }
      const multiplierString = target[0]
      const special = target.substr(0,6)
      const number = target.substr(1)
      if (special == "double") {
        const number = target.substr(6)
        return { score: parseInt(number) * 2, target: "double", multiplier: 1 }
      } else if (special == "triple") {
        const number = target.substr(6)
        return { score: parseInt(number) * 3, target: "triple", multiplier: 1 }
      } else if (number == "bull") {
        return { score: 25, target: 'bull', multiplier: multiplierString === 'd' ? 2 : 1 }
      } else if (multiplierString == "o" || multiplierString == "i") {
        return { score: parseInt(number), target: number, multiplier: 1 }
      } else if (multiplierString == "d") {
        return { score: parseInt(number), target: number, multiplier: 2 }
      } else if (multiplierString == "t") {
        return { score: parseInt(number), target: number, multiplier: 3 }
      }
    },
    changeplayer() {
      if (this.turn + 1 === this.players.length) {
        this.currentround = this.currentround + 1
      }
      this.playerchange = false
      this.turn = (this.turn + 1) % this.players.length
      this.record = []

      this.highlight()
    },
    goToMenu() {
      this.$emit('gameover')
    }
  }
}
</script>

<style scoped>
#shootout { display: relative }
.head {
  display: grid;
  grid-template-columns: 2fr 1fr;
  place-items: center;
  border-bottom: 4px solid black;
}

.tacticstable {
  width: 100%;
  border-collapse: collapse;
}
.scorerow {
  border-top: 1px solid #aaa;
}
.playerd, .targetD {
  border-right: 1px solid #aaa;
}
.targetD {
  max-width: 30px;
  padding: 2px;
}
.rounds {
  position: absolute;
  top: 0px;
  right: 10px;
  font-size: 70%;
  display: inline;
  text-align: right;
}

.players {
  display: flex;
  color: white;
}
.record {
  justify-self: end;
  background-color: black;
}

.record > div {
  color: white;
  font-weight: bold;
  font-size: 1rem;
  border: 2px solid white;
  padding: 10px;
  min-width: 100px;
  min-height: 45px;
}
.players > div {
  padding: 10px;
  border-bottom: 4px solid black;
}
h3 {
  margin: 0 0 5px 0;
}
.score {
  margin-right: 10px;
}
.player.active {
  font-weight: bold;
}

.playerchange, .gameover {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100%;
  font-weight: bold;
  transform: translate(-50%, -50%);
  background-color: rgb(189, 201, 87);
  color:white;
  font-size: 2rem;
  padding: 30px;
  cursor: pointer;
}
.gameover { padding: 0}
.gameover > div {padding: 30px}
.playagain {
  background-color: rgb(45, 102, 19);
}

</style>
