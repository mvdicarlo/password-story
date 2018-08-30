<template>
<div class="text-center" name="leetCode">
    <h5>Great! Now lets make the password a little more interesting!</h5>
    <div>
        <p>To do this, lets add a little bit of <a href="https://en.wikipedia.org/wiki/Leet" target="_blank">Leetspeak</a>.</p>
    </div>
    <div class="text-center">
        <div class="btn-group" role="group" aria-label="leet selector">
            <button type="button" class="btn btn-secondary" v-for="word in passwordWords" @click="chooseWord(word)" v-bind:class="{ active: word.selected }">{{ word.original }}</button>
        </div>
        <div class="w-100">
            <small class="form-text">Click on some of the words and watch the password change.</small>
        </div>
        <div class="mt-2 altered-password">
            <p>
                <span v-for="word in passwordWords" v-bind:class="{ highlight: word.selected }">{{ getWord(word) }}</span>
            </p>
        </div>
        <button type="button" name="acceptBtn" class="btn" v-bind:class="{' btn-success': isPassing, 'btn-danger': !isPassing }" @click="$emit('leet-complete', getCompletePassword())" :disabled="!isPassing">
            <ion-icon v-if="isPassing" class="align-middle" name="checkmark"></ion-icon>
            <ion-icon v-if="!isPassing" class="align-middle" name="close"></ion-icon>
        </button>
    </div>
</div>
</template>

<script>
export default {
    name: 'LeetCode',
    props: ['password'],
    data: function() {
        return {
            passwordWords: []
        };
    },
    created: function() {
        const words = this.password.match(/[A-Z][a-z]*/g) || [];
        this.passwordWords = words.map(word => {
            return {
                original: word,
                leet: null,
                selected: false
            }
        });
    },
    computed: {
        isPassing: function() {
            return this.passwordWords.filter(w => w.selected).length >= Math.ceil(this.passwordWords.length / 2);
        },
        requiredCount: function() {
            return Math.ceil(this.passwordWords.length / 2);
        }
    },
    methods: {
        chooseWord: function(wordObj) {
            wordObj.selected ? wordObj.leet = wordObj.original : wordObj.leet = this.generateLeetSpeak(wordObj.original);
            wordObj.selected = !wordObj.selected;
        },
        getWord: function(wordObj) {
            return wordObj.selected ? wordObj.leet : wordObj.original;
        },
        getCompletePassword: function() {
            return this.passwordWords.map(word => word.leet ? word.leet : word.original).join('') + '!!';
        },
        generateLeetSpeak: function(word) {
            const leetWord = word.replace(/(A|a)/g, '4')
                .replace(/(E|e)/g, '3')
                .replace(/(I|i)/g, '1')
                // .replace(/(T|t)/g, '7')
                .replace(/(O|o)/g, '0')
                // .replace(/(S|s)/g, '5');

            return leetWord;
        }
    }
}
</script>

<style scoped>
.altered-password {
    font-size: 1.75em;
}
</style>
