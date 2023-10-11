<template>
    <div style="width: 100%; height: 100%;">
        <div style="margin-top: 50px;" >
            <svg height="300" width="300" style="display: block; margin: 0 auto;">
                <circle class="progress_back" cx="150" cy="150" :r=set_r stroke-width="22"
                    style="fill: none; stroke: #f3f3f3 " />
                <circle ref="progress_color_set" class="progress_color" cx="150" cy="150" :r=set_r stroke-width="22"
                    style="fill: none; stroke: #f9a885;" />
                <circle class="progress_back" cx="150" cy="150" :r=rep_r stroke-width="22"
                    style="fill: none; stroke: #ededed " />
                <circle ref="progress_color_rep" class="progress_color" cx="150" cy="150" :r=rep_r stroke-width="22"
                    style="fill: none; stroke: #60e372;" />
                <circle class="progress_back" cx="150" cy="150" :r=excute_r stroke-width="22"
                    style="fill: none; stroke: #e0e0e0 " />
                <circle ref="progress_color" class="progress_color" cx="150" cy="150" :r=excute_r stroke-width="22"
                    style="fill: none; stroke: #5976cc;" />
                <!-- <circle id="reset_circle" cx="150" cy="150" :r=excute_r-11 @click="excute_stop" style="background-color: red;" fill="white"></circle> -->
            </svg>
            
            <div v-if="schedule.length > 0 && current_index < schedule.length"
                style="position: absolute; top: 200px; left: 50%; transform: translate(-50%, -50%); text-align: center">
                <div>
                    {{ schedule[current_index].type }}
                </div>
                <div>
                    {{ (parseInt(current_time * 100) / 100).toFixed(2) }} / {{ schedule[current_index].value }}s
                </div>
                <div id="small_reset_button" @click="excute_stop" style="border-radius: 10px; padding: 5px 10px; margin-top:20px;">
                    {{ small_reset_state == false ? '리셋':'시작' }}
                </div>
            </div>
            <div v-else-if="current_index >= schedule.length"
                style="position: absolute; top: 200px; left: 50%; transform: translate(-50%, -50%); text-align: center">
                <div>
                    끝!
                </div>
            </div>
        </div>

        <div style="margin-top: 10px;">
            <div style="text-align: center;">
                <span v-if="schedule.length > 0 && current_index < schedule.length">
                    <span v-if="schedule[current_index].hasOwnProperty('rep')" style="margin-right: 10px;">
                        {{ schedule[current_index].rep + 1 }} / {{ number.rep_count }}랩
                    </span>
                    {{ schedule[current_index].set + 1 }} / {{ number.set_count }}세트
                </span>
            </div>
            <div>
                <div style="position: absolute; left: 50%; transform: translateX(-50%); background-color: #c4c4c4; height: 10px; width: 90%;">
                    <div ref="color_bar" class="color_bar">

                    </div>
                </div>
            </div>
            <div
                style="display: flex; justify-content: center;align-items: center; flex-wrap: wrap; overflow-y: scroll; height: calc(100vh - 50px - 300px - 50px - 80px - 25px - 40px); margin-top: 15px;" @scroll="(e) => e.stopPropagation()">
                <div v-for="(s, index) in schedule" :key="s" class="schedule_unit"
                    v-bind:class="[current_index < index ? 'schedule_complete' : 'schedule_to_do']" @click="index_change(index)">
                    <span v-if="s.hasOwnProperty('rep')">{{ s.type }} ( {{ s.set + 1 }}셋 {{ s.rep + 1 }}랩 ) {{ s.value }}s</span> 
                    <span v-else>{{ s.type }} ( {{ s.set + 1 }}셋 ) {{ s.value }}s</span>
                </div>
            </div>
        </div>

        <div id="main_button_container">
            <div class="ui_button" @click="chuushi">
                <div v-if="is_running">중지</div>
                <div v-else>재시작</div>
            </div>
            <div class="ui_button" @click="reset_button">
                <div>리셋</div>
            </div>
        </div>
        <div v-if="black_screen"
            style="z-index: 100000; display: flex; justify-content: center; align-items: center; position:fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgb(0,0,0,0.8);">
            <div style="color: white; font-size: 50px;">
                {{ counting_down }}
            </div>
        </div>
    </div>
</template>

<script>
import end_set from "@/assets/end_set.mp3";
import beeping from "@/assets/beep.mp3";
import end_ex from "@/assets/end_ex.mp3";
export default {
    props: {
        prop_number: {
            type: Object,
            default: () => {
                return {
                    set_count: 3,
                    set_interval: 2,
                    rep_count: 2,
                    rep_excute: 2,
                    rep_rest: 2
                }
            }
        },
        is_ring_on: Boolean,
        volume_value: {
            type: Number,
        }
    },
    data() {
        return {
            number: {
                set_count: 5,
                set_interval: 3,
                rep_count: 2,
                rep_excute: 2,
                rep_rest: 1
            },
            schedule: [],
            current_index: 0,
            screen_w: 0,
            excute_r: 100,
            rep_r: 110,
            set_r: 120,
            current_time: 0,
            interval: undefined,
            schedule_tree: {},
            is_running: true,
            sound_zip: {},
            counting_down: 5,
            start_interval: undefined,
            black_screen: true,
            small_reset_state: false,
            total_time: 0,
        }
    },
    watch: {
        volume_value (n) {
            this.sound_zip.beep.volume = n / 10;
            this.sound_zip.end_set.volume = n / 10;
            this.sound_zip.end_ex.volume = n / 10;
            this.sound_zip.small_beep.volume = n / 10;
        }
    },
    mounted() {
        this.screen_w = screen.width;
        for (let i = 0; i < this.number.set_count; i++) {
            for (let j = 0; j < this.number.rep_count; j++) {
                if (this.number.rep_excute > 0) this.schedule.push({ type: "운동", value: this.number.rep_excute, set: i, rep: j });
                if (this.number.rep_rest > 0) this.schedule.push({ type: "휴식", value: this.number.rep_rest, set: i, rep: j });
            }
            if (this.number.rep_rest > 0) this.schedule.pop();
            if (this.number.set_interval > 0) this.schedule.push({ type: "세트휴식", value: this.number.set_interval, set: i + 1 });
        }
        if (this.number.set_interval > 0) this.schedule.pop();
        if (this.schedule.length == 0) {
            alert("운동스케쥴을 확인해 주세요");
            this.reset_button();
            return;
        }
        this.schedule[0]['start_time'] = 0;
        this.total_time = this.schedule[0]['value'];
        for (let i = 1; i < this.schedule.length; i++) {
            this.schedule[i]['start_time'] = this.schedule[i-1]['start_time'] + this.schedule[i-1]['value'];
            this.total_time += this.schedule[i]['value'];
        }
        this.$refs.progress_color.style.strokeDasharray = this.excute_r * Math.PI * 2;
        this.$refs.progress_color_rep.style.strokeDasharray = this.rep_r * Math.PI * 2;
        this.$refs.progress_color_set.style.strokeDasharray = this.set_r * Math.PI * 2;
        this.progress(0);
        this.progress_rep(0);
        this.progress_set(0);
        this.start_interval = setInterval(() => {
            if (this.counting_down == 0) {
                clearInterval(this.start_interval);
                this.black_screen = false;
                this.interval = setInterval(() => {
                    this.progress(this.current_time);
                    this.current_time += 0.01;
                    // console.log(this.schedule[this.current_index]['start_time'], this.current_time, );
                    // this.$refs.color_bar.style['width'] = ((this.schedule[this.current_index]['start_time'] + this.current_time) / this.total_time * 100) + '%';
                }, 10);
                this.counting_down = 5;
                return;
            }
            this.counting_down--;
            this.sound_zip.small_beep.play();
        }, 1000)
    },
    created() {
        this.sound_zip = {
            end_set: new Audio(end_set),
            beep: new Audio(beeping),
            end_ex: new Audio(end_ex),
            small_beep: new Audio("data:audio/wav;base64,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qJrx4KOQjahZxWbcZgztj2c49nKmkId44S71j0c8eV9yDK6uPRzx5X18eDvjvQ6yKo9ZSS6l//8elePK/Lf//IInrOF/FvDoADYAGBMGb7FtErm5MXMlmPAJQVgWta7Zx2go+8xJ0UiCb8LHHdftWyLJE0QIAIsI+UbXu67dZMjmgDGCGl1H+vpF4NSDckSIkk7Vd+sxEhBQMRU8j/12UIRhzSaUdQ+rQU5kGeFxm+hb1oh6pWWmv3uvmReDl0UnvtapVaIzo1jZbf/pD6ElLqSX+rUmOQNpJFa/r+sa4e/pBlAABoAAAAA3CUgShLdGIxsY7AUABPRrgCABdDuQ5GC7DqPQCgbbJUAoRSUj+NIEig0YfyWUho1VBBBA//uQZB4ABZx5zfMakeAAAAmwAAAAF5F3P0w9GtAAACfAAAAAwLhMDmAYWMgVEG1U0FIGCBgXBXAtfMH10000EEEEEECUBYln03TTTdNBDZopopYvrTTdNa325mImNg3TTPV9q3pmY0xoO6bv3r00y+IDGid/9aaaZTGMuj9mpu9Mpio1dXrr5HERTZSmqU36A3CumzN/9Robv/Xx4v9ijkSRSNLQhAWumap82WRSBUqXStV/YcS+XVLnSS+WLDroqArFkMEsAS+eWmrUzrO0oEmE40RlMZ5+ODIkAyKAGUwZ3mVKmcamcJnMW26MRPgUw6j+LkhyHGVGYjSUUKNpuJUQoOIAyDvEyG8S5yfK6dhZc0Tx1KI/gviKL6qvvFs1+bWtaz58uUNnryq6kt5RzOCkPWlVqVX2a/EEBUdU1KrXLf40GoiiFXK///qpoiDXrOgqDR38JB0bw7SoL+ZB9o1RCkQjQ2CBYZKd/+VJxZRRZlqSkKiws0WFxUyCwsKiMy7hUVFhIaCrNQsKkTIsLivwKKigsj8XYlwt/WKi2N4d//uQRCSAAjURNIHpMZBGYiaQPSYyAAABLAAAAAAAACWAAAAApUF/Mg+0aohSIRobBAsMlO//Kk4soosy1JSFRYWaLC4qZBYWFRGZdwqKiwkNBVmoWFSJkWFxX4FFRQWR+LsS4W/rFRb/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////VEFHAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAU291bmRib3kuZGUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAwNGh0dHA6Ly93d3cuc291bmRib3kuZGUAAAAAAAAAACU="),
        }
        this.number = this.prop_number;
    },
    methods: {
        progress(per) {
            let progress = per / this.schedule[this.current_index].value;
            let dash_offset = this.excute_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color.style.strokeDashoffset = dash_offset;
            this.$refs.color_bar.style['width'] = ((this.schedule[this.current_index]['start_time'] + this.current_time) / this.total_time * 100) + '%';
            if (progress >= 1) {
                this.current_index += 1;
                if (this.current_index == this.schedule.length) {
                    this.progress_rep(this.number.rep_count-1);
                    this.progress_set(this.number.set_count-1);
                    clearInterval(this.interval);
                    if (this.is_ring_on) {
                        this.sound_zip.beep.play();
                        this.sound_zip.end_ex.play();
                    }
                    return;
                }
                if (this.schedule[this.current_index - 1].rep != this.schedule[this.current_index].rep) {
                    this.progress_rep(this.schedule[this.current_index].rep);
                    if (this.is_ring_on) {
                        this.sound_zip.beep.play();
                    }
                }
                if (this.schedule[this.current_index - 1].set != this.schedule[this.current_index].set) {
                    this.progress_set(this.schedule[this.current_index].set);
                    if (this.is_ring_on) {
                        this.sound_zip.end_set.play();
                    }
                }
                if (this.schedule[this.current_index - 1].rep == this.schedule[this.current_index].rep && this.schedule[this.current_index - 1].set == this.schedule[this.current_index].set && this.is_ring_on) {
                    this.sound_zip.beep.play();
                }
                this.current_time = 0;
            }
        },
        progress_rep(per) {
            let progress = (per + 1) / this.number.rep_count;
            let dash_offset = this.rep_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color_rep.style.strokeDashoffset = dash_offset;
        },
        progress_set(per) {
            let progress = (per + 1) / this.number.set_count;
            let dash_offset = this.set_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color_set.style.strokeDashoffset = dash_offset;
        },
        reset_button() {
            clearInterval(this.interval);
            this.$emit("reset_button");
        },
        chuushi() {
            if (this.is_running) {
                // alert(this.schedule[this.current_index]['start_time'] + this.current_time);
                clearInterval(this.interval);
                this.is_running = false;
                return;
            }
            this.is_running = true;
            // this.interval = setInterval(() => {
            //     this.progress(this.current_time);
            //     this.current_time += 0.01;
            // }, 10);
            this.black_screen = true;
            this.start_interval = setInterval(() => {
                if (this.counting_down == 0) {
                    clearInterval(this.start_interval);
                    this.black_screen = false;
                    this.interval = setInterval(() => {
                        this.progress(this.current_time);
                        this.current_time += 0.01;
                        // this.$refs.color_bar.style['width'] = ((this.schedule[this.current_index]['start_time'] + this.current_time) / this.total_time * 100) + '%';
                    }, 10);
                    this.counting_down = 5;
                    return;
                }
                this.counting_down--;
                this.sound_zip.small_beep.play();
            }, 1000)
        },
        excute_stop() {
            // this.small_reset_state = !this.small_reset_state;
            // if (this.small_reset_state == true) {
            //     this.chuushi();
            //     this.current_time = 0;
            //     this.progress(0);
            //     clearInterval(this.interval);
            // } else {
            //     this.chuushi();
            // }
            this.current_time = 0;
            this.progress(0);
        },
        index_change(index) {
            this.current_index = index;
            this.current_time = 0;
            this.progress(0);
            this.progress_rep(this.schedule[index].rep);
            this.progress_set(this.schedule[index].set);
            clearInterval(this.interval);
            this.is_running = false;
        }
    }
}
</script>

<style scoped>
body {
    overscroll-behavior: none;
    -webkit-user-select: none;
}
#main_button_container {
    display: flex;
    justify-content: space-between;
    width: 80%;
    height: 50px;
    margin: 0 auto;
    position: fixed;
    left: 0;
    right: 0;
    bottom: 80px;
}

.ui_button {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 45%;
    height: 100%;
    background-color: rgb(118, 170, 210);
    color: white;
    border-radius: 10px;
}
.ui_button:hover {
    cursor: pointer;
    background-color: rgb(107, 153, 188);
}

.timer_group {
    width: 95%;
    aspect-ratio: 1 / 1;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
}

svg {
    transform: rotate(-90deg);
}

.progress_color {
    stroke-linecap: round;
}

.schedule_unit {
    margin: 3px 10px;
    padding: 5px 10px;
    border-radius: 10px;
}

.schedule_complete {
    background: rgb(236, 236, 236);
}

.schedule_to_do {
    background: rgb(0, 0, 0, 0.8);
    color: rgb(120, 120, 120);
}

/* #reset_circle:hover {
    cursor: pointer;
    fill: rgb(0, 0, 0, 0.2);
} */

.schedule_unit:hover {
    cursor: pointer;
    filter: brightness(0.5);
}

#small_reset_button {
    background-color: #adebfd;
}
#small_reset_button:hover {
    cursor: pointer;
    background-color: #96cfe1;
}
.color_bar {
    height: 10px;
    background-color: rgb(192, 94, 94);
}
</style>