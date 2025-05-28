<template>
  <transition name="sidebar">
    <aside :class="['sidebar-container', { collapsed } ]" role="navigation" aria-label="Sidebar menu">
      <div class="sidebar">
        <header class="sidebar-header">
          <button
            class="collapse-btn"
            @click="toggleCollapse"
            :aria-expanded="!collapsed"
            aria-label="Toggle sidebar"
          >
            <i :class="collapsed ? 'fas fa-chevron-right' : 'fas fa-chevron-left'"></i>
          </button>
          <h1 v-if="!collapsed" class="sidebar-brand">📚 Perpustakaan</h1>
        </header>

        <div v-if="!collapsed" class="sidebar-search" role="search">
          <label for="sidebar-search-input" class="sr-only">Cari menu</label>
          <input
            id="sidebar-search-input"
            type="search"
            v-model="search"
            placeholder="Cari menu..."
            class="search-input"
          />
          <i class="fas fa-search search-icon" aria-hidden="true"></i>
        </div>

        <ul class="sidebar-nav">
          <li v-for="item in filteredItems" :key="item.text">
            <router-link
              :to="item.to"
              class="nav-item"
              active-class="active"
              exact
              :aria-current="isActive(item.to) ? 'page' : null"
              :title="collapsed ? item.text : null"
            >
              <i :class="item.icon" class="nav-icon"></i>
              <span v-if="!collapsed" class="nav-text">{{ item.text }}</span>
            </router-link>
          </li>
        </ul>

        <footer class="sidebar-footer">
          <button
            class="logout-btn"
            @click="logout"
            aria-label="Logout"
            :title="collapsed ? 'Logout' : null"
          >
            <i class="fas fa-sign-out-alt"></i>
            <span v-if="!collapsed">Logout</span>
          </button>
        </footer>
      </div>
    </aside>
  </transition>
</template>

<script>
export default {
  name: 'Sidebar',
  data() {
    return {
      collapsed: false,
      search: '',
      items: [
        { to: '/dashboard', icon: 'fas fa-tachometer-alt', text: 'Dashboard' },
        { to: '/members', icon: 'fas fa-users', text: 'Member' },
        { to: '/buku', icon: 'fas fa-book', text: 'Kelola Buku' },
        { to: '/peminjaman', icon: 'fas fa-hand-holding', text: 'Peminjaman' },
        { to: '/pengembalian', icon: 'fas fa-undo-alt', text: 'Pengembalian' },
      ],
    };
  },
  computed: {
    filteredItems() {
      if (!this.search) return this.items;
      return this.items.filter(i =>
        i.text.toLowerCase().includes(this.search.toLowerCase())
      );
    },
  },
  methods: {
    toggleCollapse() {
      this.collapsed = !this.collapsed;
    },
    isActive(route) {
      return this.$route.path === route;
    },
    logout() {
      localStorage.removeItem('token');
      this.$router.push('/');
    },
  },
};
</script>

<style scoped>
:root {
  --sidebar-width: 240px;
  --sidebar-collapsed-width: 64px;
  --bg-gradient: linear-gradient(135deg, #2c3e50, #3498db);
}

.sidebar-enter-active, .sidebar-leave-active {
  transition: width 0.3s ease;
}
.sidebar-enter-from, .sidebar-leave-to {
  width: var(--sidebar-collapsed-width);
}
.sidebar-enter-to, .sidebar-leave-from {
  width: var(--sidebar-width);
}

.sidebar-container {
  position: fixed;
  top: 0;
  left: 0;
  width: var(--sidebar-width);
  height: 100vh;
  background: #2c3e50;
  box-shadow: 2px 0 10px rgba(0,0,0,0.15);
  color: #fff;
  overflow: hidden;
}

.sidebar {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: var(--bg-gradient);
}

.sidebar-header {
  display: flex;
  align-items: center;
  padding: 16px;
  border-bottom: 1px solid rgba(255,255,255,0.2);
}

.collapse-btn {
  background: none;
  border: none;
  color: #fff;
  cursor: pointer;
  font-size: 1.1rem;
  padding: 8px;
  transition: transform 0.2s;
}
.collapse-btn:focus {
  outline: 2px solid #f39c12;
}
.collapse-btn:hover {
  transform: scale(1.1);
}

.sidebar-brand {
  font-size: 1.4rem;
  font-weight: 700;
  margin: 0;
  margin-left: 12px;
}

.sidebar-search {
  position: relative;
  padding: 12px 16px;
}
.search-input {
  width: 100%;
  padding: 8px 32px 8px 12px;
  border: none;
  border-radius: 4px;
}
.search-input:focus {
  outline: 2px solid #f39c12;
}
.search-icon {
  position: absolute;
  right: 24px;
  top: 50%;
  transform: translateY(-50%);
  color: #999;
}

.sidebar-nav {
  flex: 1;
  overflow-y: auto;
  margin: 0;
  padding: 16px 0;
  list-style: none;
}
.nav-item {
  display: flex;
  align-items: center;
  padding: 10px 20px;
  margin: 4px 12px;
  border-radius: 6px;
  text-decoration: none;
  color: rgba(255,255,255,0.9);
  transition: background 0.2s;
}
.nav-item:focus {
  outline: 2px solid #f39c12;
}
.nav-item:hover {
  background: rgba(255,255,255,0.15);
}
.nav-item.active {
  background: rgba(255,255,255,0.3);
  border-left: 4px solid #f39c12;
  color: #fff;
}
.nav-icon {
  width: 24px;
  text-align: center;
  margin-right: 12px;
  font-size: 1.2rem;
}
.nav-text {
  flex: 1;
  white-space: nowrap;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid rgba(255,255,255,0.2);
}
.logout-btn {
  width: 100%;
  background: none;
  border: none;
  display: flex;
  align-items: center;
  padding: 10px 20px;
  cursor: pointer;
  color: rgba(255,255,255,0.9);
  border-radius: 6px;
  transition: background 0.2s;
}
.logout-btn i {
  margin-right: 12px;
  font-size: 1.2rem;
}
.logout-btn:hover {
  background: rgba(231,76,60,0.3);
  color: #fff;
}

/* Accessibility helper */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0,0,0,0);
  white-space: nowrap;
  border: 0;
}

/* Responsive */
@media (max-width: 768px) {
  :root {
    --sidebar-width: var(--sidebar-collapsed-width);
  }
  .sidebar-header h1,
  .nav-text,
  .sidebar-search,
  .logout-btn span {
    display: none;
  }
} 
</style>