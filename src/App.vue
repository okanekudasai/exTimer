<template>
  <div id="main_container" style="height: 100vh; width: 100vw; position: fixed; top: 0; left: 0;">
    <div style="position: absolute; right: 20px; top: 20px;">
      <img class="bell_image" v-if="ring_sound" src="@/assets/bell_sound.svg" @click="toggle_sound">
      <img class="bell_image" v-else src="@/assets/bell_not_sound.svg" @click="toggle_sound">
    </div>

    <SettingView v-if="setting_view" @starting-timer="starting_timer"></SettingView>
    <TimerView :prop_number="number" :is_ring_on="ring_sound" v-else @reset_button="reset_button"></TimerView>

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
      setting_view: true,
      number: {},
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
    }
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