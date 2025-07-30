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
      <span v-if="errors.name" class="field-error">{{ errors.name }}</span>
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
      <span v-if="errors.email" class="field-error">{{ errors.email }}</span>
    </div>

    <!-- Поле телефона -->
    <div class="form-group">
      <label for="phone">Телефон:</label>
      <input
        id="phone"
        v-model="formData.phone"
        type="tel"
        :placeholder="phonePlaceholder"
        class="form-input"
      />
      <span v-if="errors.phone" class="field-error">{{ errors.phone }}</span>
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
      <span v-if="errors.message" class="field-error">{{ errors.message }}</span>
    </div>

    <!-- Чекбокс согласия -->
    <div class="form-group">
      <label class="checkbox-label">
        <input
          v-model="formData.agreement"
          type="checkbox"
          :required="required"
        />
        Я согласен с условиями обработки персональных данных
      </label>
      <span v-if="errors.agreement" class="field-error">{{ errors.agreement }}</span>
    </div>

    <!-- Кнопка отправки -->
    <button 
      type="submit" 
      :disabled="isSubmitting || !isFormValid"
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
  </form>
</template>

<script>
export default {
  name: 'UserFormOptionsAPI',
  
  props: {
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
    phonePlaceholder: {
      type: String,
      default: 'Введите ваш телефон'
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
    validationRules: {
      type: Object,
      default: () => ({})
    }
  },

  emits: ['submit', 'success', 'error', 'validation-error'],

  data() {
    return {
      formData: {
        name: this.initialData.name || '',
        email: this.initialData.email || '',
        phone: this.initialData.phone || '',
        message: this.initialData.message || '',
        agreement: this.initialData.agreement || false
      },
      errors: {},
      isSubmitting: false,
      errorMessage: '',
      successMessage: ''
    }
  },

  computed: {
    isFormValid() {
      if (!this.required) return true
      
      return this.formData.name.trim() && 
             this.formData.email.trim() && 
             this.formData.message.trim() && 
             this.formData.agreement &&
             Object.keys(this.errors).length === 0
    }
  },

  watch: {
    // Валидация в реальном времени
    'formData.name'() {
      this.validateField('name')
    },
    'formData.email'() {
      this.validateField('email')
    },
    'formData.phone'() {
      this.validateField('phone')
    },
    'formData.message'() {
      this.validateField('message')
    }
  },

  methods: {
    validateField(fieldName) {
      const value = this.formData[fieldName]
      
      // Очищаем предыдущую ошибку
      this.$delete(this.errors, fieldName)
      
      // Проверяем обязательные поля
      if (this.required && ['name', 'email', 'message'].includes(fieldName)) {
        if (!value || !value.trim()) {
          this.$set(this.errors, fieldName, 'Это поле обязательно для заполнения')
          return false
        }
      }
      
      // Специфичная валидация для email
      if (fieldName === 'email' && value) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailRegex.test(value)) {
          this.$set(this.errors, fieldName, 'Введите корректный email адрес')
          return false
        }
      }
      
      // Специфичная валидация для телефона
      if (fieldName === 'phone' && value) {
        const phoneRegex = /^[\+]?[0-9\s\-\(\)]{10,}$/
        if (!phoneRegex.test(value)) {
          this.$set(this.errors, fieldName, 'Введите корректный номер телефона')
          return false
        }
      }
      
      // Пользовательские правила валидации
      if (this.validationRules[fieldName]) {
        const rule = this.validationRules[fieldName]
        if (typeof rule === 'function') {
          const result = rule(value)
          if (result !== true) {
            this.$set(this.errors, fieldName, result)
            return false
          }
        }
      }
      
      return true
    },

    validateForm() {
      const fields = ['name', 'email', 'phone', 'message']
      let isValid = true
      
      fields.forEach(field => {
        if (!this.validateField(field)) {
          isValid = false
        }
      })
      
      // Проверяем согласие
      if (this.required && !this.formData.agreement) {
        this.$set(this.errors, 'agreement', 'Необходимо согласие на обработку данных')
        isValid = false
      }
      
      return isValid
    },

    async handleSubmit() {
      // Очищаем предыдущие сообщения
      this.errorMessage = ''
      this.successMessage = ''
      
      // Валидируем форму
      if (!this.validateForm()) {
        this.$emit('validation-error', this.errors)
        return
      }
      
      this.isSubmitting = true

      try {
        // Эмитируем событие перед отправкой
        this.$emit('submit', { ...this.formData })

        // Отправляем данные на сервер
        const response = await this.submitToServer()
        
        // Успешная отправка
        this.successMessage = 'Форма успешно отправлена!'
        this.$emit('success', response)
        
        // Очищаем форму
        this.resetForm()

      } catch (error) {
        this.errorMessage = error.message || 'Произошла ошибка при отправке формы'
        this.$emit('error', error)
      } finally {
        this.isSubmitting = false
      }
    },

    async submitToServer() {
      const response = await fetch(this.apiUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(this.formData)
      })

      if (!response.ok) {
        throw new Error(`Ошибка сервера: ${response.status}`)
      }

      return await response.json()
    },

    resetForm() {
      this.formData = {
        name: this.initialData.name || '',
        email: this.initialData.email || '',
        phone: this.initialData.phone || '',
        message: this.initialData.message || '',
        agreement: this.initialData.agreement || false
      }
      this.errors = {}
      this.errorMessage = ''
      this.successMessage = ''
    },

    // Метод для программного заполнения формы
    fillForm(data) {
      Object.assign(this.formData, data)
    }
  }
}
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

.checkbox-label {
  display: flex;
  align-items: center;
  font-weight: normal;
}

.checkbox-label input[type="checkbox"] {
  margin-right: 8px;
}

.form-input,
.form-textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.form-input.error,
.form-textarea.error {
  border-color: #dc3545;
}

.field-error {
  color: #dc3545;
  font-size: 12px;
  margin-top: 4px;
  display: block;
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
</style>