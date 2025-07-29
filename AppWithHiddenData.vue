<template>
  <div id="app">
    <h1>Пример формы со скрытыми данными</h1>
    
    <!-- Форма с различными типами скрытых данных -->
    <UserFormWithHiddenData
      ref="formRef"
      :api-url="apiUrl"
      title="Форма обратной связи"
      
      <!-- Скрытые данные через объект -->
      :hidden-data="hiddenFormData"
      
      <!-- Метаданные как отдельные пропсы -->
      :user-id="currentUser.id"
      :session-id="sessionId"
      source="landing-page"
      
      <!-- Отладочная информация -->
      :show-debug-info="showDebug"
      
      @submit="onFormSubmit"
      @success="onFormSuccess"
      @error="onFormError"
    />

    <!-- Управление -->
    <div class="controls">
      <h3>Управление скрытыми данными:</h3>
      
      <div class="control-group">
        <label>
          <input type="checkbox" v-model="showDebug" />
          Показать отладочную информацию
        </label>
      </div>

      <div class="control-group">
        <label>Категория обращения:</label>
        <select v-model="hiddenFormData.category">
          <option value="support">Техподдержка</option>
          <option value="sales">Продажи</option>
          <option value="feedback">Обратная связь</option>
        </select>
      </div>

      <div class="control-group">
        <label>Приоритет:</label>
        <select v-model="hiddenFormData.priority">
          <option value="low">Низкий</option>
          <option value="medium">Средний</option>
          <option value="high">Высокий</option>
        </select>
      </div>

      <div class="control-group">
        <label>Текущая страница:</label>
        <input v-model="hiddenFormData.currentPage" type="text" />
      </div>

      <div class="control-group">
        <label>Реферер:</label>
        <input v-model="hiddenFormData.referrer" type="text" />
      </div>

      <button @click="updateUserContext" class="control-button">
        Обновить контекст пользователя
      </button>
    </div>

    <!-- Информация о текущем состоянии -->
    <div class="current-state">
      <h3>Текущие скрытые данные:</h3>
      <pre>{{ JSON.stringify(getAllHiddenData(), null, 2) }}</pre>
    </div>

    <!-- Лог событий -->
    <div v-if="eventLog.length" class="event-log">
      <h3>Лог событий:</h3>
      <div v-for="(event, index) in eventLog" :key="index" class="log-entry">
        <strong>{{ event.type }}:</strong>
        <pre>{{ event.data }}</pre>
        <small>({{ event.timestamp }})</small>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import UserFormWithHiddenData from './UserFormWithHiddenData.vue'

// Ссылка на компонент формы
const formRef = ref(null)

// API настройки
const apiUrl = 'https://jsonplaceholder.typicode.com/posts'

// Текущий пользователь (обычно из store/auth)
const currentUser = reactive({
  id: 12345,
  role: 'customer',
  subscriptionType: 'premium'
})

// ID сессии (обычно генерируется при входе)
const sessionId = ref(generateSessionId())

// Настройки отображения
const showDebug = ref(true)

// Скрытые данные формы
const hiddenFormData = reactive({
  // Контекст обращения
  category: 'support',
  priority: 'medium',
  
  // Информация о странице
  currentPage: window.location.href,
  referrer: document.referrer || 'direct',
  userAgent: navigator.userAgent,
  
  // Временные метки
  pageLoadTime: new Date().toISOString(),
  
  // Дополнительная информация
  language: navigator.language,
  timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
  screenResolution: `${screen.width}x${screen.height}`,
  
  // Бизнес-логика
  customerSegment: 'vip',
  marketingCampaign: 'summer-2024',
  
  // Техническая информация
  version: '1.2.3',
  buildNumber: '20241201'
})

// Лог событий
const eventLog = ref([])

// Вычисляемое свойство для всех скрытых данных
const getAllHiddenData = () => {
  return {
    // Данные из hiddenData пропса
    ...hiddenFormData,
    
    // Метаданные из отдельных пропсов
    userId: currentUser.id,
    sessionId: sessionId.value,
    source: 'landing-page',
    
    // Автоматически добавляемые данные
    timestamp: 'будет добавлен при отправке',
    
    // Контекст пользователя
    userRole: currentUser.role,
    subscriptionType: currentUser.subscriptionType
  }
}

// Функции для работы с событиями
const addToLog = (type, data) => {
  eventLog.value.unshift({
    type,
    data: typeof data === 'string' ? data : JSON.stringify(data, null, 2),
    timestamp: new Date().toLocaleTimeString()
  })
  
  if (eventLog.value.length > 5) {
    eventLog.value.pop()
  }
}

// Обработчики событий формы
const onFormSubmit = (formData) => {
  addToLog('SUBMIT', formData)
  console.log('Отправка формы со всеми данными:', formData)
}

const onFormSuccess = (result) => {
  addToLog('SUCCESS', `Форма отправлена. ID: ${result.id}`)
  console.log('Успешная отправка:', result)
}

const onFormError = (error) => {
  addToLog('ERROR', error.message)
  console.error('Ошибка отправки формы:', error)
}

// Утилиты
function generateSessionId() {
  return 'sess_' + Math.random().toString(36).substr(2, 9)
}

// Обновление контекста пользователя
const updateUserContext = () => {
  // Симуляция обновления данных пользователя
  currentUser.role = currentUser.role === 'customer' ? 'vip' : 'customer'
  sessionId.value = generateSessionId()
  
  // Обновляем некоторые скрытые данные
  hiddenFormData.pageLoadTime = new Date().toISOString()
  hiddenFormData.customerSegment = currentUser.role === 'vip' ? 'premium' : 'standard'
  
  addToLog('UPDATE', 'Контекст пользователя обновлен')
}

// Инициализация при монтировании
onMounted(() => {
  // Здесь можно загрузить дополнительные данные
  console.log('Компонент формы инициализирован')
  
  // Пример: получение данных из localStorage
  const savedCategory = localStorage.getItem('lastFormCategory')
  if (savedCategory) {
    hiddenFormData.category = savedCategory
  }
})
</script>

<style scoped>
#app {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.controls {
  margin: 30px 0;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #dee2e6;
}

.controls h3 {
  margin-top: 0;
  color: #495057;
}

.control-group {
  margin-bottom: 15px;
}

.control-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.control-group input,
.control-group select {
  width: 100%;
  max-width: 300px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.control-group input[type="checkbox"] {
  width: auto;
  margin-right: 8px;
}

.control-button {
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.2s;
}

.control-button:hover {
  background-color: #218838;
}

.current-state {
  margin: 30px 0;
  padding: 20px;
  background-color: #e9ecef;
  border-radius: 8px;
  border: 1px solid #adb5bd;
}

.current-state h3 {
  margin-top: 0;
  color: #495057;
}

.current-state pre {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  overflow-x: auto;
  font-size: 12px;
}

.event-log {
  margin-top: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #dee2e6;
}

.event-log h3 {
  margin-top: 0;
  color: #495057;
}

.log-entry {
  padding: 10px;
  margin-bottom: 10px;
  background-color: white;
  border-radius: 4px;
  border-left: 4px solid #007bff;
}

.log-entry pre {
  margin: 5px 0;
  font-size: 11px;
  background-color: #f8f9fa;
  padding: 8px;
  border-radius: 3px;
  overflow-x: auto;
}

.log-entry small {
  color: #6c757d;
}
</style>