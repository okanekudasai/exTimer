<template>
    <div style="width: 100%; height: 100%;">
        <div id="set_container" style="margin-top: 140px;">
            <div class="makanai">
                <div>
                    <img src="@/assets/set_image.svg" />
                </div>
                <div>
                    세트 수(회)
                </div>
                <div style="margin-left: 20px;">
                    <div @click="minusing_set_count">
                        <img src="@/assets/minus_image.svg" />
                    </div>
                </div>
                <div>
                    <input class="makanai_number" id="set_count" v-model="number.set_count" type="text">
                </div>
                <div>
                    <div @click="plusing_set_count">
                        <img src="@/assets/plus_image.svg" />
                    </div>
                </div>
            </div>
            <div class="makanai">
                <div>
                    <img src="@/assets/set_image.svg" />
                </div>
                <div>
                    세트 간격(초)
                </div>
                <div style="margin-left: 20px;">
                    <div @click="minusing_set_interval">
                        <img src="@/assets/minus_image.svg" />
                    </div>
                </div>
                <div>
                    <input class="makanai_number" id="set_count" v-model="number.set_interval" type="text">
                </div>
                <div>
                    <div @click="plusing_set_interval">
                        <img src="@/assets/plus_image.svg" />
                    </div>
                </div>
            </div>
        </div>

        <div id="rep_container">
            <div class="makanai">
                <div>
                    <img src="@/assets/set_image.svg" />
                </div>
                <div>
                    반복 수(회)
                </div>
                <div style="margin-left: 20px;">
                    <div @click="minusing_rep_count">
                        <img src="@/assets/minus_image.svg" />
                    </div>
                </div>
                <div>
                    <input class="makanai_number" id="set_count" v-model="number.rep_count" type="text">
                </div>
                <div>
                    <div @click="plusing_rep_count">
                        <img src="@/assets/plus_image.svg" />
                    </div>
                </div>
            </div>
            <div class="makanai">
                <div>
                    <img src="@/assets/set_image.svg" />
                </div>
                <div>
                    운동 시간(초)
                </div>
                <div style="margin-left: 20px;">
                    <div @click="minusing_rep_excute">
                        <img src="@/assets/minus_image.svg" />
                    </div>
                </div>
                <div>
                    <input class="makanai_number" id="set_count" v-model="number.rep_excute" type="text">
                </div>
                <div>
                    <div @click="plusing_rep_excute">
                        <img src="@/assets/plus_image.svg" />
                    </div>
                </div>
            </div>

            <div class="makanai">
                <div>
                    <img src="@/assets/set_image.svg" />
                </div>
                <div>
                    휴식 시간(초)
                </div>
                <div style="margin-left: 20px;">
                    <div @click="minusing_rep_rest">
                        <img src="@/assets/minus_image.svg" />
                    </div>
                </div>
                <div>
                    <input class="makanai_number" id="set_count" v-model="number.rep_rest" type="text">
                </div>
                <div>
                    <div @click="plusing_rep_rest">
                        <img src="@/assets/plus_image.svg" />
                    </div>
                </div>
            </div>
        </div>

        <div id="preset_button" @click="preset_button">
            <div>프리셋</div>
        </div>

        <div id="start_button" @click="starting">
            <div>
                시작
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        
        data() {
            return {
                number: {
                    set_count: 1,
                    set_interval: 0,
                    rep_count: 1,
                    rep_excute: 0,
                    rep_rest: 0
                },
            }
        },
        methods: {
            plusing_set_count() {
                this.number.set_count++;
            },
            minusing_set_count() {
                if (this.number.set_count <= 1) {
                    this.number.set_count = 1;
                    return;
                } 
                this.number.set_count--;
            },

            plusing_set_interval() {
                this.number.set_interval++;
            },
            minusing_set_interval() {
                if (this.number.set_interval <= 0) {
                    this.number.set_interval = 0;
                    return;
                } 
                this.number.set_interval--;
            },

            plusing_rep_count() {
                this.number.rep_count++;
            },
            minusing_rep_count() {
                if (this.number.rep_count <= 1) {
                    this.number.rep_count = 1;
                    return;
                } 
                this.number.rep_count--;
            },

            plusing_rep_excute() {
                this.number.rep_excute++;
            },
            minusing_rep_excute() {
                if (this.number.rep_excute <= 0) {
                    this.number.rep_excute = 0;
                    return;
                } 
                this.number.rep_excute--;
            },

            plusing_rep_rest() {
                this.number.rep_rest++;
            },
            minusing_rep_rest() {
                if (this.number.rep_rest <= 0) {
                    this.number.rep_rest = 0;
                    return;
                } 
                this.number.rep_rest--;
            },
            starting() {
                this.$emit('startingTimer', this.number)
            },
            preset_button() {
                this.number = {
                    set_count: 5,
                    set_interval: 60,
                    rep_count: 20,
                    rep_excute: 15,
                    rep_rest: 15
                }
            }
        }
    }
</script>

<style scoped>
    .makanai {
        margin: 40px 0;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .makanai_number {
        border: none;
        border-bottom: solid;
        width: 60px;
        text-align: center;
        font-size: 20px;
    }
    #start_button {
        width: 60%;
        margin: 0 auto;
        display: flex;
        height: 50px;
        background:rgb(118, 170, 210);
        color: white;
        justify-content: center;
        align-items: center;
        border-radius: 10px;
        position: fixed;
        bottom: 80px;
        left: 0;
        right: 0;
    }
    #start_button:hover {
        cursor: pointer;
        background-color: rgb(107, 153, 188);
    }
    #preset_button {
        display: flex; justify-content: center; align-items: center; border-radius: 10px; color: white; margin: 0 auto; width: 30%; height: 40px; background-color: rgb(171, 125, 203);
    }
    #preset_button:hover {
        cursor: pointer;
        background-color: rgb(140, 104, 167);
    }
</style>