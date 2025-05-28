<template>
  <div class="riwayat-peminjaman">
    <h3>Riwayat Peminjaman</h3>
    
    <div v-if="loading" class="loading">Memuat data...</div>
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
    
    <table v-else class="peminjaman-table">
      <thead>
        <tr>
          <th>ID Peminjaman</th>
          <th>Judul Buku</th>
          <th>Tanggal Pinjam</th>
          <th>Tanggal Kembali</th>
          <th>Status</th>
          <th>Denda</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="peminjamanList.length === 0">
          <td colspan="6" class="empty">Tidak ada riwayat peminjaman</td>
        </tr>
        <tr v-for="peminjaman in peminjamanList" :key="peminjaman.id">
          <td>{{ peminjaman.id }}</td>
          <td>{{ peminjaman.buku.judul }}</td>
          <td>{{ formatDate(peminjaman.tanggal_pinjam) }}</td>
          <td>{{ formatDate(peminjaman.tanggal_kembali) }}</td>
          <td>
            <span :class="'status-' + peminjaman.status.toLowerCase()">
              {{ getStatusText(peminjaman.status) }}
            </span>
          </td>
          <td>{{ formatCurrency(peminjaman.denda) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "RiwayatPeminjaman",
  props: {
    memberId: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      loading: false,
      errorMessage: '',
      peminjamanList: []
    };
  },
  watch: {
    memberId: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.fetchRiwayatPeminjaman();
        } else {
          this.peminjamanList = [];
        }
      }
    }
  },
  methods: {
   async fetchRiwayatPeminjaman() {
  this.loading = true;
  this.errorMessage = '';
  this.peminjamanList = [];

  try {
    const token = localStorage.getItem('token');
    if (!token) {
      throw new Error('Token tidak ditemukan, silakan login kembali');
    }

    if (!this.memberId) {
      throw new Error('ID Member tidak valid');
    }

    const headers = {
      'Authorization': `Bearer ${token}`,
      'Accept': 'application/json'
    };

    // Gunakan parameter query seperti teman Anda
    const response = await axios.get(
      `http://45.64.100.26:88/perpus-api/public/api/peminjaman?member_id=${this.memberId}`,
      { headers }
    );

    // Debug response
    console.log('API Response:', response.data);

    // Handle response dengan cara yang lebih robust
    let rawData = [];
    if (Array.isArray(response.data)) {
      rawData = response.data;
    } else if (response.data?.data) {
      rawData = response.data.data;
    }

    // Filter dan mapping data
    this.peminjamanList = rawData
      .filter(item => item.member_id == this.memberId || item.id_member == this.memberId)
      .map(item => ({
        id: item.id || item.id_peminjaman,
        buku: {
          id: item.buku?.id || item.id_buku || '',
          judul: item.buku?.judul || item.judul_buku || 'Judul tidak tersedia'
        },
        tanggal_pinjam: item.tanggal_pinjam || item.tgl_pinjam,
        tanggal_kembali: item.tanggal_kembali || item.tgl_kembali,
        status: this.normalizeStatus(item.status),
        denda: item.denda || 0
      }));

  } catch (error) {
    console.error('Error fetching peminjaman:', error);
    this.errorMessage = this.getErrorMessage(error);
  } finally {
    this.loading = false;
  }
},

// Tambahkan method helper baru
normalizeStatus(status) {
  if (!status) return 'PINJAM';
  status = status.toUpperCase();
  return status === 'KEMBALI' ? 'KEMBALI' :
    status === 'HILANG' ? 'HILANG' :
    status === 'TERLAMBAT' ? 'TERLAMBAT' : 'PINJAM';
},

getErrorMessage(error) {
  if (error.response) {
    switch (error.response.status) {
      case 401: return "Anda tidak memiliki akses";
      case 404: return "Data tidak ditemukan";
      case 500: return "Server error";
      default: return error.response.data?.message || "Gagal memuat data";
    }
  }
  return error.message || "Gagal memuat riwayat peminjaman";
},
    formatDate(dateString) {
      if (!dateString) return "-";
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString('id-ID', options);
    },
    formatCurrency(amount) {
      if (!amount) return "Rp 0";
      return new Intl.NumberFormat('id-ID', { 
        style: 'currency', 
        currency: 'IDR',
        minimumFractionDigits: 0
      }).format(amount);
    },
    getStatusText(status) {
      const statusMap = {
        'PINJAM': 'Dipinjam',
        'KEMBALI': 'Dikembalikan',
        'HILANG': 'Hilang',
        'TERLAMBAT': 'Terlambat'
      };
      return statusMap[status] || status;
    }
  }
};
</script>

<style scoped>
.riwayat-peminjaman {
  margin-top: 2rem;
  padding: 1rem;
  border: 1px solid #eee;
  border-radius: 8px;
  background: #fafafa;
}

.peminjaman-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.peminjaman-table th,
.peminjaman-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

.peminjaman-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.empty {
  text-align: center;
  color: #888;
  padding: 20px;
}

.loading {
  padding: 20px;
  text-align: center;
  color: #666;
}

.error-message {
  padding: 10px;
  background-color: #ffebee;
  color: #c62828;
  border-radius: 4px;
  margin-bottom: 10px;
}

.status-pinjam {
  color: #ff9800;
  font-weight: bold;
}

.status-kembali {
  color: #4caf50;
  font-weight: bold;
}

.status-hilang {
  color: #f44336;
  font-weight: bold;
}

.status-terlambat {
  color: #e91e63;
  font-weight: bold;
}
</style>