# Создание компонента формы в Vue 3

Это руководство показывает, как создать переиспользуемый компонент формы в Vue 3 с использованием разных подходов.

## Содержание

1. [Основные концепции](#основные-концепции)
2. [Composition API подход](#composition-api-подход)
3. [Options API подход](#options-api-подход)
4. [Использование компонентов](#использование-компонентов)
5. [Ключевые особенности](#ключевые-особенности)

## Основные концепции

### Пропсы (Props)
Пропсы позволяют передавать данные от родительского компонента к дочернему:

```javascript
// Определение пропсов в Composition API
const props = defineProps({
  title: {
    type: String,
    default: 'Форма обратной связи'
  },
  apiUrl: {
    type: String,
    required: true
  },
  required: {
    type: Boolean,
    default: true
  }
})
```

### События (Events)
События позволяют дочернему компоненту отправлять данные родительскому:

```javascript
// Определение событий
const emit = defineEmits(['submit', 'success', 'error'])

// Использование
emit('success', result)
```

### Реактивность
Vue 3 предоставляет несколько способов создания реактивных данных:

```javascript
import { ref, reactive } from 'vue'

// ref для примитивных значений
const isSubmitting = ref(false)

// reactive для объектов
const formData = reactive({
  name: '',
  email: '',
  message: ''
})
```

## Composition API подход

### Преимущества:
- Более логичная группировка связанного кода
- Лучшая поддержка TypeScript
- Более гибкое переиспользование логики
- Меньше магии, более явный код

### Структура компонента:

```vue
<script setup>
import { ref, reactive, defineProps, defineEmits } from 'vue'

// 1. Определение пропсов
const props = defineProps({
  // конфигурация пропсов
})

// 2. Определение событий
const emit = defineEmits(['submit', 'success', 'error'])

// 3. Реактивные данные
const formData = reactive({
  name: '',
  email: '',
  message: ''
})

// 4. Состояние компонента
const isSubmitting = ref(false)

// 5. Методы
const handleSubmit = async () => {
  // логика отправки
}

// 6. Экспорт для родительского компонента
defineExpose({
  resetForm,
  formData
})
</script>
```

## Options API подход

### Преимущества:
- Привычная структура для разработчиков Vue 2
- Четкое разделение на секции
- Простота понимания для начинающих

### Структура компонента:

```vue
<script>
export default {
  name: 'UserForm',
  
  // 1. Пропсы
  props: {
    title: {
      type: String,
      default: 'Форма'
    }
  },
  
  // 2. События
  emits: ['submit', 'success', 'error'],
  
  // 3. Данные
  data() {
    return {
      formData: {
        name: '',
        email: ''
      },
      isSubmitting: false
    }
  },
  
  // 4. Вычисляемые свойства
  computed: {
    isFormValid() {
      return this.formData.name && this.formData.email
    }
  },
  
  // 5. Наблюдатели
  watch: {
    'formData.email'() {
      this.validateEmail()
    }
  },
  
  // 6. Методы
  methods: {
    async handleSubmit() {
      // логика отправки
    }
  }
}
</script>
```

## Использование компонентов

### Базовое использование:

```vue
<template>
  <UserForm
    :api-url="'https://api.example.com/contact'"
    title="Свяжитесь с нами"
    @submit="onFormSubmit"
    @success="onFormSuccess"
    @error="onFormError"
  />
</template>

<script setup>
import UserForm from './UserForm.vue'

const onFormSubmit = (formData) => {
  console.log('Отправка формы:', formData)
}

const onFormSuccess = (result) => {
  console.log('Успешная отправка:', result)
}

const onFormError = (error) => {
  console.error('Ошибка:', error)
}
</script>
```

### Расширенное использование с настройками:

```vue
<template>
  <UserForm
    ref="formRef"
    :api-url="apiEndpoint"
    title="Форма обратной связи"
    name-placeholder="Ваше имя"
    email-placeholder="Email для связи"
    submit-button-text="Отправить сообщение"
    :required="true"
    :initial-data="initialFormData"
    @submit="handleFormSubmit"
    @success="handleFormSuccess"
    @error="handleFormError"
  />
</template>

<script setup>
import { ref, reactive } from 'vue'
import UserForm from './UserForm.vue'

const formRef = ref(null)
const apiEndpoint = 'https://api.example.com/contact'

const initialFormData = reactive({
  name: 'Иван',
  email: 'ivan@example.com'
})

// Программное управление формой
const resetForm = () => {
  formRef.value.resetForm()
}

const fillTestData = () => {
  formRef.value.formData.name = 'Тест'
  formRef.value.formData.email = 'test@example.com'
}
</script>
```

## Ключевые особенности

### 1. Валидация форм
- Клиентская валидация в реальном времени
- Отображение ошибок для каждого поля
- Блокировка отправки при невалидных данных

### 2. Состояние загрузки
- Индикатор отправки формы
- Блокировка повторной отправки
- Визуальная обратная связь пользователю

### 3. Обработка ошибок
- Перехват ошибок сети
- Отображение понятных сообщений
- Эмиссия событий для родительского компонента

### 4. Гибкая настройка
- Настройка через пропсы
- Поддержка начальных данных
- Кастомизация текстов и плейсхолдеров

### 5. Отправка на сервер
```javascript
const handleSubmit = async () => {
  try {
    const response = await fetch(props.apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(formData)
    })

    if (!response.ok) {
      throw new Error(`Ошибка сервера: ${response.status}`)
    }

    const result = await response.json()
    emit('success', result)
    
  } catch (error) {
    emit('error', error)
  }
}
```

### 6. Доступность (a11y)
- Правильные `label` для всех полей
- Использование `id` и `for` атрибутов
- Поддержка клавиатурной навигации
- Семантическая разметка

## Примеры API ответов

### Успешный ответ:
```json
{
  "id": 123,
  "status": "success",
  "message": "Сообщение отправлено"
}
```

### Ошибка валидации:
```json
{
  "status": "error",
  "message": "Ошибка валидации",
  "errors": {
    "email": "Некорректный email",
    "name": "Имя обязательно"
  }
}
```

## Стилизация

Компоненты используют scoped CSS для изоляции стилей:

```vue
<style scoped>
.user-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.form-input:focus {
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.submit-button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}
</style>
```

## Работа со скрытыми данными

### Основные подходы

Есть несколько способов передавать данные, которые не отображаются в форме, но отправляются на сервер:

#### 1. Через проп `hiddenData`
```vue
<UserForm
  :hidden-data="{
    category: 'support',
    priority: 'high',
    source: 'landing-page'
  }"
/>
```

#### 2. Через отдельные пропсы
```vue
<UserForm
  :user-id="12345"
  :session-id="sessionId"
  source="contact-form"
/>
```

#### 3. Автоматические метаданные
Компонент автоматически добавляет:
- `timestamp` - время отправки
- `source` - источник формы
- Техническую информацию о браузере

### Примеры скрытых данных

```javascript
const hiddenFormData = reactive({
  // Контекст пользователя
  userId: 12345,
  userRole: 'premium',
  
  // Контекст страницы
  currentPage: window.location.href,
  referrer: document.referrer,
  userAgent: navigator.userAgent,
  
  // Бизнес-логика
  category: 'support',
  priority: 'medium',
  customerSegment: 'vip',
  
  // Аналитика
  timeOnPage: 120,
  scrollDepth: 75,
  clickCount: 5
})
```

### Итоговая структура данных

При отправке формы объединяются:
```javascript
{
  // Видимые поля формы
  name: "Иван Иванов",
  email: "ivan@example.com",
  message: "Текст сообщения",
  
  // Скрытые данные
  userId: 12345,
  sessionId: "sess_abc123",
  category: "support",
  priority: "high",
  
  // Автоматические метаданные
  timestamp: "2024-12-01T10:30:00.000Z",
  source: "contact-form",
  userAgent: "Mozilla/5.0...",
  
  // Аналитические данные
  analytics: {
    timeOnPage: 120,
    scrollDepth: 75,
    interactionEvents: [...]
  }
}
```

### Composable для управления скрытыми данными

```javascript
function useHiddenFormData() {
  const userContext = reactive({
    userId: getCurrentUserId(),
    role: getUserRole(),
    subscriptionType: getSubscriptionType()
  })
  
  const pageContext = reactive({
    source: 'support-page',
    currentUrl: window.location.href,
    referrer: document.referrer,
    pageLoadTime: new Date().toISOString()
  })
  
  const getHiddenData = () => ({
    ...userContext,
    ...pageContext,
    timestamp: new Date().toISOString()
  })
  
  return { getHiddenData }
}
```

## Заключение

Создание переиспользуемых компонентов форм в Vue 3 позволяет:
- Сократить дублирование кода
- Обеспечить консистентность интерфейса
- Упростить поддержку и тестирование
- Повысить производительность разработки
- Собирать богатую аналитическую информацию
- Передавать контекстные данные без загромождения UI

Выбор между Composition API и Options API зависит от ваших предпочтений и требований проекта. Composition API лучше подходит для сложной логики и TypeScript, а Options API может быть проще для небольших компонентов.
