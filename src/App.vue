<template>
<div id="app" v-bind:class="['hsimp-level', hsimpLevel ? 'text-white' : 'text-dark', hsimpLevel]">
    <div class="container">
        <div class="text-center mb-3">
            <h1>Password Story</h1>
            <h4>Lets build a password together!</h4>
        </div>
        <div name="hsimp" class="d-none">
            <input id="password" type="text" v-bind:value="password">
        </div>
        <div class="text-center highlight" style="font-size: 3em" v-if="step <= 4">
            <span>{{ password }}</span>
            <span v-if="step > 1" class="pl-3 align-middle"><button type="button" name="restart" class="btn btn-danger" @click="restart()">Scrap This Password</button></span>
        </div>
        <div class="text-center alert alert-info infos" role="alert" v-if="timeToCrack">
            <div v-if="timeToCrack === 'INSTANTLY'">
                Your password would be cracked
                <h3>{{ timeToCrack }}</h3>
            </div>
            <div v-else>
                It would take a computer about
                <h3>{{ timeToCrack }}</h3> to crack your password
            </div>

        </div>
        <StoryGenerator v-if="step === 1" v-on:story-complete="incrementStep" />
        <LeetCode v-if="step === 3" :password="password" v-on:leet-complete="incrementStep" />
        <ReviewPassword v-if="reviewPassword" :toReview="password" times="3" v-on:review-complete="reviewCompleted" />

        <div class="text-center" v-if="step > 4">
            <p v-if="!reviewPassword">Congratulations! You have successfully completed your password story!</p>
            <small class="d-block">Click on the button below to copy your password to your clipboard!</small>
            <button class="btn btn-info btn-lg" type="button" @click="copy()">{{ password }}</button>
            <br>
            <button class="btn btn" type="button" @click="reviewPassword = true" v-if="!reviewPassword">I want to practice some more!</button>
            <br>
            <button type="button" name="restart" class="btn btn-danger" @click="restart()">Scrap This Password</button>
        </div>
    </div>
</div>
</template>

<script>
import StoryGenerator from './components/StoryGenerator.vue'
import ReviewPassword from './components/ReviewPassword.vue'
import LeetCode from './components/LeetCode.vue'

export default {
    name: 'app',
    components: {
        StoryGenerator,
        ReviewPassword,
        LeetCode
    },
    data: function() {
        return {
            step: 1,
            password: null,
            reviewPassword: false,
            hsimpLevel: null,
            timeToCrack: null
        };
    },
    mounted: function() {
        window.hsimp({
            options: {
                calculationsPerSecond: 1e10, // 10 billion,
                good: 31557600e3, // 1,000 years
                ok: 31557600 // 1 year
            },
            outputTime: function(time, input) {
                this.timeToCrack = time;
                const classes = input.className.split(' ');
                for (let i = 0; i < classes.length; i++) {
                    const c = classes[i];
                    if (c.includes('hsimp-level--')) {
                        this.hsimpLevel = c;
                        break;
                    }
                }
            }.bind(this),
            outputChecks: function(checks, input) {
                // console.log(checks, input);
            }
        }, document.getElementById("password"));
    },
    methods: {
        copy: function() {
            const el = document.createElement('textarea');
            el.value = this.password;
            document.body.appendChild(el);
            el.select();
            const success = document.execCommand('copy');
            document.body.removeChild(el);

            if (success) {
                window.alertify.notify('Password copied', 'success', 5);
            } else {
                window.alertify.notify('Unable to copy password to clipboard', 'danger', 5)
            }
        },
        fakeEvent: function() {
            setTimeout(function() {
                const element = document.getElementById('password');
                element.dispatchEvent(new Event('keyup'));
            }, 100);
        },
        incrementStep: function(password) {
            if (password) {
                this.password = password;
                this.step += 1;
                this.reviewPassword = true;

                this.fakeEvent();
            }
        },
        restart: function() {
            window.alertify.confirm('Start Over?', 'Are you sure you want to start over?', function() {
                this.step = 1;
                this.password = null;
                this.timeToCrack = null;
                this.hsimpLevel = null;
                this.reviewPassword = false;
            }.bind(this), function() {});
        },
        reviewCompleted: function() {
            this.step += 1;
            this.reviewPassword = false;
        }
    }
}
</script>

<style>
#app {
    margin: auto;
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 10;
    overflow: auto;
}

.infos {
    background: rgba(255, 255, 255, 0.6);
    border: transparent;
}

.highlight {
    color: #f99e63;
}

.hsimp-level--bad *[class*="text-danger"] {
    color: #e66f7a !important;
}

.hsimp-level--bad *[class*="btn-danger"] {
    background-color: #e66f7a !important;
}

body,
html {
    margin: 0;
    padding: 0;
    /* overflow: hidden; */
    /* height: 100%; */
}
</style>
