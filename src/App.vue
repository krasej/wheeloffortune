<script setup lang="ts">
import { ref, watch } from 'vue'
import SpinnerModule from './components/SpinnerModule.vue';
import ProModule from './components/ProModule.vue'

const winner = ref<string | undefined>('')
const newPrize = ref('')
const titleElement = ref<HTMLElement | null>(null);


let title = ref('Who is the best director?')

function validate(event: Event) {
  (event.target as HTMLInputElement).blur()

  if (!titleElement.value?.innerText) return

  wasEdited.value = true
  title.value = titleElement.value.innerText.trim()
}

defineExpose({ titleElement })

const wasEdited = ref(false)

const watchTitle = watch(title, (newValue) => {

  document.title = newValue
}
  , { immediate: true })

const segments = ref([
  'Quentin Tarantino',
  'Christopher Nolan',
])


function addOption() {

  let newPriceFormatted = newPrize.value.trim()

  if( newPriceFormatted === '') return

  else if (newPriceFormatted.length >= 25) {
    newPriceFormatted = newPriceFormatted.substring(0, 25) + '...'
  }

  segments.value.push(newPriceFormatted)
  newPrize.value = ''
}

function removeOption(index: number) {
  Array.prototype.splice.call(segments.value, index, 1)
}


</script>

<template>
  <div class="wheel-container">
    <h1 class="title-heading" :class="{ 'notEdited': !wasEdited }" ref="titleElement" spellcheck="false"
      contenteditable="true" @blur="validate" @keydown.enter="validate">{{ title }}</h1>

      <SpinnerModule :segments="segments" @update:winner="winner = $event" />
  </div>

  <div class="manage-options">
    <h2>Manage Options</h2>
    <div class="add-prize">
      <input @keydown.enter="addOption" v-model="newPrize" placeholder="Enter new option" class="prize-input" />
      <button @click="addOption" class="add-button">Add Option</button>
    </div>
    <h3>Current Options:</h3>
    <ul v-if="segments.length > 0" class="prize-list">
      <li v-for="(prize, index) in segments" :key="index">
        {{ prize }}
        <button @click="removeOption(index)" class="remove-button">&times;</button>
      </li>
    </ul>

    <ProModule :options="segments" @options="(options) => { segments = options }" />

  </div>
</template>
