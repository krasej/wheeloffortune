<script setup lang="ts">
import { ref, watch } from 'vue'

const emit = defineEmits(['options'])

const props = defineProps({
    options: {
        type: Array as () => string[], 
        required: true
    }
})

const optionsInput = ref(props.options.join(', '));
const isOpen = ref(false);
const error = ref('');


watch(() => props.options, (newOptions) => {
    optionsInput.value = newOptions.join(', ');
}, { deep: true });

function formatOptions() {
    let processedOptions = optionsInput.value
        .split(',')
        .map(option => option.trim())
        .filter(option => option !== '')
        .map(option => (option.length > 40 ? option.substring(0, 40) + '...' : option));

    if (processedOptions.length > 0) {
        error.value = '';
        emit('options', processedOptions);
        isOpen.value = false;
    } else {
        error.value = 'Please enter at least one option.';
    }
}
</script>

<template>
    <div class="pro-button-container">
        <button class="pro-button" @click="isOpen = !isOpen">
            {{ isOpen ? 'Hide pro mode' : 'Show pro mode' }}
        </button>
    </div>

    <div class="pro-module" v-show="isOpen">
        <div class="pro-instructions">
            <p>Enter all options at once, separated by commas. This will replace all existing options!</p>
        </div>

        <div v-if="error" class="error">
            {{ error }}
        </div>

        <form @submit.prevent="formatOptions" class="pro-form">
            <textarea class="pro-text-input" rows="4" v-model="optionsInput"
                placeholder="Option 1, Option 2, Option 3"></textarea>
            <button class="add-button" type="submit">Format & Update</button>
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