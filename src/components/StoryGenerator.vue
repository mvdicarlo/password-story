<template>
<div name="storyGenerator">
    <div class="form-group">
        <label for="passwordSize">How long would you like the password to be?</label>
        <input type="range" class="form-control-range" name="passwordSize" id="passwordSize" v-model="length" min="4" max="30" step="1" @input="generatePasswordWords()">
        <div class="text-center">
            {{ length }}
        </div>
    </div>
    <h4>Write a brief story to this prompt:</h4>
    <p v-if="loading">Generating story prompt...</p>
    <p v-else>
        {{ story.character }} who has a {{ story.prop.toLowerCase() }}. And to make it more interesting, why don't you make it so {{ (story.bonus || '').toLowerCase() }}
        <span @click="load()" class="text-success clickable" title="Generate a new story prompt"><ion-icon class="align-middle" name="refresh"></ion-icon></span>
    </p>
    <div class="form-group">
        <textarea autofocus v-model="userStory" v-on:input="generatePasswordWords()" class="form-control" rows="3" placeholder="A brief story" required></textarea>
        <small class="form-text text-muted">Don't forget to punctuate your story.</small>
    </div>
    <div class="text-center">
        <button v-if="generatedPassword" type="button" name="acceptBtn" class="btn btn-success" @click="$emit('story-complete', generatedPassword)">
            <ion-icon class="align-middle" name="checkmark"></ion-icon>
        </button>
        <p class="highlighted-password" v-if="generatedPassword">
            <span style="font-size: 3em">{{ generatedPassword }}</span>
            <span @click="generatePasswordWords()" class="text-success clickable" title="Generate a new one"><ion-icon class="align-middle" name="refresh"></ion-icon></span>
            <br>
            <small class="form-text text-danger">This is not the final password. Press the checkmark and we will spice it up a bit.</small>
        </p>
        <p v-else-if="generatedPassword !== null" class="text-danger">Unable to create a good starting password. Please try fleshing out your story some more.</p>
    </div>
</div>
</template>

<script>
export default {
    name: 'StoryGenerator',
    data: function() {
        return {
            loading: true,
            length: 16,
            story: null,
            userStory: null,
            generatedPassword: null,
        }
    },
    created: function() {
        this.load();
    },
    computed: {
        isDone: function() {
            return this.userStory &&
                this.userStory.replace(' ', '').length > this.length &&
                this.userStory.split(' ').length >= 3 &&
                (this.userStory.includes('!') || this.userStory.includes('.') || this.userStory.includes('?'));
        }
    },
    methods: {
        generatePasswordWords: async function() {
            if (!this.isDone) {
                this.generatedPassword = '';
                return;
            }

            const words = this.userStory.replace(/(\.|!|\?)/g, '').split(' ') || [];
            const wordsCopy = [...words];
            const randomWords = [];
            while (randomWords.join('').length < this.length - 2) {
                const sliced = words.splice(Math.floor((Math.random() * words.length)), 1);
                if (sliced.length > 0) {
                    if (!randomWords.includes(sliced[0])) { // skip already chosen words
                        randomWords.push(sliced[0].replace(/\W/g, ''));
                    }
                } else {
                    this.generatedPassword = '';
                    return;
                }
            }

            randomWords.sort((a, b) => {
                const aIndex = wordsCopy.indexOf(a);
                const bIndex = wordsCopy.indexOf(b);

                if (aIndex < bIndex) return -1;
                if (aIndex > bIndex) return 1;
                return 0;
            });

            this.generatedPassword = randomWords.map(word => window._.capitalize(word)).join('') + '!!';

            if (this.generatedPassword.length > this.length) {
                this.generatePasswordWords();
            }
        },
        load: function() {
            let vm = this;
            this.loading = true;
            window.axios.get('https://thestoryshack.com/api/writing-prompt-generator')
                .then((response) => {
                    vm.story = response.data;
                    vm.loading = false;
                }).catch(() => {
                    alert('We were unable to generate a story. Please come back later.')
                });
        }
    }
}
</script>

<style scoped>
.clickable {
    cursor: pointer;
}

.highlighted-password:not(span) {
    color: #f99e63;
}
</style>
