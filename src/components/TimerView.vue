<template>
    <div style="width: 100%; height: 100%;">
        <div style="border: solid;">
            <div v-if="schedule.length > 0 && current_index < schedule.length" style="position: absolute; top: 150px; left: 50%; transform: translate(-50%, -50%); text-align: center">
                <div>
                    {{ schedule[current_index].type }}
                </div>
                <div>
                    {{ parseInt(current_time) }} / {{ schedule[current_index].value }}
                </div>
            </div>
            <svg height="300" width="300" style="display: block; border:dashed; margin: 0 auto;">
                <circle class="progress_back" cx="150" cy="150" :r=excute_r stroke-width="22" style="fill: none; stroke: #e0e0e0 "/>
                <circle ref="progress_color" class="progress_color" cx="150" cy="150" :r=excute_r stroke-width="22" style="fill: none; stroke: #7be679;"/>
            </svg>
        </div>
        
        <div id="main_button_container">
            <div class="ui_button">
                <div>중지</div>
            </div>
            <div class="ui_button">
                <div>리셋</div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            number: {
                set_count: 2,
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
        }
    },
    mounted() {
        this.screen_w = screen.width;
        console.log(this.screen_w);
        for (let i = 0; i < this.number.set_count; i++) {
            for (let j = 0; j < this.number.rep_count; j++) {
                this.schedule.push({ type: "excute", value: this.number.rep_excute });
                this.schedule.push({ type: "rest", value: this.number.rep_rest });
            }
            this.schedule.pop();
            this.schedule.push({ type: "set_rest", value: this.number.set_interval });
        }
        this.schedule.pop();
        console.log(this.schedule);
        this.$refs.progress_color.style.strokeDasharray = this.excute_r * Math.PI * 2;
        this.interval = setInterval(() => {
            this.progress(this.current_time);
            this.current_time += 0.01;
        }, 10);
    },
    methods: {
        progress(per) {
            let progress = per / this.schedule[this.current_index].value;
            let dash_offset = this.excute_r * Math.PI * 2 * (1 - progress);
            this.$refs.progress_color.style.strokeDashoffset = dash_offset;
            if (progress >= 1) {
                this.current_index += 1; 
                if (this.current_index == this.schedule.length) {
                    clearInterval(this.interval);
                }
                this.current_time = 0;
            }
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
    border: dashed;
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
</style>