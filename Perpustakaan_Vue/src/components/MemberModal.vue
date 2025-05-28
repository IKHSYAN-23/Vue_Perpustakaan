<template>
  <div class="modal-overlay">
    <div class="modal-card card">
      <div class="card-header">
        <h2 class="mb-0">{{ mode === 'detail' ? 'Detail Member' : 'Edit Member' }}</h2>
        <button class="close-btn" @click="$emit('close')">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>

      <div class="card-body">
        <div v-if="mode === 'detail'" class="detail-container">
          <div class="form-row">
            <div class="form-group col-md-6">
              <label class="detail-label"><i class="fas fa-user"></i> Nama:</label>
              <span class="detail-value">{{ member.nama }}</span>
            </div>
            <div class="form-group col-md-6">
              <label class="detail-label"><i class="fas fa-id-card"></i> No KTP:</label>
              <span class="detail-value">{{ member.no_ktp }}</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group col-md-6">
              <label class="detail-label"><i class="fas fa-calendar-alt"></i> Tanggal Lahir:</label>
              <span class="detail-value">{{ formatDate(member.tgl_lahir) }}</span>
            </div>
            <div class="form-group col-md-6">
              <label class="detail-label"><i class="fas fa-map-marker-alt"></i> Alamat:</label>
              <span class="detail-value">{{ member.alamat }}</span>
            </div>
          </div>
        </div>

        <form v-else @submit.prevent="updateMember" class="edit-form">
          <div class="form-row">
            <div class="form-group col-md-6">
              <label class="input-label"><i class="fas fa-user"></i> Nama</label>
              <input v-model="form.nama" class="form-control" required />
            </div>
            <div class="form-group col-md-6">
              <label class="input-label"><i class="fas fa-id-card"></i> No KTP</label>
              <input v-model="form.no_ktp" class="form-control" required />
            </div>
          </div>
          <div class="form-row">
            <div class="form-group col-md-6">
              <label class="input-label"><i class="fas fa-calendar-alt"></i> Tanggal Lahir</label>
              <input v-model="form.tgl_lahir" type="date" class="form-control" required />
            </div>
            <div class="form-group col-md-6">
              <label class="input-label"><i class="fas fa-map-marker-alt"></i> Alamat</label>
              <textarea v-model="form.alamat" class="form-control" required rows="3"></textarea>
            </div>
          </div>
          <div class="form-actions">
            <button type="submit" class="btn btn-primary">
              <i class="fas fa-save"></i> Simpan
            </button>
          </div>
        </form>
      </div>

      <div class="card-footer">
        <button class="btn btn-secondary" @click="$emit('close')">
          <i class="fas fa-times"></i> Tutup
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  props: ['member', 'mode'],
  data() {
    return {
      form: {
        nama: '',
        no_ktp: '',
        tgl_lahir: '',
        alamat: ''
      }
    }
  },
  watch: {
    member: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.form.nama = newVal.nama
          this.form.no_ktp = newVal.no_ktp
          this.form.tgl_lahir = newVal.tgl_lahir
          this.form.alamat = newVal.alamat
        }
      }
    }
  },
  methods: {
    async updateMember() {
      try {
        await axios.put(
          `http://45.64.100.26:88/perpus-api/public/api/member/${this.member.id}`,
          this.form,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          }
        )

        this.$emit('refresh')
        this.$emit('close')
      } catch (error) {
        console.error('Gagal update member:', error)
        alert('Gagal menyimpan perubahan.')
      }
    },
    formatDate(dateStr) {
      const date = new Date(dateStr)
      return date.toLocaleDateString('id-ID', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.modal-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  width: 600px;
  max-width: 90vw;
  overflow: hidden;
  animation: modalFadeIn 0.3s ease-out;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-header {
  background-color: #2980b9;
  color: white;
  padding: 16px 24px;
  font-weight: 600;
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
  font-size: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-body {
  padding: 24px;
  background-color: #f9f9f9;
}

.card-footer {
  padding: 12px 24px;
  background-color: #f1f1f1;
  border-top: 1px solid #e2e2e2;
  display: flex;
  justify-content: flex-end;
  border-bottom-left-radius: 12px;
  border-bottom-right-radius: 12px;
}

.close-btn {
  background: none;
  border: none;
  padding: 6px;
  cursor: pointer;
  color: white;
  transition: color 0.3s;
}

.close-btn:hover {
  color: #e74c3c;
}

.form-row {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.form-group {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.detail-label {
  font-weight: 600;
  margin-bottom: 8px;
  color: #2980b9;
}

.detail-value {
  padding: 10px;
  color: #34495e;
  background: #ecf0f1;
  border-radius: 8px;
}

.input-label {
  font-weight: 600;
  margin-bottom: 8px;
  color: #2980b9;
}

.form-control {
  padding: 12px 16px;
  border: 1.5px solid #ccc;
  border-radius: 8px;
  font-size: 1rem;
  outline-offset: 2px;
  transition: border-color 0.3s ease;
}

.form-control:focus {
  border-color: #2980b9;
  outline: none;
}

.form-actions {
  margin-top: 20px;
  display: flex;
  gap: 15px;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  font-size: 1rem;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  transition: background-color 0.3s ease;
}

.btn-primary {
  background-color: #2980b9;
  color: white;
}

.btn-primary:hover {
  background-color: #1f6391;
}

.btn-secondary {
  background-color: #7f8c8d;
  color: white;
}

.btn-secondary:hover {
  background-color: #606b6d;
}

@media (max-width: 768px) {
  .form-row {
    flex-direction: column;
    gap: 12px;
  }

  .form-group {
    width: 100%;
  }
}
</style>
