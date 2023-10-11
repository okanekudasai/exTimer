<template>
  <div id="main_container" style="height: 100vh; width: 100vw; position: fixed; top: 0; left: 0;">

    <SettingView v-if="setting_view" @starting-timer="starting_timer"></SettingView>
    <TimerView :prop_number="number" :is_ring_on="ring_sound" :volume_value="Number(volume_value)" v-else @reset_button="reset_button"></TimerView>

    
    <div style="position: absolute; right: 20px; top: 20px;">
      <div>
        <input ref="volume_range" type="range" style="width: 80px" min="0" max="10" v-model="volume_value">
      </div>
      <div style="text-align:center">
        <img class="bell_image" v-if="ring_sound" src="@/assets/bell_sound.svg" @click="toggle_sound">
        <img class="bell_image" v-else src="@/assets/bell_not_sound.svg" @click="toggle_sound">
      </div>
      
    </div>
  </div>
</template>

<script>
import SettingView from "@/components/SettingView.vue"
import TimerView from "@/components/TimerView.vue"
export default {
  components: {
    SettingView,
    TimerView
  },
  data() {
    return {
      ring_sound: true,
      // ring_sound: false,
      volume_value: 10,
      // setting_view: false,
      setting_view: true,
      number: {},
    }
  },
  watch: {
    ring_sound(b) {
      if (b == false) {
        this.$refs.volume_range.disabled = true;
      } else {
        this.$refs.volume_range.disabled = false;
      }
    }
  },
  methods: {
    toggle_sound() {
      this.ring_sound = !this.ring_sound;
    },
    starting_timer(number) {
      this.number = number;
      this.setting_view = false;
    },
    reset_button() {
      this.setting_view = true;
    },
    // change_volume_value() {
    //   console.log(this.volume_value);
    // }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
}

input:focus {
  outline: none
}
</style>

<style scoped>
.bell_image {
  width: 50px;
}
</style>