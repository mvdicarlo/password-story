<template>
<div name="passwordReview">
    <h5>Lets try practicing what we have so far!</h5>
    <div class="w-100 text-center" style="font-size: 5em">
        <span v-for="c of correctCounter">
            <span v-if="c.correct" class="text-success"><ion-icon name="checkmark"></ion-icon></span>
            <span v-else class="text-danger"><ion-icon name="close"></ion-icon></span>
        </span>
    </div>
    <div class="form-group">
        <input class="form-control form-control-lg" type="text" v-bind:placeholder="toReview" v-model="userInput" @input="onInput" aria-describedby="review-addon" autofocus>
    </div>
</div>
</template>

<script>
export default {
    name: 'ReviewPassword',
    props: ['toReview', 'times'],
    data: function() {
        return {
            correct: 0,
            userInput: '',
            correctCounter: []
        };
    },
    created: function() {
        for (let i = 0; i < this.times; i++) {
            this.correctCounter.push({
                correct: false
            });
        }
    },
    methods: {
        onInput: function() {
            if (this.userInput === this.toReview) {
                this.userInput = '';
                this.correctCounter[this.correct].correct = true;
                this.correct += 1;
            }
        }
    },
    watch: {
        correct: function(newCount) {
            if (newCount >= this.times) {
                this.$emit('review-complete');
            }
        }
    }
}
</script>

<style scoped>
    .form-group {
        padding-bottom: 1rem;
        margin-bottom: 0rem;
    }
</style>
