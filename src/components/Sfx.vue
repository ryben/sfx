<template>
  <div id="root">

    <head>
      <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons" />
    </head>

    <div id="sfx_table" tabindex="0">
      <span id="menu_dropdown" class="dropdown">
        <button @click="show_config_dropdown()" class="dropbtn">
          ☰
        </button>
        <div id="config_dropdown" class="dropdown-content">
          <a @click="show_add_sfx_modal()">Add SFX</a>
          <label for="upload_config" style="cursor: pointer;">
            <a>
              Import Config
            </a>
          </label>
          <a @click="download()">Download current config</a>
          <a @click="clear()">Clear added effects</a>
        </div>
        <input id="upload_config" type="file" @change="upload" class="invisible" /><br />
      </span>

      <button @click="stop_all" class="stop_all_button">
        Stop all
        <br>
        <img src="../assets/stop.png" class="effect_icon">
        <br>
        [Esc]
      </button><br><br>

      <div class="slidecontainer" style="height: 100;">
        <i class="material-icons">volume_up</i>
        <input type="range" min="1" max="100" v-model="volume" class="slider" id="myRange"
          style="height: 23px; width: 150px;">
      </div>

      <div id="div_added_sfx" v-if="added_sfx.length > 0">
        <br><br>
        <button v-for="e in added_sfx" :key="e.hotkey" @click="play_effect(e)"
          @contextmenu="on_effect_right_click(e, $event)" class="effect">
          {{ e.name }}
          <br>
          <br>
          <img :src="get_img(e.icon)" class="effect_icon_small">
          <br>
          {{ e.hotkey }}
        </button>
      </div><br><br>

      <div v-for="group in effect_groups" :key="group.no">
        <button v-for="e in group" :key="e.hotkey" @click="play_effect(e)" class="effect"
          @contextmenu="$event.preventDefault()">
          {{ e.name }}
          <br />
          <img :src="get_img(e.icon)" class="effect_icon">
          <br />
          {{ e.hotkey }}
        </button>
      </div>
    </div><br><br><br>

    <modal name="modal_add_sfx" :height="470" :width="450" :adaptive="true">
      <div>
        <h3>
          <p>Add SFX</p>
        </h3>
        <br>
        Name<br>
        <input type="text" v-model="sfx_to_add.name" autocomplete="off" placeholder="Name"
          class="ui_text_input" /><br><br>
        File URL<br>
        <input type="text" autocomplete="off" v-model="sfx_to_add.url" placeholder="URL"
          class="ui_text_input"><br><br><br>
        <button class="ui_button_medium" @click="add_sfx(sfx_to_add)">Add SFX</button> <br>
        <br>
        <br>
      </div>
    </modal>

  </div>
</template>

<script>
import effectGroups from "@/assets/config.json"

const EFFECT_STATUS = {
  READY: 0,
  PLAYING: 1,
  STOPPING: 2,
  STOPPED: 3
}

const SFX_PROP = {
  NAME: 'name',
  URL: 'url',
  AUDIO: 'audio',
  BLOB: 'blob',
  STATUS: 'status'
}

const LOCAL_STORAGE = {
  CONFIG: 'config'
}

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      effect_groups: effectGroups,
      added_sfx: [],
      sfx_to_add: {
        name: "",
        url: "https://drive.google.com/uc?id=1WIdryvQHAP7h34SaIWeDV8rVQ9sypqQF"
      },
      playing: [],
      volume: 40
    }
  },
  computed: {
    all_sfx() {
      return [this.added_sfx].concat(this.effect_groups)
    },
    added_sfx_config() {
      return JSON.stringify(this.added_sfx,
        (key, value) => {
          if (key === SFX_PROP.AUDIO || value === "") {
            return undefined
          }
          return value
        },
        2)
    }
  },
  mounted() {
    this.load_from_local_storage()

    this.load_sfx()

    document.getElementById("sfx_table").addEventListener("keydown", function (e) {
      this.on_key_press(e)
    }.bind(this));

    window.onclick = function (event) {
      if (!event.target.matches('.dropbtn')) {
        var dropdowns = document.getElementsByClassName("dropdown-content");
        var i;
        for (i = 0; i < dropdowns.length; i++) {
          var openDropdown = dropdowns[i];
          if (openDropdown.classList.contains('show')) {
            openDropdown.classList.remove('show');
          }
        }
      }
    }
  },
  watch: {
    volume: function () {
      this.playing.forEach(p => {
        if (p.status != EFFECT_STATUS.STOPPING) {
          p.volume = this.get_volume()
        }
      })
    },
    added_sfx: function () {
      this.save_config_to_local_storage()
    }
  },
  methods: {
    load_sfx() {
      let sfx_list = this.effect_groups.flat()
      for (let i = 0; i < sfx_list.length; i++) {
        this.preload_effect(sfx_list[i])
      }
    },
    preload_effect(effect) {
      effect[SFX_PROP.AUDIO] = new Audio(effect[SFX_PROP.URL])
    },
    get_img(filename) {
      if (filename) {
        return filename
      } else {
        return require('@/assets/sound.png')
      }

    },
    get_volume() {
      return this.volume / 100
    },
    play_effect(effect) {
      let audio = effect[SFX_PROP.AUDIO]

      audio.volume = this.get_volume()

      // make sure audio is reset before playing
      audio.pause()
      audio.currentTime = 0
      audio[SFX_PROP.STATUS] = EFFECT_STATUS.READY

      this.playing.push(audio)
      audio.play()
    },
    show_add_sfx_modal() {
      this.sfx_to_add.name = "SFX " + (this.added_sfx.length + 1)
      this.$modal.show('modal_add_sfx')
    },
    async stop_all() {
      let all_playing = this.playing

      all_playing.forEach(p => {
        p[SFX_PROP.STATUS] = EFFECT_STATUS.STOPPING
      })

      let has_stoppable = true
      while (has_stoppable) {
        has_stoppable = false

        // Find marked as STOPPING and decrease volume
        all_playing.forEach(p => {
          if (p[SFX_PROP.STATUS] === EFFECT_STATUS.STOPPING) {
            if (p.volume > 0.1) {
              p.volume -= Math.min(0.05, p.volume)
            } else {
              p.volume -= Math.min(0.01, p.volume)
            }
            has_stoppable = true
          }
        })

        // remove all effects with 0 volume
        all_playing.forEach(effect => {
          if (effect.volume === 0) {
            var index = all_playing.indexOf(effect)
            all_playing.splice(index, 1)
          }
        })
        await this.sleep(50)
      }
    },
    on_effect_right_click(effect, event) {
      event.preventDefault();
    },
    on_key_press(e) {
      if (e.keyCode == 27) { // Detect Escape key pressed
        this.stop_all()
        return
      }

      let all_effects = this.effect_groups.flat()
      // Check if a shortcut matches
      for (let i = 0; i < all_effects.length; i++) {
        let effect = all_effects[i]
        if (effect.hotkey && String.fromCharCode(e.keyCode) &&
          effect.hotkey.toLowerCase() === String.fromCharCode(e.keyCode).toLowerCase()) {
          this.play_effect(effect)
          return
        }
      }

    },
    add_sfx(inp) {
      // prepare audio
      let sfx_new = {}
      sfx_new[SFX_PROP.NAME] = inp.name
      sfx_new[SFX_PROP.URL] = inp.url // TODO: Detect if Google Drive share link, auto transform to a download link
      // TODO: Error handling if URL provided is invalid

      this.preload_effect(sfx_new)
      this.added_sfx.push(sfx_new)

      this.sfx_to_add.name = ""
      this.sfx_to_add.url = ""

      this.$modal.hide('modal_add_sfx');

    },
    download() {
      var pom = document.createElement('a');
      pom.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(this.added_sfx_config));
      pom.setAttribute('download', "sfx_config.json");
      pom.click();
    },
    upload(event) {
      if (event.target.files.length !== 0) {
        let reader = new FileReader()
        reader.onload = (e) => {
          this.load_sfx_from_config(e.target.result)
        }
        reader.readAsText(event.target.files[0])
      }
    },
    load_sfx_from_config(json) {
      // perform json validation first
      if (!this.is_valid_json(json)) {
        alert("Invalid configuration format")
        return
      }

      // TODO: Perform validation of sfx format
      this.added_sfx = []
      JSON.parse(json).flat().forEach(sfx => {
        this.added_sfx.concat(this.add_sfx(sfx))
      })
    },
    clear() {
      if (confirm("Are you sure you want to clear all added SFX?") == true) {
        this.added_sfx = []
      }
    },
    is_valid_json(str) {
      try {
        JSON.parse(str);
      } catch (e) {
        return false;
      }
      return true;
    },
    show_config_dropdown() {
      document.getElementById("config_dropdown").classList.toggle("show");
    },
    save_config_to_local_storage() {
      localStorage.setItem(LOCAL_STORAGE.CONFIG, this.added_sfx_config)
    },
    load_from_local_storage() {
      this.load_sfx_from_config(localStorage.getItem(LOCAL_STORAGE.CONFIG))
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
  width: 300px;
}

.ui_button_medium {
  height: 50px;
  width: 200px;
}

.ui_text_input {
  height: 50px;
  width: 300px;
  padding-left: 10px;
  margin: 5px;
}

.ui_textarea {
  height: 200px;
  width: 300px;
  padding-left: 10px;
  margin: 5px;
  padding: 10px;
}

.effect_icon {
  height: 50px;
  width: 50px;
}

.effect_icon_small {
  height: 25px;
  width: 25px;
}

.add_effect_button {}

.download_button {
  height: 50px;
  width: 50px;
  cursor: pointer;
  border: none;
  background: none;
}

img {
  pointer-events: none;
}

#sfx_table {
  display: inline-block;
  background: WhiteSmoke;
  padding: 50px;
  margin-left: 50px;
  margin-right: 50px;
  border: 1px solid rgb(210, 210, 210);
}

#sfx_table:focus-within {
  border: 2px solid black;
  background-color: whitesmoke;
}

.dropbtn {
  padding-left: 16px;
  padding-right: 16px;
  padding-top: 10px;
  padding-bottom: 14px;
  font-size: 30px;
  border: none;
  cursor: pointer;
}

.dropbtn:hover,
.dropbtn:focus {
  background-color: #a5a5a5;
}

.dropdown {
  position: relative;
  display: inline-block;
}

.dropdown-content {
  display: none;
  position: absolute;
  background-color: #f1f1f1;
  min-width: 230px;
  overflow: auto;
  text-align: left;
  box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
  z-index: 1;
}

.dropdown-content a {
  color: black;
  padding: 12px 16px;
  text-decoration: none;
  display: block;
}

.dropdown a:hover {
  background-color: #ddd;
}

.show {
  display: block;
}

.invisible {
  opacity: 0;
  position: absolute;
  z-index: -1;
}

#menu_dropdown {
  float: left;
  margin-top: -50px;
  margin-left: -50px;
}
</style>
