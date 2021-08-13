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
            file: require('@/assets/positive/applause.mp3'),
            icon: "applause.png"
          },
          {
            hotkey: '2',
            name: 'Cheers',
            file: require('@/assets/positive/cheers.mp3'),
            icon: "cheers.png"
          },
          {
            hotkey: '3',
            name: 'Bell',
            file: require('@/assets/positive/bell.mp3'),
            icon: "bell.png"
          },
          {
            hotkey: '4',
            name: 'Laugh',
            file: require('@/assets/positive/laugh.mp3'),
            icon: "laugh.png"
          },
          {
            hotkey: '5',
            name: 'Tiny Laugh',
            file: require('@/assets/positive/tiny_laugh.mp3'),
            icon: "tiny_laugh.png"
          },
          {
            hotkey: '6',
            name: 'Wow!',
            file: require('@/assets/positive/wow.mp3')
          },
          {
            hotkey: '7',
            name: 'Yey!',
            file: require('@/assets/positive/yey.mp3')
          },
          {
            hotkey: '8',
            name: 'Awesome!',
            file: require('@/assets/positive/awesome.mp3')
          },
          {
            hotkey: '9',
            name: 'That\'s Right!',
            file: require('@/assets/positive/thats_right.mp3')
          },
          {
            hotkey: '0',
            name: 'Gong',
            file: require('@/assets/positive/Gong.mp3'),
            icon: 'gong.png'
          },
          {
            hotkey: '-',
            name: 'Horn',
            file: require('@/assets/positive/MLG Horns  MLG Sound Effects HD.mp3'),
            icon: "horn.png"
          },
          {
            hotkey: '=',
            name: 'Toinks',
            file: require('@/assets/positive/Toinks.mp3')
          }
        ],
        [
          {
            hotkey: 'Q',
            name: 'Awww',
            file: require('@/assets/negative/awww.mp3'),
            icon: "awww.png"
          },
          {
            hotkey: 'W',
            name: 'Buzzer',
            file: require('@/assets/negative/buzzer.mp3'),
            icon: "buzzer.png"
          },
          {
            hotkey: 'E',
            name: 'Crickets',
            file: require('@/assets/negative/Crickets Awkward Silence.mp3'),
            icon: "cricket.png"
          },
          {
            hotkey: 'R',
            name: 'Fart',
            file: require('@/assets/negative/Fart.mp3'),
            icon: "fart.png"
          },
          {
            hotkey: 'T',
            name: 'Mario death',
            file: require('@/assets/negative/Mario death.mp3'),
            icon: 'mario.png'
          },
          {
            hotkey: 'Y',
            name: 'Failure',
            file: require('@/assets/negative/Funny sound of failure.mp3')
          },
          {
            hotkey: 'U',
            name: 'Trombone',
            file: require('@/assets/negative/Sad Trombone.mp3')
          }
        ],
        [
          {
            hotkey: 'A',
            name: 'Drum roll',
            file: require('@/assets/neutral/drum_roll.mp3'),
            icon: "drum_roll.png"
          },
          {
            hotkey: 'S',
            name: 'Entrance',
            file: require('@/assets/neutral/Drum Roll grand entrance.mp3')
          },
          {
            hotkey: 'D',
            name: 'Camera',
            file: require('@/assets/neutral/camera_shot.mp3'),
            icon: "camera.png"
          },
          {
            hotkey: 'F',
            name: '10 secs',
            file: require('@/assets/neutral/10 sec countdown.mp3'),
            icon: 'timer.png'
          },
          {
            hotkey: 'G',
            name: 'Pressure',
            file: require('@/assets/neutral/Clock pressure.mp3')
          },
          {
            hotkey: 'H',
            name: 'Fly away',
            file: require('@/assets/neutral/Fly away.mp3')
          },
          {
            hotkey: 'J',
            name: 'Alert',
            file: require('@/assets/neutral/metal gear solid.mp3'),
            icon: 'alert.png'
          },
          {
            hotkey: 'K',
            name: 'Nature',
            file: require('@/assets/neutral/Nature.mp3'),
            icon: 'nature.png'
          },
          {
            hotkey: 'L',
            name: 'Punch',
            file: require('@/assets/neutral/Punch.mp3'),
            icon: 'punch.png'
          },
          {
            hotkey: ';',
            name: 'Round one',
            file: require('@/assets/neutral/Round one.mp3'),
            icon: 'one.png'
          },
          {
            hotkey: '\'',
            name: 'Round two',
            file: require('@/assets/neutral/Round two.mp3'),
            icon: 'two.png'
          },
          {
            hotkey: ':',
            name: 'Film Rewind',
            file: require('@/assets/neutral/Sound film rewind.mp3'),
            icon: 'rewind.png'
          },
          {
            hotkey: '"',
            name: 'Wheel',
            file: require('@/assets/neutral/Spinning Wheel.mp3'),
            icon: 'wheel.png'
          },
          {
            hotkey: '_',
            name: 'Strategy',
            file: require('@/assets/neutral/Strategy.mp3')
          }
        ],
        [
          {
            hotkey: 'Z',
            name: 'Alam Niya',
            file: require('@/assets/music/Alam Nya - Chorus Cut.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'X',
            name: 'Sigaw ng Puso',
            file: require('@/assets/music/Sigaw ng Puso - Chorus cut.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'C',
            name: 'Heavenly Music',
            file: require('@/assets/music/Heavenly Music.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'V',
            name: 'Titanic flute',
            file: require('@/assets/music/Titanic flute.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'B',
            name: 'Maligayang Pagbati',
            file: require('@/assets/music/Maligayang Pagbati - 1.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'N',
            name: 'Chicken happy song',
            file: require('@/assets/music/Chicken happy song.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: 'M',
            name: 'Spongebob',
            file: require('@/assets/music/Spongebob.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: ',',
            name: 'Super Mario',
            file: require('@/assets/music/Super Mario.mp3'),
            icon: 'music.png'
          },
          {
            hotkey: ',',
            name: 'KNC Song',
            file: require('@/assets/music/KNC Song.mp3'),
            icon: 'music.png'
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
        if (effect.hotkey.toLowerCase() === char.toLowerCase()) {
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
  img {
    pointer-events: none;
  }
</style>
