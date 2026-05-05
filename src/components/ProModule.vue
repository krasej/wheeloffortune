<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits(['options'])

const props = defineProps({
    options: {
        type: Array,
        required: true
    }
})

const optionsInput = ref(props.options.join(', '));
const isOpen = ref(false);
const error = ref('');


function formatOptions() {
    let options = optionsInput.value
        .split(',')
        .map(option => option.trim())
        .filter(option => option !== '')

    options = options.map(option => {
        if (option.length > 25) {
            return option.substring(0, 25) + '...'
        }
        return option
    })

    if (options.length > 0) {
        error.value = ''
        emit('options', options)
        optionsInput.value = ''
        isOpen.value = false
    } else {
        error.value = 'Please enter at least one option.'
    }
}
</script>
<template>
    <div class="pro-button-container">
        <button class="pro-button" @click="isOpen = !isOpen">Show pro mode</button>
    </div>
    <div class="pro-module" v-show="isOpen">
        <div class="pro-instructions">
            <p>Enter all options at once, separated by commas. For example: "Option 1, Option 2, Option 3". This will
                replace all existing options at once!</p>
        </div>
        <div v-if="error" class="error">
            {{ error }}
        </div>
        <form @submit.prevent="formatOptions" class="pro-form">
            <textarea class="pro-text-input" rows="4" v-model="optionsInput"
                placeholder="Enter all options at once, separated by comma"></textarea>
            <button class="add-button" type="submit">Format</button>
        </form>
    </div>


</template>

<style lang="css" scoped>
.pro-text-input {
    margin-top: 20px;
    flex: 1;
    width: 100%;
    width: fill-available;
    box-sizing: border-box;
}

.pro-button-container {
    margin-top: 10px;
    margin-bottom: 10px;
}

.error {
    color: var(--color-button-accent);
    margin-top: 10px;
}

.pro-form {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.pro-button {
    cursor: pointer;
    padding: 5px 10px;
    background-color: transparent;
    border: 1px solid var(--color-button-accent);
    color: var(--color-button-accent);
    border-radius: 5px;
    flex: 1;
}

.add-button {
    margin-top: 10px;
    width: 100%;
}
</style>