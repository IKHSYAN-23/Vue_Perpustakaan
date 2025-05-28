<template>
  <div class="dashboard-page">
    <header class="header">
      <h1>Dashboard Perpustakaan</h1>
      <p class="subtitle">Ringkasan statistik terkini</p>
    </header>

    <section class="stats-grid">
      <div class="stat-card">
        <div class="icon">📚</div>
        <div class="info">
          <h2>{{ totalBuku }}</h2>
          <p>Total Buku</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="icon">👥</div>
        <div class="info">
          <h2>{{ totalAnggota }}</h2>
          <p>Total Anggota</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="icon">🔄</div>
        <div class="info">
          <h2>{{ totalPeminjaman }}</h2>
          <p>Total Peminjaman</p>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DashboardView",
  data() {
    return {
      totalBuku: 0,
      totalAnggota: 0,
      totalPeminjaman: 0,
    };
  },
  mounted() {
    this.fetchTotalBuku();
    this.fetchTotalAnggota();
    this.fetchTotalPeminjaman();
  },
  methods: {
    async fetchTotalBuku() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "http://45.64.100.26:88/perpus-api/public/api/buku",
          { headers: { Authorization: `Bearer ${token}` } }
        );
        const data = res.data.data || res.data;
        this.totalBuku = Array.isArray(data) ? data.length : 0;
      } catch (err) {
        console.error("Gagal ambil data buku", err);
      }
    },
    async fetchTotalAnggota() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "http://45.64.100.26:88/perpus-api/public/api/member",
          { headers: { Authorization: `Bearer ${token}` } }
        );
        const data = res.data.data || res.data;
        this.totalAnggota = Array.isArray(data) ? data.length : 0;
      } catch (err) {
        console.error("Gagal ambil data anggota", err);
      }
    },
    async fetchTotalPeminjaman() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "http://45.64.100.26:88/perpus-api/public/api/peminjaman",
          { headers: { Authorization: `Bearer ${token}` } }
        );
        const data = res.data.data || res.data;
        this.totalPeminjaman = Array.isArray(data) ? data.length : 0;
      } catch (err) {
        console.error("Gagal ambil data peminjaman", err);
      }
    },
  },
};
</script>

<style scoped>
.dashboard-page {
  height: 40vh;                      /* Penuhi tinggi viewport */
  display: flex;
  flex-direction: column;
  justify-content: center;            /* Pusat vertikal */
  align-items: center;                /* Pusat horizontal */
  background: linear-gradient(135deg, #eef2ff, #dfe7fd);
  padding: 2rem 1rem;
  box-sizing: border-box;
}

/* HEADER */
.header {
  text-align: center;
  margin-bottom: 2rem;
}
.header h1 {
  font-size: 2.75rem;
  font-weight: 700;
  color: #333;
  margin: 0;
  position: relative;
  display: inline-block;
}
.header h1::after {
  content: "";
  position: absolute;
  width: 60%;
  height: 4px;
  background: #4f6ef7;
  bottom: -6px;
  left: 20%;
  border-radius: 2px;
}
.header .subtitle {
  margin-top: 0.5rem;
  color: #555;
  font-size: 1.1rem;
}

/* GRID STAT */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1.75rem;
  width: 100%;
  max-width: 1000px;
}

/* CARD STAT */
.stat-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 1.5rem;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
  display: flex;
  align-items: center;
  transition: transform 0.2s, box-shadow 0.2s;
}
.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 24px rgba(0, 0, 0, 0.12);
}

/* ICON */
.icon {
  font-size: 2.5rem;
  margin-right: 1rem;
  animation: bounce 2s infinite;
}

/* INFO */
.info h2 {
  margin: 0;
  font-size: 2rem;
  color: #4f6ef7;
}
.info p {
  margin: 0.25rem 0 0;
  color: #666;
  font-size: 1rem;
}

/* BOUNCE ANIMATION ICON */
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0) }
  40% { transform: translateY(-6px) }
  60% { transform: translateY(-3px) }
}

/* RESPONSIVE PADDING */
@media (max-width: 480px) {
  .dashboard-page {
    padding: 1rem;
  }
  .header h1 {
    font-size: 2rem;
  }
}
</style>