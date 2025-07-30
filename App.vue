<template>
  <div id="app">
    <h1>Пример использования компонента формы</h1>
    
    <!-- Использование компонента формы с пропсами -->
    <UserForm
      ref="userFormRef"
      :api-url="formApiUrl"
      title="Свяжитесь с нами"
      name-placeholder="Как вас зовут?"
      email-placeholder="Ваш email для связи"
      message-placeholder="Расскажите нам о вашем вопросе"
      submit-button-text="Отправить сообщение"
      :required="true"
      :initial-data="initialFormData"
      @submit="onFormSubmit"
      @success="onFormSuccess"
      @error="onFormError"
    />

    <!-- Дополнительные кнопки для управления формой -->
    <div class="form-controls">
      <button @click="resetForm" class="control-button">
        Сбросить форму
      </button>
      <button @click="fillTestData" class="control-button">
        Заполнить тестовыми данными
      </button>
    </div>

    <!-- Лог событий -->
    <div v-if="eventLog.length" class="event-log">
      <h3>Лог событий:</h3>
      <div v-for="(event, index) in eventLog" :key="index" class="log-entry">
        <strong>{{ event.type }}:</strong> {{ event.message }}
        <small>({{ event.timestamp }})</small>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import UserForm from './UserForm.vue'

// Ссылка на компонент формы
const userFormRef = ref(null)

// Настройки формы
const formApiUrl = 'https://jsonplaceholder.typicode.com/posts'

// Начальные данные для формы
const initialFormData = reactive({
  name: '',
  email: '',
  message: ''
})

// Лог событий для демонстрации
const eventLog = ref([])

// Функция добавления события в лог
const addToLog = (type, message) => {
  eventLog.value.unshift({
    type,
    message,
    timestamp: new Date().toLocaleTimeString()
  })
  
  // Ограничиваем количество записей в логе
  if (eventLog.value.length > 10) {
    eventLog.value.pop()
  }
}

// Обработчики событий формы
const onFormSubmit = (formData) => {
  addToLog('SUBMIT', `Отправка данных: ${JSON.stringify(formData)}`)
  console.log('Форма отправляется:', formData)
}

const onFormSuccess = (result) => {
  addToLog('SUCCESS', `Форма успешно отправлена. ID: ${result.id}`)
  console.log('Успешная отправка:', result)
}

const onFormError = (error) => {
  addToLog('ERROR', `Ошибка: ${error.message}`)
  console.error('Ошибка отправки формы:', error)
}

// Функции управления формой
const resetForm = () => {
  if (userFormRef.value) {
    userFormRef.value.resetForm()
    addToLog('RESET', 'Форма сброшена')
  }
}

const fillTestData = () => {
  if (userFormRef.value) {
    userFormRef.value.formData.name = 'Иван Иванов'
    userFormRef.value.formData.email = 'ivan@example.com'
    userFormRef.value.formData.message = 'Это тестовое сообщение для проверки формы.'
    addToLog('FILL', 'Форма заполнена тестовыми данными')
  }
}
</script>

<style scoped>
#app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.form-controls {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin: 20px 0;
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
  padding: 8px;
  margin-bottom: 8px;
  background-color: white;
  border-radius: 4px;
  border-left: 4px solid #007bff;
}

.log-entry small {
  color: #6c757d;
  margin-left: 10px;
}
</style>