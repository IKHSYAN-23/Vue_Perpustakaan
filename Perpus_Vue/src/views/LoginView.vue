<template>
  <div class="app-container">
    <div class="form-card">
      <form v-if="!isLoggedIn" @submit.prevent="handleLogin">
        <h2>Login Perpustakaan</h2>

        <!-- Error message -->
        <div v-if="errorMsg" class="alert alert-error">
          ⚠️ {{ errorMsg }}
        </div>

        <!-- Loading message -->
        <div v-if="loading" class="alert alert-info">
          ⏳ Sedang mengidentifikasi akun Anda...
        </div>

        <!-- Email -->
        <div class="form-group">
          <label for="inputEmail">Email</label>
          <input
            type="email"
            id="inputEmail"
            v-model="email"
            placeholder="Masukkan Email"
            required
          />
        </div>

        <!-- Password -->
        <div class="form-group">
          <label for="inputPassword">Password</label>
          <input
            type="password"
            id="inputPassword"
            v-model="password"
            placeholder="Masukkan Password"
            required
          />
        </div>

        <!-- Button -->
        <button type="submit" :disabled="loading">
          🔐 Login
        </button>
      </form>

      <!-- Redirect spinner -->
      <div v-if="isLoggedIn" class="redirect">
        <div class="spinner"></div>
        <p>Mengarahkan ke halaman dashboard...</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token')
      window.location.reload()
    }
    return Promise.reject(error)
  }
)

export default {
  name: 'LoginView',
  data() {
    return {
      email: '',
      password: '',
      errorMsg: '',
      loading: false,
      isLoggedIn: false,
    }
  },
  methods: {
    async handleLogin() {
      this.errorMsg = ''
      this.loading = true

      try {
        localStorage.removeItem('token')
        delete axios.defaults.headers.common['Authorization']

        const response = await axios.post(
          'http://45.64.100.26:88/perpus-api/public/api/login',
          {
            email: this.email.trim(),
            password: this.password,
          },
          {
            validateStatus: status => status < 500
          }
        )

        if (response.status === 200 && response.data.token) {
          const token = response.data.token
          localStorage.setItem('token', token)
          axios.defaults.headers.common['Authorization'] = `Bearer ${token}`

          try {
            await axios.get('http://45.64.100.26:88/perpus-api/public/api/member')
            this.isLoggedIn = true
            this.$router.push('/dashboard')
          } catch {
            this.errorMsg = 'Gagal memverifikasi sesi. Silakan coba lagi.'
            localStorage.removeItem('token')
          }
        } else {
          if (response.data?.message) {
            this.errorMsg = response.data.message
          } else if (response.status === 401) {
            this.errorMsg = 'Email atau password salah'
          } else if (response.status === 422) {
            this.errorMsg = 'Format email tidak valid'
          } else {
            this.errorMsg = 'Terjadi kesalahan pada server. Silakan coba lagi.'
          }
        }
      } catch (err) {
        this.errorMsg = 'Gagal login. Silakan periksa koneksi atau coba lagi.'
      } finally {
        this.loading = false
      }
    },
    async verifyToken(token) {
      try {
        await axios.get(
          'http://45.64.100.26:88/perpus-api/public/api/member',
          {
            headers: {
              Authorization: `Bearer ${token}`
            }
          }
        )
      } catch {
        throw new Error('Token tidak valid')
      }
    }
  },
  async mounted() {
    const token = localStorage.getItem('token')
    if (token) {
      this.loading = true
      try {
        await this.verifyToken(token)
        axios.defaults.headers.common['Authorization'] = `Bearer ${token}`
        this.isLoggedIn = true
        this.$router.push('/members')
      } catch {
        localStorage.removeItem('token')
        delete axios.defaults.headers.common['Authorization']
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
/* Centering & Background */
.app-container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #e0f7fa, #80deea);
  padding: 24px;
}

/* Card Styling */
.form-card {
  background: #ffffff;
  padding: 32px;
  border-radius: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
}

/* Header */
h2 {
  font-size: 2rem;
  color: #0277bd;
  text-align: center;
  margin-bottom: 24px;
}

/* Alerts */
.alert {
  padding: 12px 16px;
  border-radius: 8px;
  font-weight: 500;
}
.alert-error {
  background-color: #ffebee;
  color: #c62828;
}
.alert-info {
  background-color: #e3f2fd;
  color: #1565c0;
}

/* Form Groups */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
label {
  font-weight: 600;
}

input {
  padding: 14px 16px;
  border: 1px solid #b0bec5;
  border-radius: 12px;
  font-size: 1rem;
  width: 100%;
  box-sizing: border-box;
  transition: border-color 0.3s;
}
input:focus {
  outline: none;
  border-color: #0288d1;
}

/* Button */
button {
  margin-top: 16px;
  padding: 14px;
  background: linear-gradient(90deg, #0288d1, #26c6da);
  color: white;
  border: none;
  border-radius: 12px;
  font-weight: bold;
  font-size: 1rem;
  cursor: pointer;
  transition: opacity 0.3s;
  width: 100%;
}
button:hover:not(:disabled) {
  opacity: 0.9;
}
button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Redirect Spinner */
.redirect {
  text-align: center;
}
.spinner {
  width: 48px;
  height: 48px;
  border: 5px solid #ddd;
  border-top-color: #0288d1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 12px;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>