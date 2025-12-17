<template>
  <div 
    class="color-card" 
    :style="{ backgroundColor: color.hex }"
    @click="copyColor"
    :class="{ 'pinned': color.pinned }"
  >
    <div class="color-info" :class="textColor">
      <span>{{ displayValue }}</span>
      <button @click.stop="togglePin" :class="textColor">
        {{ color.pinned ? '📌' : '📍' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  color: Object,
  displayFormat: { type: String, default: 'HEX' }
})

const emit = defineEmits(['toggle-pin', 'copy'])

const displayValue = computed(() => 
  props.displayFormat === 'RGB' ? props.color.rgb : props.color.hex
)

const textColor = computed(() => {
  const hex = props.color.hex.replace('#', '')
  const r = parseInt(hex.substr(0, 2), 16)
  const g = parseInt(hex.substr(2, 2), 16)
  const b = parseInt(hex.substr(4, 2), 16)
  return (r * 299 + g * 587 + b * 114) / 1000 > 128 ? 'dark' : 'light'
})

const copyColor = () => {
  const text = displayValue.value
  navigator.clipboard.writeText(text)
  emit('copy', text)
}

const togglePin = () => emit('toggle-pin', props.color.id)
</script>

<style scoped>
.color-card {
  flex: 1;
  min-height: 180px;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: flex-end;
  padding: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.color-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.color-card.pinned {
  border: 3px solid #333;
}

.color-info {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 8px;
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(10px);
  border-radius: 4px;
}

.dark { color: #333; }
.light { color: #fff; }

button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  padding: 4px 8px;
  border-radius: 4px;
}

button:hover {
  background: rgba(0,0,0,0.1);
}
</style>