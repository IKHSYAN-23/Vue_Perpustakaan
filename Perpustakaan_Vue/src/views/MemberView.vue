<template>
  <div class="member-page">
    <!-- Header Bar -->
    <div class="header-bar">
      <h1 class="page-title">Daftar Member</h1>
      <button @click="openForm" class="btn-add">Tambah Member</button>
    </div>

    <!-- Form Tambah/Edit -->
    <transition name="fade">
      <div v-if="showForm" class="card form-card">
        <h3>{{ isEditMode ? 'Edit Member' : 'Tambah Member Baru' }}</h3>
        <form @submit.prevent="submitMember" class="form-grid">
          <div class="form-group">
            <label>Nama</label>
            <input v-model="form.nama" required />
          </div>
          <div class="form-group">
            <label>No. KTP</label>
            <input
              v-model="form.no_ktp"
              required
              minlength="16"
              maxlength="16"
              pattern="\d{16}"
              title="Nomor KTP harus 16 digit angka"
              inputmode="numeric"
            />
          </div>
          <div class="form-group">
            <label>Alamat</label>
            <input v-model="form.alamat" required />
          </div>
          <div class="form-group">
            <label>Tanggal Lahir</label>
            <input v-model="form.tgl_lahir" type="date" required />
          </div>
          <div class="form-actions">
            <button type="submit" :disabled="loading" class="btn-primary">
              {{ loading ? 'Menyimpan...' : (isEditMode ? 'Update' : 'Simpan') }}
            </button>
            <button type="button" @click="cancelForm" :disabled="loading" class="btn-secondary">
              Batal
            </button>
          </div>
        </form>
        <p v-if="errorMessage" class="alert error">{{ errorMessage }}</p>
        <p v-if="successMessage" class="alert success">{{ successMessage }}</p>
      </div>
    </transition>

    <!-- Tabel Member -->
    <div class="card table-card">
      <div class="table-container">
        <table class="member-table">
          <thead>
            <tr>
              <th>Nama</th>
              <th>No. KTP</th>
              <th>Alamat</th>
              <th>Tanggal Lahir</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="!members.length">
              <td colspan="5" class="empty">Belum ada data member</td>
            </tr>
            <tr v-for="member in members" :key="member.id">
              <td>{{ member.nama }}</td>
              <td>{{ member.no_ktp }}</td>
              <td>{{ member.alamat }}</td>
              <td>{{ formatDate(member.tgl_lahir) }}</td>
              <td>
                <div class="actions">
                  <button @click="viewDetail(member)" class="action detail">Lihat</button>
                  <button @click="editMember(member)" class="action edit">Edit</button>
                  <button @click="openRiwayatModal(member)" class="action history">Riwayat</button>
                  <button @click="promptDelete(member)" class="action delete">Hapus</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modal Detail Member -->
    <div v-if="selectedMember" class="modal-overlay" @click.self="selectedMember = null">
      <div class="modal-content">
        <h3>Detail Member</h3>
        <p><strong>Nama:</strong> {{ selectedMember.nama }}</p>
        <p><strong>No. KTP:</strong> {{ selectedMember.no_ktp }}</p>
        <p><strong>Alamat:</strong> {{ selectedMember.alamat }}</p>
        <p><strong>Tanggal Lahir:</strong> {{ formatDate(selectedMember.tgl_lahir) }}</p>
        <button @click="selectedMember = null">Tutup</button>
      </div>
    </div>

    <!-- Modal Riwayat Peminjaman -->
    <div v-if="showRiwayatModal" class="modal-overlay" @click.self="closeRiwayatModal">
      <div class="modal-content large-modal">
        <div class="modal-header">
          <h3>Riwayat Peminjaman</h3>
          <div class="header-actions">
            <button @click="exportRiwayatPDF" class="btn-export">Export PDF</button>
            <button @click="closeRiwayatModal" class="btn-close">Tutup</button>
          </div>
        </div>
        <div class="member-selection">
          <label>Pilih Member:</label>
          <select v-model="selectedMemberId" @change="fetchRiwayatPeminjaman">
            <option value="">-- Pilih Member --</option>
            <option v-for="m in members" :key="m.id" :value="m.id">
              {{ m.nama }} ({{ m.no_ktp }})
            </option>
          </select>
        </div>
        <div v-if="loadingRiwayat" class="loading">Memuat data...</div>
        <div v-else-if="riwayatError" class="alert error">{{ riwayatError }}</div>
        <div v-if="selectedMemberForRiwayat">
          <h4>Riwayat {{ selectedMemberForRiwayat.nama }}</h4>
          <table v-if="riwayatPeminjaman.length" class="riwayat-table">
            <thead>
              <tr>
                <th>No</th>
                <th>Judul Buku</th>
                <th>Tgl Pinjam</th>
                <th>Tgl Kembali</th>
                <th>Status</th>
                <th>Denda</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(p, i) in riwayatPeminjaman" :key="p.id">
                <td>{{ i+1 }}</td>
                <td>{{ p.buku.judul }}</td>
                <td>{{ formatDate(p.tgl_pinjam) }}</td>
                <td>{{ formatDate(p.tgl_kembali) }}</td>
                <td :class="'status-' + p.status.toLowerCase()">{{ getStatusText(p.status) }}</td>
                <td>{{ formatCurrency(p.denda) }}</td>
              </tr>
            </tbody>
          </table>
          <div v-else class="no-data">Tidak ada riwayat peminjaman</div>
          <div class="total-denda" v-if="totalDenda > 0">
            <strong>Total Denda:</strong> {{ formatCurrency(totalDenda) }}
          </div>
        </div>
      </div>
    </div>

    <!-- Modal Hapus Member -->
    <div v-if="showDeleteModal" class="modal-overlay" @click.self="cancelDelete">
      <div class="modal-content">
        <h3>Konfirmasi Hapus</h3>
        <p>Apakah Anda yakin ingin menghapus member "<strong>{{ memberToDelete.nama }}</strong>"?</p>
        <div class="form-actions" style="justify-content:flex-end; margin-top:1rem;">
          <button @click="confirmDelete" class="btn-primary">Ya</button>
          <button @click="cancelDelete" class="btn-secondary">Batal</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { nextTick } from 'vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

export default {
  name: 'memberView',
  data() {
    return {
      // Form
      showForm: false,
      isEditMode: false,
      loading: false,
      errorMessage: '',
      successMessage: '',
      form: { id: null, nama: '', no_ktp: '', alamat: '', tgl_lahir: '' },

      // Data Member
      members: [],
      selectedMember: null,

      // Riwayat
      showRiwayatModal: false,
      loadingRiwayat: false,
      riwayatError: '',
      riwayatPeminjaman: [],
      totalDenda: 0,
      selectedMemberId: '',
      selectedMemberForRiwayat: null,

      // Hapus
      showDeleteModal: false,
      memberToDelete: null,
    };
  },
  mounted() {
    this.fetchMembers();
  },
  methods: {
    // Fetch Semua Member
    async fetchMembers() {
      const token = localStorage.getItem('token');
      try {
        const res = await axios.get('http://45.64.100.26:88/perpus-api/public/api/member', {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.members = res.data.data || res.data;
      } catch {
        this.errorMessage = 'Gagal mengambil data member';
      }
    },

    // Form Tambah/Edit
    openForm() {
      this.resetForm();
      this.isEditMode = false;
      this.showForm = true;
      nextTick(() => window.scrollTo(0, 0));
    },
    editMember(member) {
      this.form = { ...member };
      this.isEditMode = true;
      this.showForm = true;
      this.errorMessage = '';
      this.successMessage = '';
      nextTick(() => window.scrollTo(0, 0));
    },
    async submitMember() {
      this.loading = true; this.errorMessage = ''; this.successMessage = '';
      const token = localStorage.getItem('token');
      try {
        if (this.isEditMode) {
          await axios.put(`http://45.64.100.26:88/perpus-api/public/api/member/${this.form.id}`, this.form, { headers: { Authorization: `Bearer ${token}` } });
          this.successMessage = 'Member berhasil diperbarui.';
        } else {
          await axios.post('http://45.64.100.26:88/perpus-api/public/api/member', this.form, { headers: { Authorization: `Bearer ${token}` } });
          this.successMessage = 'Member berhasil ditambahkan.';
        }
        await this.fetchMembers();
        this.cancelForm();
      } catch {
        this.errorMessage = 'Gagal menyimpan data member.';
      } finally {
        this.loading = false;
      }
    },

    // Ekspor Riwayat ke PDF
    async exportRiwayatPDF() {
      await this.fetchRiwayatPeminjaman();
      const modalContent = this.$el.querySelector('.modal-content.large-modal');
      html2canvas(modalContent, { scale: 2 }).then(canvas => {
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF({ orientation: 'landscape', unit: 'px', format: [canvas.width, canvas.height] });
        pdf.addImage(imgData, 'PNG', 0, 0, canvas.width, canvas.height);
        pdf.save(`Riwayat_${this.selectedMemberForRiwayat.nama}.pdf`);
      }).catch(err => {
        console.error(err);
        window.alert('Gagal membuat PDF riwayat.');
      });
    },

    // Hapus via Modal
    promptDelete(member) { this.memberToDelete = member; this.showDeleteModal = true; },
    cancelDelete() { this.showDeleteModal = false; this.memberToDelete = null; },
    async confirmDelete() {
      this.showDeleteModal = false;
      const token = localStorage.getItem('token');
      try {
        await axios.delete(`http://45.64.100.26:88/perpus-api/public/api/member/${this.memberToDelete.id}`, { headers: { Authorization: `Bearer ${token}` } });
        await this.fetchMembers();
        window.alert('Member berhasil dihapus.');
      } catch {
        window.alert('Gagal menghapus member.');
      } finally {
        this.memberToDelete = null;
      }
    },

    // Detail Member
    viewDetail(member) { this.selectedMember = member; },

    // Riwayat Peminjaman
    openRiwayatModal(member) {
      this.selectedMemberId = member.id;
      this.showRiwayatModal = true;
      this.fetchRiwayatPeminjaman();
    },
    closeRiwayatModal() {
      this.showRiwayatModal = false; this.selectedMemberId = ''; this.selectedMemberForRiwayat = null;
      this.riwayatPeminjaman = []; this.totalDenda = 0;
    },
    async fetchRiwayatPeminjaman() {
      if (!this.selectedMemberId) return;
      this.loadingRiwayat = true; this.riwayatError = '';
      const token = localStorage.getItem('token');
      try {
        this.selectedMemberForRiwayat = this.members.find(m => m.id == this.selectedMemberId);
        const res = await axios.get(`http://45.64.100.26:88/perpus-api/public/api/peminjaman?member_id=${this.selectedMemberId}`, { headers: { Authorization: `Bearer ${token}` } });
        let raw = Array.isArray(res.data) ? res.data : (res.data.data || []);
        raw = raw.filter(item => item.member_id == this.selectedMemberId || item.id_member == this.selectedMemberId);
        this.riwayatPeminjaman = raw.map(item => {
          const tP = item.tanggal_pinjam || item.tgl_pinjam;
          const tK = item.tanggal_kembali || item.tgl_kembali || item.tgl_pengembalian;
          const statusRaw = (item.status || '').toUpperCase();
          const due = item.tgl_jatuh_tempo || this.calculateDueDate(tP);
          const isReturned = statusRaw === 'KEMBALI';
          const isLost = statusRaw === 'HILANG';
          const denda = item.denda != null
            ? Number(item.denda)
            : (!isReturned && this.isOverdue(due) ? this.calculateFine(due) : 0);
          let status = 'PINJAM';
          if (isReturned) status = 'KEMBALI';
          else if (isLost) status = 'HILANG';
          else if (this.isOverdue(due)) status = 'TERLAMBAT';
          return { id: item.id || item.id_peminjaman, buku: { judul: item.buku?.judul || item.judul_buku || '—' }, tgl_pinjam: tP, tgl_kembali: tK, status, denda };
        });
        this.totalDenda = this.riwayatPeminjaman.reduce((sum, x) => sum + (x.denda || 0), 0);
      } catch {
        this.riwayatError = 'Gagal mengambil riwayat peminjaman.';
      } finally {
        this.loadingRiwayat = false;
      }
    },

    // Utility
    calculateDueDate(pinjam) {
      if (!pinjam) return null;
      const d = new Date(pinjam);
      d.setDate(d.getDate() + 7);
      return d.toISOString().split('T')[0];
    },
    isOverdue(due) { return due && new Date() > new Date(due); },
    calculateDaysLate(due) {
      return this.isOverdue(due)
        ? Math.ceil((new Date() - new Date(due)) / 8.64e7)
        : 0;
    },
    calculateFine(due) { const days = this.calculateDaysLate(due); return Math.min(days * 5000, 5000 * 30); },
    formatDate(str) { if (!str) return '-'; const d = new Date(str); return isNaN(d) ? str : d.toLocaleDateString('id-ID', { year:'numeric', month:'long', day:'numeric' }); },
    getStatusText(s) { const m = { PINJAM:'Dipinjam', KEMBALI:'Dikembalikan', HILANG:'Hilang', TERLAMBAT:'Terlambat' }; return m[s.toUpperCase()] || s; },
    formatCurrency(v) { return new Intl.NumberFormat('id-ID',{ style:'currency', currency:'IDR', minimumFractionDigits:0 }).format(v||0); },
    cancelForm() { this.resetForm(); this.showForm = false; this.isEditMode = false; this.errorMessage = ''; this.successMessage = ''; },
    resetForm() { this.form = { id:null, nama:'', no_ktp:'', alamat:'', tgl_lahir:'' }; }
  }
};
</script>

<style scoped>
.member-page { padding: 2rem; background: #f4f6fa; min-height: 100vh; }
.header-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1.5rem; }
.page-title { font-size: 2rem; color: #333; margin: 0; border-bottom: 3px solid #4caf50; padding-bottom: 0.5rem; }
.btn-add { background-color: #4caf50; color: white; padding: 0.6rem 1.2rem; border: none; border-radius: 0.5rem; font-weight: bold; box-shadow: 0 2px 6px rgba(0,0,0,0.15); transition: transform 0.2s; }
.btn-add:hover { transform: translateY(-2px); }
.card { background: white; border-radius: 0.75rem; box-shadow: 0 4px 12px rgba(0,0,0,0.1); padding: 1.5rem; margin-bottom: 1.5rem; }
.form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem; align-items: end; }
.form-group label { display: block; font-weight: 600; color: #555; margin-bottom: 0.3rem; }
.form-group input { width: 100%; padding: 0.6rem; border: 1px solid #ccc; border-radius: 0.5rem; }
.form-actions { grid-column: 1 / -1; display: flex; gap: 1rem; }
.btn-primary { background-color: #2196f3; color: white; padding: 0.6rem 1.2rem; border: none; border-radius: 0.5rem; cursor: pointer; flex: 1; }
.btn-secondary { background-color: #e0e0e0; color: #555; padding: 0.6rem 1.2rem; border: none; border-radius: 0.5rem; cursor: pointer; flex: 1; }
.alert { margin-top: 1rem; padding: 0.8rem; border-radius: 0.5rem; }
.alert.error { background: #ffebee; color: #c62828; }
.alert.success { background: #e8f5e9; color: #2e7d32; }
.table-card { padding: 0; }
.table-container { overflow-x: auto; }
.member-table { width: 100%; border-collapse: separate; border-spacing: 0; }
.member-table th, .member-table td { padding: 1rem; text-align: left; }
.member-table thead th { background: #4caf50; color: white; position: sticky; top: 0; }
.member-table tbody tr:nth-child(even) { background: #fafafa; }
.member-table tbody tr:hover { background: #f1f5f9; }
.empty { text-align: center; color: #888; font-style: italic; }
.actions { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.action { padding: 0.4rem 0.8rem; border: none; border-radius: 0.4rem; font-size: 0.85rem; font-weight: 600; cursor: pointer; }
.detail  { background: #3f51b5; color: #fff; }
.edit    { background: #ffb300; color: #fff; }
.history { background: #9c27b0; color: #fff; }
.delete  { background: #f44336; color: #fff; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter, .fade-leave-to { opacity: 0; }
.modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 1000; }
.modal-content { background: white; padding: 1.5rem; border-radius: 0.75rem; width: 500px; max-width: 95%; max-height: 90vh; overflow-y: auto; box-shadow: 0 4px 12px rgba(0,0,0,0.2); position: relative; z-index: 1001; }
.modal-content.large-modal { width: 800px; }
.modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; padding-bottom: 0.5rem; border-bottom: 1px solid #eee; }
.header-actions { display: flex; gap: 0.5rem; }
.btn-export { background-color: #007bff; color: #fff; border: none; padding: 0.4rem 0.8rem; border-radius: 0.4rem; cursor: pointer; font-weight: 600; }
.btn-export:hover { opacity: 0.9; } 
.btn-close { background-color: #f44336; color: #fff; border: none; padding: 0.4rem 0.8rem; border-radius: 0.4rem; cursor: pointer; }
.riwayat-table { width: 100%; border-collapse: collapse; margin: 1rem 0; background: #fff; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.riwayat-table th { background: #607d8b; color: #fff; padding: 0.75rem; text-align: left; }
.riwayat-table td { padding: 0.6rem; border-bottom: 1px solid #eee; }\n.no-data, .loading { text-align: center; padding: 1rem; color: #555; }
.total-denda { margin-top: 1rem; font-size: 1rem; color: #2e7d32; }
.status-pinjam    { color: #ff9800; font-weight: bold; }
.status-kembali   { color: #4caf50; font-weight: bold; }
.status-hilang    { color: #f44336; font-weight: bold; }
.status-terlambat { color: #e91e63; font-weight: bold; }
.status-unknown   { color: #9e9e9e; font-weight: bold; }
</style>
