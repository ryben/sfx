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
      [space]
    </button> 
  </div>
</template>

<script>

const EFFECT_STATUS = {
  READY : 0,
  PLAYING : 1,
  STOPPING : 2,
  STOPPED : 3
}
      
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      effect_groups: [
        [
          {
            hotkey: '1',
            name: 'Applause',
            file: require('@/assets/applause.mp3'),
            icon: "applause.png"
          },
          {
            hotkey: '2',
            name: 'Cheers',
            file: require('@/assets/cheers.mp3'),
            icon: "cheers.png"
          },
          {
            hotkey: '3',
            name: 'Bell',
            file: require('@/assets/bell.mp3'),
            icon: "bell.png"
          }
        ],
        [
          {
            hotkey: 'q',
            name: 'Awww',
            file: require('@/assets/awww.mp3'),
            icon: "awww.png"
          },
          {
            hotkey: 'w',
            name: 'Buzzer',
            file: require('@/assets/buzzer.mp3'),
            icon: "buzzer.png"
          }
        ],
        [
          {
            hotkey: 'a',
            name: 'Drum roll',
            file: require('@/assets/drum_roll.mp3'),
            icon: "drum_roll.png"
          },
          {
            hotkey: 's',
            name: 'Camera',
            file: require('@/assets/camera_shot.mp3'),
            icon: "camera.png"
          }
        ],
        [
          {
            hotkey: 'z',
            name: 'Laugh 1',
            file: require('@/assets/laugh_1.mp3'),
            icon: "laugh_1.png"
          },
          {
            hotkey: 'x',
            name: 'Laugh 2',
            file: require('@/assets/laugh_2.mp3'),
            icon: "laugh_2.png"
          },
          {
            hotkey: 'c',
            name: 'Laugh 3',
            file: require('@/assets/laugh_3.mp3'),
            icon: "laugh_3.png"
          },
          {
            hotkey: 'v',
            name: 'Tiny Laugh 1',
            file: require('@/assets/tiny_laugh_1.mp3'),
            icon: "tiny_laugh_1.png"
          },
          {
            hotkey: 'b',
            name: 'Tiny Laugh 2',
            file: require('@/assets/tiny_laugh_2.mp3'),
            icon: "tiny_laugh_2.png"
          }
        ]
      ],
      playing: [],
      volume: 40
    }

  },
  mounted() {
    window.addEventListener("keypress", function(e) {
      this.on_key_press(String.fromCharCode(e.keyCode))
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
    get_img(filename) {
      if (filename) {
        return require('../assets/' + filename)
      } else {
        return require('../assets/sound.png')
      }
      
    },
    get_volume() {
      return this.volume/100
    },
    play_effect(effect) {
      this.$refs.stop_all_button.focus()
      var audio = new Audio(effect.file)
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
    on_key_press(char) {
      let all_effects = this.effect_groups.flat()
      // Check if a shortcut matches
      for (let i=0 ; i<all_effects.length ; i++) {
        let effect = all_effects[i]
        if (effect.hotkey === char) {
          document.getElementById(effect.hotkey).click();
          return
        }
      }

      switch (char) {
        case ' ':
          this.stop_all()
          break
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
</style>
