<template>
  <div class="advanced-form-example">
    <h2>Продвинутый пример со скрытыми данными</h2>
    
    <!-- Компонент формы -->
    <UserFormWithHiddenData
      :api-url="apiUrl"
      title="Форма поддержки"
      :hidden-data="hiddenData"
      :user-id="userContext.userId"
      :session-id="userContext.sessionId"
      :source="pageContext.source"
      :show-debug-info="true"
      @submit="handleFormSubmit"
      @success="handleFormSuccess"
      @error="handleFormError"
    />
    
    <!-- Информация о контексте -->
    <div class="context-info">
      <h3>Текущий контекст:</h3>
      <div class="context-section">
        <h4>Пользователь:</h4>
        <pre>{{ JSON.stringify(userContext, null, 2) }}</pre>
      </div>
      <div class="context-section">
        <h4>Страница:</h4>
        <pre>{{ JSON.stringify(pageContext, null, 2) }}</pre>
      </div>
      <div class="context-section">
        <h4>Аналитика:</h4>
        <pre>{{ JSON.stringify(analyticsData, null, 2) }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import UserFormWithHiddenData from './UserFormWithHiddenData.vue'

// Composable для управления скрытыми данными
function useHiddenFormData() {
  // Контекст пользователя
  const userContext = reactive({
    userId: 12345,
    role: 'premium_user',
    subscriptionType: 'pro',
    registrationDate: '2023-01-15',
    lastLoginDate: new Date().toISOString(),
    preferredLanguage: 'ru',
    timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
  })

  // Контекст страницы
  const pageContext = reactive({
    source: 'support-page',
    currentUrl: window.location.href,
    referrer: document.referrer || 'direct',
    userAgent: navigator.userAgent,
    pageLoadTime: new Date().toISOString(),
    viewportSize: `${window.innerWidth}x${window.innerHeight}`,
    screenSize: `${screen.width}x${screen.height}`,
    colorDepth: screen.colorDepth,
    pixelRatio: window.devicePixelRatio
  })

  // Аналитические данные
  const analyticsData = reactive({
    sessionDuration: 0,
    pageViews: 1,
    clickCount: 0,
    scrollDepth: 0,
    timeOnPage: 0,
    interactionEvents: []
  })

  // Таймер для отслеживания времени на странице
  const startTime = Date.now()
  const timeTracker = ref(null)

  // Отслеживание взаимодействий
  const trackInteraction = (type, details = {}) => {
    analyticsData.interactionEvents.push({
      type,
      timestamp: new Date().toISOString(),
      details
    })
    
    // Ограничиваем количество событий
    if (analyticsData.interactionEvents.length > 10) {
      analyticsData.interactionEvents.shift()
    }
  }

  // Обработчики событий для аналитики
  const handleClick = (event) => {
    analyticsData.clickCount++
    trackInteraction('click', {
      target: event.target.tagName,
      x: event.clientX,
      y: event.clientY
    })
  }

  const handleScroll = () => {
    const scrollTop = window.pageYOffset || document.documentElement.scrollTop
    const documentHeight = document.documentElement.scrollHeight - window.innerHeight
    const scrollPercent = Math.round((scrollTop / documentHeight) * 100)
    
    if (scrollPercent > analyticsData.scrollDepth) {
      analyticsData.scrollDepth = scrollPercent
      trackInteraction('scroll', { depth: scrollPercent })
    }
  }

  // Инициализация отслеживания
  const initTracking = () => {
    // Отслеживание времени на странице
    timeTracker.value = setInterval(() => {
      analyticsData.timeOnPage = Math.round((Date.now() - startTime) / 1000)
      analyticsData.sessionDuration = analyticsData.timeOnPage
    }, 1000)

    // Добавляем обработчики событий
    document.addEventListener('click', handleClick)
    window.addEventListener('scroll', handleScroll)
    
    // Отслеживание видимости страницы
    document.addEventListener('visibilitychange', () => {
      trackInteraction('visibility_change', {
        hidden: document.hidden
      })
    })
  }

  // Очистка ресурсов
  const cleanup = () => {
    if (timeTracker.value) {
      clearInterval(timeTracker.value)
    }
    document.removeEventListener('click', handleClick)
    window.removeEventListener('scroll', handleScroll)
  }

  // Получение всех скрытых данных
  const getHiddenData = () => {
    return {
      // Контекст пользователя
      ...userContext,
      
      // Контекст страницы
      ...pageContext,
      
      // Аналитические данные
      analytics: {
        ...analyticsData,
        // Добавляем финальные метрики
        finalTimeOnPage: Math.round((Date.now() - startTime) / 1000),
        submissionTime: new Date().toISOString()
      },
      
      // Техническая информация
      technical: {
        browserMemory: navigator.deviceMemory || 'unknown',
        connectionType: navigator.connection?.effectiveType || 'unknown',
        cookiesEnabled: navigator.cookieEnabled,
        doNotTrack: navigator.doNotTrack,
        onlineStatus: navigator.onLine
      },
      
      // Дополнительные метаданные
      metadata: {
        formVersion: '2.1.0',
        clientVersion: '1.0.0',
        apiVersion: 'v1',
        environment: process.env.NODE_ENV || 'development'
      }
    }
  }

  return {
    userContext,
    pageContext,
    analyticsData,
    initTracking,
    cleanup,
    getHiddenData,
    trackInteraction
  }
}

// Использование composable
const {
  userContext,
  pageContext,
  analyticsData,
  initTracking,
  cleanup,
  getHiddenData,
  trackInteraction
} = useHiddenFormData()

// API URL
const apiUrl = 'https://jsonplaceholder.typicode.com/posts'

// Вычисляемые скрытые данные
const hiddenData = ref({})

// Обновление скрытых данных
const updateHiddenData = () => {
  hiddenData.value = getHiddenData()
}

// Обработчики событий формы
const handleFormSubmit = (formData) => {
  // Обновляем скрытые данные перед отправкой
  updateHiddenData()
  
  trackInteraction('form_submit', {
    fieldsCount: Object.keys(formData).length,
    hasName: !!formData.name,
    hasEmail: !!formData.email,
    hasMessage: !!formData.message
  })
  
  console.log('Отправка формы:', formData)
}

const handleFormSuccess = (result) => {
  trackInteraction('form_success', {
    responseId: result.id,
    responseTime: new Date().toISOString()
  })
  
  console.log('Успешная отправка:', result)
}

const handleFormError = (error) => {
  trackInteraction('form_error', {
    errorMessage: error.message,
    errorTime: new Date().toISOString()
  })
  
  console.error('Ошибка отправки:', error)
}

// Lifecycle hooks
onMounted(() => {
  initTracking()
  updateHiddenData()
  
  // Периодическое обновление скрытых данных
  setInterval(updateHiddenData, 5000)
})

onUnmounted(() => {
  cleanup()
})
</script>

<style scoped>
.advanced-form-example {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.context-info {
  margin-top: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #dee2e6;
}

.context-info h3 {
  margin-top: 0;
  color: #495057;
  border-bottom: 2px solid #007bff;
  padding-bottom: 10px;
}

.context-section {
  margin-bottom: 20px;
}

.context-section h4 {
  color: #6c757d;
  margin-bottom: 10px;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.context-section pre {
  background-color: #ffffff;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  overflow-x: auto;
  font-size: 11px;
  line-height: 1.4;
  max-height: 200px;
  overflow-y: auto;
}
</style>