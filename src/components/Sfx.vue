<template>
  <div>
    <head>
      <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons"/>
    </head>
    <div v-for="group in effect_groups" :key="group.no">
      <button v-for="e in group" :key="e.hotkey" @click="play_effect(e)" :id="e.hotkey" class="effect">
        {{ e.name }}
        <br/>
        <img :src="get_img(e.icon)" class="effect_icon">
        <br/>
        {{ e.hotkey }}
      </button>
    </div>

    <br/>
    <br/>
    <br/>
    <div class="slidecontainer" style="height: 100;">
      <i class="material-icons">volume_up</i>
      <input type="range" min="1" max="100" v-model="volume" class="slider" id="myRange" style="height: 23px; width: 150px;">
    </div>

    <br/>
    <br/>
    <button @click="stop_all" ref="stop_all_button" class="stop_all_button">
      Stop all
      <br/>
      <img src="../assets/stop.png" class="effect_icon">
      <br/>
      [Esc]
    </button> 
  </div>
</template>

<script>

import effectGroups from "@/assets/config.json"

const EFFECT_STATUS = {
  READY : 0,
  PLAYING : 1,
  STOPPING : 2,
  STOPPED : 3
}

const SFX_PROP = {
  AUDIO : 'audio'
}

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      effect_groups: effectGroups,
      playing: [],
      volume: 40
    }

  },
  mounted() {
    this.load_sfx()

    window.addEventListener("keydown", function(e) {
      this.on_key_press(e)
    }.bind(this));
  },
  watch: {
    volume: function() {
      this.playing.forEach(p => {
        if (p.status != EFFECT_STATUS.STOPPING) {
          p.volume = this.get_volume()
        }
      })
    }
  },
  methods: {
    load_sfx() {
      let all_effects = this.effect_groups.flat()
      for (let i=0 ; i<all_effects.length ; i++) {
        let effect = all_effects[i]
        effect[SFX_PROP.AUDIO] = new Audio(effect.file)
      }
    },
    get_img(filename) {
      if (filename) {
        return filename
      } else {
        return require('@/assets/sound.png')
      }
      
    },
    get_volume() {
      return this.volume/100
    },
    play_effect(effect) {
      var audio = effect[SFX_PROP.AUDIO].cloneNode()
      audio.volume = this.get_volume()
      this.playing.push(audio)
      audio.play()
    },
    async stop_all() {
      let all_playing = this.playing

      all_playing.forEach(p => {
        p.status = EFFECT_STATUS.STOPPING
      })

      let has_stoppable = true
      while (has_stoppable) {
        has_stoppable = false

        // Find marked as STOPPING and decrease volume
        all_playing.forEach(p => {
          if (p.status === EFFECT_STATUS.STOPPING) {
            if (p.volume > 0.1) {
              p.volume -= Math.min(0.05, p.volume)
            } else {
              p.volume -= Math.min(0.01, p.volume)
            }
            has_stoppable = true
          }
        })
        
        // remove all effects with 0 volume
        all_playing.forEach(function(effect) {
          if (effect.volume === 0) {
            var index = all_playing.indexOf(effect)
            all_playing.splice(index, 1)
          }
        })
        await this.sleep(50)
      }
    },
    on_key_press(e) {
      if(e.keyCode == 27) { // Detect Escape key pressed
          this.stop_all()
          return
      }

      let all_effects = this.effect_groups.flat()
      // Check if a shortcut matches
      for (let i=0 ; i<all_effects.length ; i++) {
        let effect = all_effects[i]
        if (effect.hotkey && String.fromCharCode(e.keyCode) && 
            effect.hotkey.toLowerCase() === String.fromCharCode(e.keyCode).toLowerCase()) {
          document.getElementById(effect.hotkey).click();
          return
        }
      }

    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }
  .effect {
    height: 100px;
    width: 100px;
  }
  .stop_all_button {
    height: 100px;
    width: 200px;
  }
  .effect_icon {
    height: 50px;
    width: 50px;
  }
  img {
    pointer-events: none;
  }
</style>
