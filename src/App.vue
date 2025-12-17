<template>
  <div class="app" :class="{ 'dark-bg': isDarkBackground }">
    <header class="header">
      <h1>Генератор цветовых палитр</h1>
    </header>
    <main class="main-content">
      <div class="controls">
        <div class="control-group">
          <button class="btn generate-btn" @click="generatePalette">
             Случайная палитра
          </button>
          <div class="settings">
            <div class="setting">
              <label>Количество цветов:</label>
              <select v-model="colorCount" @change="updatePaletteCount">
                <option value="3">3</option>
                <option value="5" selected>5</option>
                <option value="7">7</option>
              </select>
            </div>
            <div class="setting">
              <label>Формат:</label>
              <div class="format-toggle">
                <button 
                  class="format-btn" 
                  :class="{ active: displayFormat === 'HEX' }"
                  @click="displayFormat = 'HEX'"
                >
                  HEX
                </button>
                <button 
                  class="format-btn"
                  :class="{ active: displayFormat === 'RGB' }"
                  @click="displayFormat = 'RGB'"
                >
                  RGB
                </button>
              </div>
            </div>
            
            <div class="setting">
              <label>Фон:</label>
              <button class="bg-toggle" @click="toggleBackground">
                {{ isDarkBackground ? '🌙 Тёмный' : '☀️ Светлый' }}
              </button>
            </div>
          </div>
        </div>
      </div>
      <div class="palette-container">
        <div class="palette">
          <ColorCard
            v-for="color in colors"
            :key="color.id"
            :color="color"
            :display-format="displayFormat"
            @toggle-pin="togglePin"
            @copy="handleCopyNotification"
          />
        </div>
      </div>
      <div class="preview-section">
        <h3>Превью интерфейса</h3>
        <div class="mockup" :class="{ 'dark': isDarkBackground }">
          <div class="mockup-header" :style="{ backgroundColor: colors[0]?.hex }">
            <h4 class="mockup-title" :style="{ color: getTextColor(colors[0]?.hex) }">
              Заголовок
            </h4>
          </div>
          <div class="mockup-content">
            <div class="mockup-card" :style="{ backgroundColor: colors[1]?.hex }">
              <p class="mockup-text" :style="{ color: getTextColor(colors[1]?.hex) }">
                Это пример карточки с контентом
              </p>
            </div>
            <button 
              class="mockup-button" 
              :style="{ 
                backgroundColor: colors[2]?.hex,
                color: getTextColor(colors[2]?.hex)
              }"
            >
              Кнопка действия
            </button>
          </div>
        </div>
      </div>
    </main>
    <div v-if="showNotification" class="notification">
      Цвет {{ notificationText }} скопирован!
    </div>
    <footer class="footer">
      <p>Футер</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import ColorCard from './components/ColorCard.vue'

const colors = ref([])
const colorCount = ref(5)
const displayFormat = ref('HEX')
const isDarkBackground = ref(false)
const showNotification = ref(false)
const notificationText = ref('')
const generateRandomHex = () => {
  return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')
}
const hexToRgb = (hex) => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
  return result ? 
    `rgb(${parseInt(result[1], 16)}, ${parseInt(result[2], 16)}, ${parseInt(result[3], 16)})` : 
    'rgb(0, 0, 0)'
}
const generateHarmoniousColors = (count) => {
  const baseHue = Math.floor(Math.random() * 360)
  const saturation = 60 + Math.random() * 30
  const lightness = 40 + Math.random() * 30
  const newColors = []
  for (let i = 0; i < count; i++) {
    const hue = (baseHue + (i * 360 / count)) % 360
    const hex = hslToHex(hue, saturation, lightness + (i % 2 === 0 ? -10 : 10))
    const rgb = hexToRgb(hex)
    const existingColor = colors.value[i]
    newColors.push({
      id: i,
      hex,
      rgb,
      pinned: existingColor?.pinned || false
    })
  }
  return newColors
}
const hslToHex = (h, s, l) => {
  h /= 360
  s /= 100
  l /= 100
  let r, g, b
  if (s === 0) {
    r = g = b = l
  } else {
    const hue2rgb = (p, q, t) => {
      if (t < 0) {
        t += 1;
      }
      if (t > 1) {
        t -= 1;
      }
      if (t < 1/6) {
        return p + (q - p) * 6 * t;
      }
      if (t < 1/2) {
        return q
      }
      if (t < 2/3) {
        return p + (q - p) * (2/3 - t) * 6;
      }
      return p
    }
    const q = l < 0.5 ? l * (1 + s) : l + s - l * s
    const p = 2 * l - q
    r = hue2rgb(p, q, h + 1/3)
    g = hue2rgb(p, q, h)
    b = hue2rgb(p, q, h - 1/3)
  }
  const toHex = x => {
    const hex = Math.round(x * 255).toString(16)
    return hex.length === 1 ? '0' + hex : hex
  }
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`
}

const generatePalette = () => {
  const newColors = generateHarmoniousColors(colorCount.value)
  colors.value.forEach((color, index) => {
    if (color.pinned && newColors[index]) {
      newColors[index] = { ...color }
    }
  })
  for (let i = 0; i < colorCount.value; i++) {
    if (!newColors[i]) {
      const hex = generateRandomHex()
      newColors[i] = {
        id: i,
        hex,
        rgb: hexToRgb(hex),
        pinned: false
      }
    }
  }
  colors.value = newColors
  saveToLocalStorage()
}

const togglePin = (colorId) => {
  colors.value = colors.value.map(color => 
    color.id === colorId ? { ...color, pinned: !color.pinned } : color
  )
  saveToLocalStorage()
}
const updatePaletteCount = () => {
  const newCount = parseInt(colorCount.value)
  if (newCount > colors.value.length) {
    for (let i = colors.value.length; i < newCount; i++) {
      const hex = generateRandomHex()
      colors.value.push({
        id: i,
        hex,
        rgb: hexToRgb(hex),
        pinned: false
      })
    }
  } else if (newCount < colors.value.length) {
    const pinnedColors = colors.value.filter(c => c.pinned)
    colors.value = colors.value.slice(0, newCount)
    pinnedColors.forEach(pinnedColor => {
      if (!colors.value.find(c => c.id === pinnedColor.id)) {
        if (colors.value.length < newCount) {
          colors.value.push(pinnedColor)
        } else {
          for (let i = colors.value.length - 1; i >= 0; i--) {
            if (!colors.value[i].pinned) {
              colors.value[i] = pinnedColor
              break
            }
          }
        }
      }
    })
  }
  saveToLocalStorage()
}

const toggleBackground = () => {
  isDarkBackground.value = !isDarkBackground.value
}
const handleCopyNotification = (colorValue) => {
  notificationText.value = colorValue
  showNotification.value = true
  setTimeout(() => {
    showNotification.value = false
  }, 2000)
}
const getTextColor = (backgroundColor) => {
  if (!backgroundColor) {
    return '#333';
  }
  const hex = backgroundColor.replace('#', '')
  const r = parseInt(hex.substr(0, 2), 16)
  const g = parseInt(hex.substr(2, 2), 16)
  const b = parseInt(hex.substr(4, 2), 16)
  const brightness = (r * 299 + g * 587 + b * 114) / 1000
  return brightness > 128 ? '#333' : '#fff'
}
const saveToLocalStorage = () => {
  const data = {
    colors: colors.value,
    colorCount: colorCount.value,
    displayFormat: displayFormat.value,
    isDarkBackground: isDarkBackground.value
  }
  localStorage.setItem('colorPalette', JSON.stringify(data))
}
const loadFromLocalStorage = () => {
  const saved = localStorage.getItem('colorPalette')
  if (saved) {
    const data = JSON.parse(saved)
    colors.value = data.colors || []
    colorCount.value = data.colorCount || 5
    displayFormat.value = data.displayFormat || 'HEX'
    isDarkBackground.value = data.isDarkBackground || false
  } else {
    generatePalette()
  }
}
onMounted(() => {
  loadFromLocalStorage()
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
  transition: background-color 0.3s;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;
  transition: background-color 0.3s;
}

.app.dark-bg {
  background-color: #1a1a1a;
  color: #fff;
}

.header {
  text-align: center;
  padding: 2rem 1rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.5rem;
}

.main-content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1rem;
  width: 100%;
}

.controls {
  margin-bottom: 2rem;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
}

.generate-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.settings {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.setting {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
}

.setting label {
  font-weight: 600;
  font-size: 0.9rem;
  opacity: 0.8;
}

.setting select {
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: 2px solid #ddd;
  background: white;
  cursor: pointer;
}

.app.dark-bg .setting select {
  background: #333;
  border-color: #555;
  color: white;
}

.format-toggle {
  display: flex;
  gap: 0.5rem;
}

.format-btn {
  padding: 0.5rem 1rem;
  border: 2px solid #ddd;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.format-btn.active {
  background: #667eea;
  color: white;
  border-color: #667eea;
}

.app.dark-bg .format-btn {
  background: #333;
  border-color: #555;
  color: white;
}

.bg-toggle {
  padding: 0.5rem 1rem;
  border: 2px solid #ddd;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.app.dark-bg .bg-toggle {
  background: #333;
  border-color: #555;
  color: white;
}

.palette-container {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.app.dark-bg .palette-container {
  background: #2d2d2d;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.palette {
  display: flex;
  gap: 1rem;
  min-height: 200px;
}

.preview-section {
  margin-top: 2rem;
}

.preview-section h3 {
  margin-bottom: 1rem;
  text-align: center;
}

.mockup {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  max-width: 600px;
  margin: 0 auto;
}

.app.dark-bg .mockup {
  background: #333;
}

.mockup-header {
  padding: 1.5rem;
  transition: background-color 0.3s;
}

.mockup-title {
  font-size: 1.5rem;
  font-weight: bold;
}

.mockup-content {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.mockup-card {
  padding: 1rem;
  border-radius: 8px;
  transition: background-color 0.3s;
}

.mockup-text {
  margin: 0;
}

.mockup-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-size: 1rem;
  transition: all 0.3s;
  align-self: flex-start;
}

.notification {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  background: #10b981;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.footer {
  text-align: center;
  padding: 1.5rem;
  background: #f0f0f0;
  margin-top: auto;
}

.app.dark-bg .footer {
  background: #2d2d2d;
}

.footer p {
  margin-bottom: 0.5rem;
  font-weight: 600;
}


@media (max-width: 768px) {
  .palette {
    flex-direction: column;
    min-height: auto;
  }
  
  .color-card {
    min-height: 120px;
  }
  
  .settings {
    flex-direction: column;
    gap: 1rem;
  }
  
  .header h1 {
    font-size: 1.8rem;
  }
}

@media (max-width: 480px) {
  .palette-container {
    padding: 1rem;
  }
  
  .color-info {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }
}
</style>