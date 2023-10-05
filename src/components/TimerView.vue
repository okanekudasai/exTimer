<template>
    <div style="width: 100%; height: 100%;">
        <div style="margin-top: 50px;" >
            <div v-if="schedule.length > 0 && current_index < schedule.length" style="position: absolute; top: 200px; left: 50%; transform: translate(-50%, -50%); text-align: center">
                <div>
                    {{ schedule[current_index].type }}
                </div>
                <div>
                    {{ (parseInt(current_time * 100) / 100).toFixed(2) }} / {{ schedule[current_index].value }}s
                </div>
            </div>
            <div v-else-if="current_index >= schedule.length" style="position: absolute; top: 200px; left: 50%; transform: translate(-50%, -50%); text-align: center">
                <div>
                    끝!
                </div>
            </div>
            <svg height="300" width="300" style="display: block; margin: 0 auto;">
                <circle class="progress_back" cx="150" cy="150" :r=set_r stroke-width="22" style="fill: none; stroke: #f3f3f3 "/>
                <circle ref="progress_color_set" class="progress_color" cx="150" cy="150" :r=set_r stroke-width="22" style="fill: none; stroke: #f9a885;"/>
                <circle class="progress_back" cx="150" cy="150" :r=rep_r stroke-width="22" style="fill: none; stroke: #ededed "/>
                <circle ref="progress_color_rep" class="progress_color" cx="150" cy="150" :r=rep_r stroke-width="22" style="fill: none; stroke: #dbec7b;"/>
                <circle class="progress_back" cx="150" cy="150" :r=excute_r stroke-width="22" style="fill: none; stroke: #e0e0e0 "/>
                <circle ref="progress_color" class="progress_color" cx="150" cy="150" :r=excute_r stroke-width="22" style="fill: none; stroke: #7be679;"/>
            </svg>
        </div>

        <div style="margin-top: 10px;">
            <div style="text-align: center;">스케쥴 |
                <span v-if="schedule.length > 0 && current_index < schedule.length" >
                    <span v-if="schedule[current_index].hasOwnProperty('rep')">
                        {{ schedule[current_index].rep }} / {{ number.rep_count }}랩
                    </span>
                    | {{ schedule[current_index].set }} / {{ number.set_count }}세트
                </span>
            </div>
            <div style="display: flex; justify-content: center;align-items: center; flex-wrap: wrap; overflow-y: scroll; height: calc(100vh - 50px - 300px - 50px - 80px - 25px - 40px);">
                <div v-for="(s, index) in schedule" :key="s" class="schedule_unit" v-bind:class="[current_index < index ? 'schedule_complete' : 'schedule_to_do']">
                    {{ s.type }} : {{ s.value }}s
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
    </div>
</template>

<script>
import do_rest from "@/assets/do_rest.mp3";
import do_excute from "@/assets/excute.mp3";
import end_rep from "@/assets/end_rep.mp3";
import end_set from "@/assets/end_set.mp3";
import beeping from "@/assets/beep.mp3";
import end_ex from "@/assets/end_ex.mp3";
export default {
    props: {
        prop_number: {
            type: Object,
            default: () => {
                return {
                    set_count: 0,
                    set_interval: 0,
                    rep_count: 0,
                    rep_excute: 0,
                    rep_rest: 0
                }
            }
        },
        is_ring_on: Boolean
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
            sound_zip: {}
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
            if(this.number.set_interval > 0) this.schedule.push({ type: "세트휴식", value: this.number.set_interval, set: i});
        }
        if(this.number.set_interval > 0) this.schedule.pop();
        if (this.schedule.length == 0) return;
        this.$refs.progress_color.style.strokeDasharray = this.excute_r * Math.PI * 2;
        this.$refs.progress_color_rep.style.strokeDasharray = this.rep_r * Math.PI * 2;
        this.$refs.progress_color_set.style.strokeDasharray = this.set_r * Math.PI * 2;
        this.progress(0);
        this.progress_rep(0);
        this.progress_set(0);
        this.interval = setInterval(() => {
            this.progress(this.current_time);
            this.current_time += 0.01;
        }, 10);
    },
    created() {
        this.sound_zip = {
            do_rest: new Audio(do_rest),
            do_excute: new Audio(do_excute),
            end_rep: new Audio(end_rep),
            end_set: new Audio(end_set),
            beep: new Audio(beeping),
            end_ex: new Audio(end_ex)
        }
        this.number = this.prop_number;
    },
    methods: {
        progress(per) {
            let progress = per / this.schedule[this.current_index].value;
            let dash_offset = this.excute_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color.style.strokeDashoffset = dash_offset;
            if (progress >= 1) {
                this.current_index += 1; 
                if (this.current_index == this.schedule.length) {
                    this.progress_rep(this.number.rep_count);
                    this.progress_set(this.number.set_count);
                    clearInterval(this.interval);
                    if (this.is_ring_on) {
                        this.sound_zip.beep.play();
                        this.sound_zip.end_ex.play();  
                    }
                    return;
                }
                if (this.schedule[this.current_index-1].rep != this.schedule[this.current_index].rep) {
                    this.progress_rep(this.schedule[this.current_index].rep);
                    if (this.is_ring_on) {
                        this.sound_zip.beep.play();
                    }
                }
                if (this.schedule[this.current_index-1].set != this.schedule[this.current_index].set) {
                    this.progress_set(this.schedule[this.current_index].set);
                    if (this.is_ring_on) {
                        this.sound_zip.end_set.play();
                    }
                }
                if (this.schedule[this.current_index-1].rep == this.schedule[this.current_index].rep && this.schedule[this.current_index-1].set == this.schedule[this.current_index].set && this.is_ring_on) {
                    console.log(this.schedule[this.current_index]);
                    this.sound_zip.beep.play();
                }
                this.current_time = 0;
            }
        },
        progress_rep(per) {
            let progress = per / this.number.rep_count;
            let dash_offset = this.rep_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color_rep.style.strokeDashoffset = dash_offset;
        },
        progress_set(per) {
            let progress = per / this.number.set_count;
            let dash_offset = this.set_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color_set.style.strokeDashoffset = dash_offset;
        },
        reset_button() {
            clearInterval(this.interval);
            this.$emit("reset_button");
        },
        chuushi() {
            if (this.is_running) {
                clearInterval(this.interval);
                this.is_running = false;
                return;
            }
            this.is_running = true;
            this.interval = setInterval(() => {
                this.progress(this.current_time);
                this.current_time += 0.01;
            }, 10);
        }
    }
}
</script>

<style scoped>
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
</style>