<template>
  <div class="container">
    <h1>Translate App</h1>

    <div class="input-group">
      <textarea v-model="inputText" placeholder="Enter English text"></textarea>
      <select v-model="targetLang">
        <option value="hi">Hindi</option>
        <option value="fr">French</option>
        <option value="es">Spanish</option>
        <option value="de">German</option>
      </select>
      <button @click="translateText">Translate</button>
    </div>

    <div v-if="translatedText" class="result">
      <h3>Translated ({{ languageName[targetLang] }}):</h3>
      <p>{{ translatedText }}</p>
      <button @click="speakTranslation">🔊 Speak</button>
    </div>

    <div class="history" v-if="history.length > 0">
      <h3>Translation History</h3>
      <ul>
        <li v-for="(item, index) in history" :key="index">
          <strong>{{ item.source }}</strong> → <em>{{ item.target }}</em>
        </li>
      </ul>
      <button @click="clearHistory">Clear History</button>
    </div>

    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

export default defineComponent({
  setup() {
    const inputText = ref('')
    const translatedText = ref('')
    const targetLang = ref('hi')
    const error = ref('')
    const history = ref<{ source: string; target: string }[]>([])
    const languageName: Record<string, string> = {
      hi: 'Hindi',
      fr: 'French',
      es: 'Spanish',
      de: 'German'
    }

    const translateText = async () => {
      if (!inputText.value.trim()) return
      const encodedText = encodeURIComponent(inputText.value)
      const url = `https://api.mymemory.translated.net/get?q=${encodedText}&langpair=en|${targetLang.value}`

      try {
        const response = await fetch(url)
        const data = await response.json()
        translatedText.value = data.responseData.translatedText
        history.value.unshift({ source: inputText.value, target: translatedText.value })
        if (history.value.length > 10) history.value.pop()
        error.value = ''
      } catch (err) {
        error.value = 'Failed to fetch translation. Please try again.'
      }
    }

    const clearHistory = () => {
      history.value = []
    }

    const speakTranslation = () => {
      const msg = new SpeechSynthesisUtterance(translatedText.value)
      msg.lang = targetLang.value
      window.speechSynthesis.speak(msg)
    }

    return {
      inputText,
      translatedText,
      targetLang,
      error,
      history,
      translateText,
      clearHistory,
      speakTranslation,
      languageName
    }
  }
})
</script>

<style>
.container {
  max-width: 600px;
  margin: auto;
  padding: 2rem;
  font-family: Arial, sans-serif;
}
.input-group {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
textarea {
  width: 100%;
  height: 100px;
  padding: 0.5rem;
  font-size: 1rem;
}
select, button {
  padding: 0.5rem;
  font-size: 1rem;
}
.result, .history {
  margin-top: 2rem;
}
.error {
  color: red;
  margin-top: 1rem;
}
</style>
