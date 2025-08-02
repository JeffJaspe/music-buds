<script setup>
import { ref } from 'vue'

// define props before using any logic
defineProps({
  msg: String,
})
1
const count = ref(0)
const color = ref('#ffffff')
const isPopping = ref(false)

function handleClick() {
  count.value++
  color.value = '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')
  isPopping.value = false
  requestAnimationFrame(() => {
    isPopping.value = true
  })
}
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button
      type="button"
      @click="handleClick"
      :style="{ backgroundColor: count === 0 ? '#808080' : color }"
      :class="{ pop: isPopping }"
      @animationend="isPopping = false"
    >
      count is {{ count }}
    </button>

    <p>
      Edit <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank">create-vue</a>,
    the official Vue + Vite starter
  </p>

  <p>
    Install
    <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>

  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #eeeef0;
}

button {
  transition: transform 0.2s ease;
  padding: 1rem 2rem;
  border: none;
  border-radius: 8px;
  font-size: 1.2rem;
  cursor: pointer;
  color: white;
}

.pop {
  animation: popAnim 0.3s ease;
}

@keyframes popAnim {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}
</style>
