<template>
  <form @submit.prevent="handleSubmit" class="user-form">
    <h2>{{ title }}</h2>
    
    <!-- Поле имени -->
    <div class="form-group">
      <label for="name">Имя:</label>
      <input
        id="name"
        v-model="formData.name"
        type="text"
        :placeholder="namePlaceholder"
        :required="required"
        class="form-input"
      />
    </div>

    <!-- Поле email -->
    <div class="form-group">
      <label for="email">Email:</label>
      <input
        id="email"
        v-model="formData.email"
        type="email"
        :placeholder="emailPlaceholder"
        :required="required"
        class="form-input"
      />
    </div>

    <!-- Поле сообщения -->
    <div class="form-group">
      <label for="message">Сообщение:</label>
      <textarea
        id="message"
        v-model="formData.message"
        :placeholder="messagePlaceholder"
        :required="required"
        class="form-textarea"
        rows="4"
      ></textarea>
    </div>

    <!-- Кнопка отправки -->
    <button 
      type="submit" 
      :disabled="isSubmitting"
      class="submit-button"
    >
      {{ isSubmitting ? 'Отправка...' : submitButtonText }}
    </button>

    <!-- Сообщения об ошибках или успехе -->
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>

    <!-- Отладочная информация (можно убрать в продакшене) -->
    <div v-if="showDebugInfo" class="debug-info">
      <h4>Данные для отправки:</h4>
      <pre>{{ JSON.stringify(getSubmitData(), null, 2) }}</pre>
    </div>
  </form>
</template>

<script setup>
import { ref, reactive, computed, defineProps, defineEmits } from 'vue'

// Определяем пропсы
const props = defineProps({
  title: {
    type: String,
    default: 'Форма обратной связи'
  },
  apiUrl: {
    type: String,
    required: true
  },
  namePlaceholder: {
    type: String,
    default: 'Введите ваше имя'
  },
  emailPlaceholder: {
    type: String,
    default: 'Введите ваш email'
  },
  messagePlaceholder: {
    type: String,
    default: 'Введите ваше сообщение'
  },
  submitButtonText: {
    type: String,
    default: 'Отправить'
  },
  required: {
    type: Boolean,
    default: true
  },
  initialData: {
    type: Object,
    default: () => ({})
  },
  
  // СКРЫТЫЕ ДАННЫЕ - не отображаются в форме
  hiddenData: {
    type: Object,
    default: () => ({})
  },
  
  // Дополнительные метаданные
  userId: {
    type: [String, Number],
    default: null
  },
  sessionId: {
    type: String,
    default: null
  },
  source: {
    type: String,
    default: 'contact-form'
  },
  
  // Настройки отображения
  showDebugInfo: {
    type: Boolean,
    default: false
  }
})

// Определяем события
const emit = defineEmits(['submit', 'success', 'error'])

// Реактивные данные формы (только видимые поля)
const formData = reactive({
  name: props.initialData.name || '',
  email: props.initialData.email || '',
  message: props.initialData.message || ''
})

// Состояние формы
const isSubmitting = ref(false)
const errorMessage = ref('')
const successMessage = ref('')

// Вычисляемое свойство для объединения всех данных
const getSubmitData = () => {
  const submitData = {
    // Основные данные формы
    ...formData,
    
    // Скрытые данные из пропса
    ...props.hiddenData,
    
    // Метаданные
    timestamp: new Date().toISOString(),
    source: props.source
  }
  
  // Добавляем userId если передан
  if (props.userId) {
    submitData.userId = props.userId
  }
  
  // Добавляем sessionId если передан
  if (props.sessionId) {
    submitData.sessionId = props.sessionId
  }
  
  return submitData
}

// Функция отправки формы
const handleSubmit = async () => {
  // Очищаем предыдущие сообщения
  errorMessage.value = ''
  successMessage.value = ''
  isSubmitting.value = true

  try {
    // Валидация на клиенте
    if (props.required && (!formData.name || !formData.email || !formData.message)) {
      throw new Error('Все поля должны быть заполнены')
    }

    // Получаем полные данные для отправки
    const submitData = getSubmitData()
    
    // Эмитируем событие перед отправкой
    emit('submit', submitData)

    // Отправляем данные на сервер
    const response = await fetch(props.apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(submitData)
    })

    if (!response.ok) {
      throw new Error(`Ошибка сервера: ${response.status}`)
    }

    const result = await response.json()
    
    // Успешная отправка
    successMessage.value = 'Форма успешно отправлена!'
    
    // Эмитируем событие успеха
    emit('success', result)
    
    // Очищаем форму
    resetForm()

  } catch (error) {
    errorMessage.value = error.message || 'Произошла ошибка при отправке формы'
    
    // Эмитируем событие ошибки
    emit('error', error)
  } finally {
    isSubmitting.value = false
  }
}

// Функция сброса формы
const resetForm = () => {
  formData.name = props.initialData.name || ''
  formData.email = props.initialData.email || ''
  formData.message = props.initialData.message || ''
}

// Экспортируем методы для родительского компонента
defineExpose({
  resetForm,
  formData,
  getSubmitData
})
</script>

<style scoped>
.user-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.form-input,
.form-textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  box-sizing: border-box;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.submit-button {
  width: 100%;
  padding: 12px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.submit-button:hover:not(:disabled) {
  background-color: #0056b3;
}

.submit-button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

.error-message {
  margin-top: 10px;
  padding: 10px;
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
}

.success-message {
  margin-top: 10px;
  padding: 10px;
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
  border-radius: 4px;
}

.debug-info {
  margin-top: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  font-size: 12px;
}

.debug-info h4 {
  margin-top: 0;
  margin-bottom: 10px;
  color: #495057;
}

.debug-info pre {
  margin: 0;
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>