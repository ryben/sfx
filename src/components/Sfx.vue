<template>
  <div>
    <div v-for="group in effect_groups" :key="group.no">
      <button v-for="e in group" :key="e.id" @click="play_effect(e)" class="effect">
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
            id: 1,
            hotkey: 'q',
            name: 'Applause',
            file: require('@/assets/applause.mp3'),
            icon: "applause.png"
          },
          {
            id: 2,
            hotkey: 'w',
            name: 'Cheers',
            file: require('@/assets/cheers.mp3'),
            icon: "cheers.png"
          },
          {
            id: 3,
            hotkey: 'e',
            name: 'Bell',
            file: require('@/assets/bell.mp3'),
            icon: "bell.png"
          }
        ],
        [
          {
            id: 4,
            hotkey: 'a',
            name: 'Awww',
            file: require('@/assets/awww.mp3'),
            icon: "awww.png"
          },
          {
            id: 5,
            hotkey: 's',
            name: 'Buzzer',
            file: require('@/assets/buzzer.mp3'),
            icon: "buzzer.png"
          }
        ],
        [
          {
            id: 6,
            hotkey: 'z',
            name: 'Drum roll',
            file: require('@/assets/drum_roll.mp3'),
            icon: "drum_roll.png"
          },
          {
            id: 7,
            hotkey: 'x',
            name: 'Camera',
            file: require('@/assets/camera_shot.mp3'),
            icon: "camera.png"
          }
        ]
      ],
      playing: []
    }

  },
  mounted() {
    window.addEventListener("keypress", function(e) {
      this.on_key_press(String.fromCharCode(e.keyCode))
    }.bind(this));
  },
  methods: {
    get_img(filename) {
      if (filename) {
        return require('../assets/' + filename)
      } else {
        return require('../assets/sound.png')
      }
      
    },
    play_effect(effect) {
      this.$refs.stop_all_button.focus()
      var audio = new Audio(effect.file)
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
          this.play_effect(effect)
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
