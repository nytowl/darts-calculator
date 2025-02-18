<template>
  <div id="app">
    <div v-if="game == 0" class="menu">
      <h2>Select game mode</h2>
      <div class="gamemode shootout" @click="game = 1">Shoot out</div>
      <div class="gamemode cricket" @click="game = 3">Cricket</div>
      <div class="gamemode tactics" @click="game = 4">Tactics</div>
      <div class="gamemode zero1" @click="game = 2; zero1score = 301">301</div>
      <div class="gamemode zero12" @click="game = 2; zero1score = 501">501</div>
      <div class="gamemode zero13" @click="game = 2; zero1score = 701">701</div>

      <div class="highscore" v-if="shootoutScores.length > 0">
        <h3>Shootout highscores</h3>
        <hr />
        <div v-for="(score, index) in shootoutScores" :key="index" class="line">
          <div class="name">{{score.name}}</div>
          <div class="score">{{score.score}}</div>
        </div>
      </div>
    </div>
    <div v-else-if="players.length == 0">
      <h2>Select player count</h2>
      <div class="playercount">
        <div @click="setPlayers(1)">1</div>
        <div @click="setPlayers(2)">2</div>
        <div @click="setPlayers(3)">3</div>
        <div @click="setPlayers(4)">4</div>
        <div @click="setPlayers(5)">5</div>
      </div>
    </div>
    <div v-else-if="(game == 2 || game == 1) && bullrule == null">
      <h2>Select bulls eye rules</h2>
      <div class="playercount">
        <div @click="setBullruleCombined()">Single and double bull are combined (50)</div>
        <div @click="setBullruleSeparate()">Single and double bull are separate (25 and 50)</div>
      </div>
    </div>
    <div v-else-if="!gamestarted">
      <h2>Edit player names</h2>
      <div class="playernames">
        <input v-for="index in players.length" v-model="players[index - 1]" :key="index">
      </div>
      <div class="startgame" @click="gamestarted = true">Start game</div>
    </div>
    <div v-else>
      <shootout v-if="game == 1" v-bind:playernames="players" v-on:saveScore="handleAddScore" v-on:gameover="reset" :bullrule="bullrule"/>
      <zero1 v-if="game == 2"  v-bind:playernames="players" :initialScore="zero1score" v-on:gameover="reset" :bullrule="bullrule" />
      <cricket v-if="game == 3"  v-bind:playernames="players" v-on:gameover="reset" :targets="['20', '19', '18', '17', '16', '15', 'bull']" />
      <tactics v-if="game == 4"  v-bind:playernames="players" v-on:gameover="reset" :targets="['20', '19', '18', '17', '16', '15', 'bull', 'double', 'triple']" />
    </div>
  </div>
</template>

<script>
import shootout from './shootout.vue'
import zero1 from './zero1.vue'
import cricket from './cricket.vue';
import tactics from './tactics.vue';
import './assets/global.css'

export default {
  name: 'app',
  components: {
    shootout,
    zero1,
    cricket,
    tactics,
  },
  data: function() {
    return {
      game: 0,
      players: [],
      gamestarted: false,
      highscores: [],
      zero1score: 301,
      bullrule: null,
    }
  },
  created() {
    if (localStorage) {
      this.highscores = JSON.parse(localStorage.getItem('highscore') || '[]')
    }
    const search = new URLSearchParams(document.location.search)
    const quickgame = search.get("quickgame");
    if (quickgame) {
      this.players = ["player 1", "player 2"]
      if (quickgame.endsWith("01")) {
        this.game = 2
        this.zero1score = Number(quickgame)
        this.gamestarted = true
      } else if (quickgame === "cricket") {
        this.game = 3
        this.gamestarted = true
      } else if (quickgame === "tactics") {
        this.game = 4
        this.gamestarted = true
      } else if (quickgame === "shootout") {
        this.game = 1
        this.gamestarted = true
      }
    }
  },
  computed: {
    shootoutScores: function() {
      return this.highscores.filter(s => s.mode === 'shootout').sort((a, b) => a.score < b.score).slice(0, 5)
    },
  },
  methods: {
    handleAddScore(score) {
      this.highscores.push(score)
      if (localStorage) {
        const scores = JSON.parse(localStorage.getItem('highscore') || '[]')
        localStorage.setItem('highscore', JSON.stringify([...scores, score]))
      }
    },
    reset() {
      this.game = 0
      this.players = []
      this.gamestarted = false
      this.bullrule = null
    },
    setPlayers(count) {
      const players = []
      for (let i = 0; i < count; i++) {
        players.push("player " + (i+1))
      }
      this.players = players
    },
    setBullruleCombined() {
      this.bullrule = "combined"
    },
    setBullruleSeparate() {
      this.bullrule = "separate"
    }
  }
}
</script>

<style scoped>
.gamemode, .playercount > div, .startgame {
  color: white;
  font-weight: bold;
  padding: 20px 30px;
  cursor: pointer;
}
.shootout {
  background-color: #EAC435
}
.tactics {
  background-color: #345995
}
.cricket {
  background-color: #340095
}
.zero1 {
  background-color: #E40066
}
.zero12 {
  background-color: #03CEA4
}
.zero13 {
  background-color: #FB4D3D
}
.playercount > div:nth-child(1) {
  background-color: #FB4D3D
}
.playercount > div:nth-child(2) {
  background-color: #03CEA4
}
.playercount > div:nth-child(3) {
  background-color: #E40066
}
.playercount > div:nth-child(4) {
  background-color: #345995
}
.playercount > div:nth-child(5) {
  background-color: #EAC435
}
.highscore {
  padding: 30px;
}
.highscore > h3 {
  margin-bottom: 30px;
}
.line {
  display: flex;
  font-weight: bold;
  padding: 10px;
}
.line:nth-child(even) {
  background-color: #f0f0f0;
}
.name {
  color: #333;
}
.score {
  color: rgb(45, 121, 45);
  margin-left: auto;
}

.playernames {
  display: flex;
  flex-direction: column;
  padding: 30px;
  gap: 20px;
}
.playernames > input {
  max-width: 300px;
}
.startgame {
  background-color: #345995
}


</style>
