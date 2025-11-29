<!-- src/App.vue -->
<script setup>
import { ref, onMounted, watch } from 'vue';
import AuthModal from './components/AuthModal.vue';

const cvFile = ref(null);
const jobQuery = ref('');

const showAuthModal = ref(false);
const authMode = ref('login'); // 'login' | 'signup'
const isLoggedIn = ref(false);
const userEmail = ref('');
const pendingSearch = ref(false);

// theme: 'dark' | 'light'
const theme = ref('dark');

function applyTheme(value) {
  document.documentElement.setAttribute('data-theme', value);
}

onMounted(() => {
  // default: ikuti data-theme di html kalau ada
  const current = document.documentElement.getAttribute('data-theme');
  theme.value = current === 'light' ? 'light' : 'dark';
  applyTheme(theme.value);
});

watch(theme, (val) => {
  applyTheme(val);
});

function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark';
}

// dummy job results
const hasSearched = ref(false);
const jobResults = ref([]);

const dummyJobs = [
  {
    id: 1,
    rank: 1,
    title: 'Software Developer (.NET)',
    company: 'Savant Degrees Pte Ltd',
    location: 'Jakarta',
    employmentType: 'Full-time',
    workMode: 'Hybrid',
    matchScore: 95,
    description:
      'This job perfectly matches your search query for a Software Developer (.NET) in Jakarta with a preferred Hybrid work setting.',
    url: 'https://example.com/jobs/software-developer-dotnet'
  },
  {
    id: 2,
    rank: 2,
    title: 'Backend Engineer (.NET Core)',
    company: 'Fintech Nusantara',
    location: 'Jakarta',
    employmentType: 'Full-time',
    workMode: 'On-site',
    matchScore: 92,
    description:
      'Work with a modern .NET Core stack to build payment and reconciliation services for enterprise clients.',
    url: 'https://example.com/jobs/backend-engineer-dotnet-core'
  },
  {
    id: 3,
    rank: 3,
    title: 'Full Stack Engineer (.NET & React)',
    company: 'Digital Solutions ID',
    location: 'Remote from Indonesia',
    employmentType: 'Contract',
    workMode: 'Remote',
    matchScore: 88,
    description:
      'Join a distributed team to build internal dashboards using .NET APIs and a React front-end.',
    url: 'https://example.com/jobs/fullstack-dotnet-react'
  },
  {
    id: 4,
    rank: 4,
    title: 'Senior .NET Developer',
    company: 'Global Retail Group',
    location: 'Tangerang',
    employmentType: 'Full-time',
    workMode: 'Hybrid',
    matchScore: 86,
    description:
      'Lead the migration of legacy .NET Framework services to .NET 8 and mentor a small dev team.',
    url: 'https://example.com/jobs/senior-dotnet-developer'
  },
  {
    id: 5,
    rank: 5,
    title: 'Application Developer (.NET)',
    company: 'Bank Digital Indonesia',
    location: 'Jakarta',
    employmentType: 'Full-time',
    workMode: 'On-site',
    matchScore: 83,
    description:
      'Build core banking features and internal tools with strict security and performance requirements.',
    url: 'https://example.com/jobs/application-developer-dotnet'
  }
];

function runDummySearch() {
  hasSearched.value = true;
  // di real app kamu bisa filter berdasarkan jobQuery, di sini kita tampilkan semua
  jobResults.value = [...dummyJobs];
}

function handleSearchClick() {
  // validasi ringan
  if (!cvFile.value && !jobQuery.value.trim()) {
    alert('Silakan upload CV atau isi pekerjaan yang Anda cari terlebih dahulu.');
    return;
  }

  if (!isLoggedIn.value) {
    pendingSearch.value = true;
    authMode.value = 'login';
    showAuthModal.value = true;
    return;
  }

  runDummySearch();
}

function handleFileChange(event) {
  const file = event.target.files?.[0];
  if (!file) {
    cvFile.value = null;
    return;
  }

  const allowedTypes = [
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
  ];

  if (!allowedTypes.includes(file.type)) {
    alert('Hanya file PDF atau Word (.doc, .docx) yang diperbolehkan.');
    event.target.value = '';
    cvFile.value = null;
    return;
  }

  cvFile.value = file;
}

function openAuth(mode) {
  authMode.value = mode;
  showAuthModal.value = true;
}

function handleAuthSuccess(payload) {
  isLoggedIn.value = true;
  userEmail.value = payload.email;
  showAuthModal.value = false;

  if (pendingSearch.value) {
    pendingSearch.value = false;
    runDummySearch();
  }
}
</script>

<template>
  <div class="app-root">
    <!-- HEADER -->
    <header class="app-header">
      <div class="brand">
        <div class="brand-mark">MJ</div>
        <div class="brand-text">
          <h1>My Job Agent</h1>
          <p>Bantu kamu mencari pekerjaan yang relevan, cepat dan terarah.</p>
        </div>
      </div>

      <div class="header-right">
        <button
          type="button"
          class="icon-button theme-toggle"
          @click="toggleTheme"
          :aria-label="theme === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'"
        >
          <span v-if="theme === 'dark'">🌙</span>
          <span v-else>☀️</span>
        </button>

        <button
          v-if="!isLoggedIn"
          class="btn btn-ghost"
          type="button"
          @click="openAuth('login')"
        >
          Masuk
        </button>

        <div v-else class="user-pill">
          <span class="user-avatar">{{ userEmail.charAt(0).toUpperCase() }}</span>
          <span class="user-email">{{ userEmail }}</span>
        </div>
      </div>
    </header>

    <!-- MAIN -->
    <main class="app-main">
      <section class="layout">
        <div class="hero-copy">
          <h2>Cari pekerjaan dengan CV dan kata kunci dalam satu langkah.</h2>
          <p>
            Unggah CV-mu, ketik jenis pekerjaan yang kamu minati, dan biarkan My Job Agent
            mengolahnya menjadi rekomendasi lowongan yang relevan.
          </p>
        </div>

        <div class="search-panel">
          <form @submit.prevent="handleSearchClick" class="search-form">
            <div class="form-group">
              <label for="cvInput">Upload CV</label>
              <div class="file-input-wrapper">
                <input
                  id="cvInput"
                  type="file"
                  class="file-input"
                  accept=".pdf,.doc,.docx,application/pdf,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document"
                  @change="handleFileChange"
                />
                <span class="file-display">
                  <span class="file-button">Pilih File</span>
                  <span class="file-name">
                    {{ cvFile ? cvFile.name : 'Upload CV' }}
                  </span>
                </span>
              </div>
            </div>

            <div class="form-group">
              <label for="jobQuery">Pekerjaan yang Anda cari</label>
              <input
                id="jobQuery"
                v-model="jobQuery"
                type="text"
                class="text-input"
                placeholder="Pekerjaan apa yang anda cari?"
              />
            </div>

            <button type="submit" class="btn btn-primary full-width">
              Mulai Cari Pekerjaan
            </button>
            <p class="sub-note">
              *Untuk melihat hasil, Anda perlu login ke My Job Agent.
            </p>
          </form>
        </div>
      </section>

      <!-- RESULTS -->
      <section v-if="hasSearched" class="results-section">
        <div class="results-header">
          <div>
            <h3>Top Matches</h3>
            <p>{{ jobResults.length }} found</p>
          </div>
          <span class="powered-by">Powered by dummy data</span>
        </div>

        <div class="results-list">
          <article v-for="job in jobResults" :key="job.id" class="job-card">
            <div class="job-card-main">
              <div class="job-rank">
                <span>#{{ job.rank }}</span>
              </div>

              <div class="job-info">
                <h4 class="job-title">{{ job.title }}</h4>
                <p class="job-company">{{ job.company }}</p>

                <div class="job-meta">
                  <span class="meta-item">
                    <span class="meta-icon">📍</span>{{ job.location }}
                  </span>
                  <span class="meta-separator">•</span>
                  <span class="meta-item">{{ job.employmentType }}</span>
                  <span class="meta-separator">•</span>
                  <span class="meta-chip">{{ job.workMode }}</span>
                </div>

                <p class="job-desc">
                  “{{ job.description }}”
                </p>
              </div>

              <div class="job-score-block">
                <div class="score-pill">
                  <span class="score-value">{{ job.matchScore }}%</span>
                  <span class="score-label">MATCH</span>
                </div>
                <a
                  :href="job.url"
                  target="_blank"
                  rel="noreferrer"
                  class="btn btn-outline"
                >
                  View Job
                </a>
              </div>
            </div>
          </article>
        </div>
      </section>
    </main>

    <AuthModal
      :visible="showAuthModal"
      :mode="authMode"
      @close="showAuthModal = false"
      @switch-mode="authMode = $event"
      @success="handleAuthSuccess"
    />
  </div>
</template>

<style scoped>
.app-root {
  min-height: 100vh;
  background-color: var(--bg);
  color: var(--text);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  display: flex;
  flex-direction: column;
}

/* HEADER */
.app-header {
  position: sticky;
  top: 0;
  z-index: 10;
  backdrop-filter: blur(10px);
  background-color: var(--header-bg);
  border-bottom: 1px solid var(--border-subtle);
  padding: 0.9rem 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.brand-mark {
  width: 2.25rem;
  height: 2.25rem;
  border-radius: 0.9rem;
  background-color: var(--primary-soft);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--primary-strong-text);
  font-weight: 700;
  font-size: 0.9rem;
}

.brand-text h1 {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
}
.brand-text p {
  margin: 0;
  margin-top: 0.1rem;
  font-size: 0.8rem;
  color: var(--muted);
}

.header-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.icon-button {
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  background: transparent;
  padding: 0.35rem 0.6rem;
  font-size: 0.9rem;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.theme-toggle span {
  display: inline-block;
}

.user-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  padding: 0.25rem 0.6rem 0.25rem 0.25rem;
  background-color: var(--card-bg-soft);
  font-size: 0.8rem;
}
.user-avatar {
  width: 1.6rem;
  height: 1.6rem;
  border-radius: 999px;
  background-color: var(--primary);
  color: #fff;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}
.user-email {
  max-width: 11rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* MAIN */
.app-main {
  flex: 1;
  padding: 1.75rem 1.5rem 2.5rem;
  max-width: 1040px;
  width: 100%;
  margin: 0 auto;
}

.layout {
  display: grid;
  grid-template-columns: minmax(0, 1.1fr) minmax(0, 0.9fr);
  gap: 2rem;
  align-items: flex-start;
  margin-bottom: 2rem;
}

.hero-copy h2 {
  font-size: 1.75rem;
  line-height: 1.25;
  margin-bottom: 0.6rem;
}
.hero-copy p {
  margin: 0;
  font-size: 0.95rem;
  color: var(--muted);
  max-width: 30rem;
}

/* Search panel */
.search-panel {
  background-color: var(--card-bg);
  border-radius: 1rem;
  border: 1px solid var(--border-strong);
  box-shadow: var(--card-shadow);
  padding: 1.25rem 1.25rem 1rem;
}

.search-form {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 500;
}

/* File input custom */
.file-input-wrapper {
  position: relative;
  border-radius: 0.75rem;
  border: 1px solid var(--border-subtle-strong);
  background-color: var(--input-bg);
  overflow: hidden;
}

.file-input {
  position: absolute;
  inset: 0;
  opacity: 0;
  cursor: pointer;
}

.file-display {
  display: flex;
  align-items: center;
  padding: 0.6rem 0.8rem;
  gap: 0.6rem;
}

.file-button {
  border-radius: 999px;
  border: 1px solid var(--primary);
  padding: 0.2rem 0.7rem;
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--primary);
  background-color: var(--primary-soft);
  white-space: nowrap;
}

.file-name {
  font-size: 0.86rem;
  color: var(--muted);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Text input */
.text-input {
  border-radius: 0.75rem;
  border: 1px solid var(--border-subtle-strong);
  background-color: var(--input-bg);
  padding: 0.6rem 0.8rem;
  font-size: 0.9rem;
  color: var(--text);
  outline: none;
}
.text-input::placeholder {
  color: var(--muted);
}
.text-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1px var(--primary-soft);
}

/* Buttons */
.btn {
  border-radius: 999px;
  padding: 0.6rem 1.25rem;
  font-size: 0.9rem;
  font-weight: 500;
  border: none;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  transition: background-color 0.12s ease, border-color 0.12s ease,
    box-shadow 0.12s ease, transform 0.08s ease;
}

.btn-primary {
  background-color: var(--primary);
  color: #fff;
  box-shadow: 0 8px 16px rgba(37, 99, 235, 0.4);
}
.btn-primary:hover {
  background-color: var(--primary-strong);
  transform: translateY(-1px);
}

.btn-ghost {
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  background-color: transparent;
  color: var(--text);
  padding-inline: 0.9rem;
}
.btn-ghost:hover {
  background-color: var(--card-bg-soft);
}

.btn-outline {
  border-radius: 999px;
  border: 1px solid var(--border-subtle-strong);
  background-color: transparent;
  color: var(--text);
  padding-inline: 0.95rem;
  font-size: 0.86rem;
}
.btn-outline:hover {
  border-color: var(--primary);
  color: var(--primary-strong);
}

.full-width {
  width: 100%;
}

.sub-note {
  font-size: 0.78rem;
  color: var(--muted);
}

/* RESULTS */
.results-section {
  margin-top: 0.5rem;
}

.results-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.75rem;
}

.results-header h3 {
  margin: 0;
  font-size: 1rem;
}
.results-header p {
  margin: 0.15rem 0 0;
  font-size: 0.82rem;
  color: var(--muted);
}

.powered-by {
  font-size: 0.75rem;
  color: var(--muted);
}

.results-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.job-card {
  background-color: var(--card-bg);
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle-strong);
  padding: 0.9rem 1rem;
}

.job-card-main {
  display: grid;
  grid-template-columns: auto minmax(0, 1.5fr) auto;
  column-gap: 1rem;
  row-gap: 0.35rem;
  align-items: flex-start;
}

.job-rank span {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1.9rem;
  height: 1.9rem;
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--muted-strong);
}

.job-title {
  margin: 0;
  font-size: 0.98rem;
  font-weight: 600;
}

.job-company {
  margin: 0.15rem 0 0.25rem;
  font-size: 0.85rem;
  color: var(--muted);
}

.job-meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.8rem;
  color: var(--muted-strong);
}

.meta-item {
  display: inline-flex;
  align-items: center;
  gap: 0.2rem;
}

.meta-icon {
  font-size: 0.85rem;
}

.meta-separator {
  color: var(--border-subtle-strong);
}

.meta-chip {
  border-radius: 999px;
  padding: 0.1rem 0.55rem;
  border: 1px solid var(--primary-soft-border);
  background-color: var(--primary-soft);
  color: var(--primary-strong);
  font-size: 0.78rem;
}

.job-desc {
  margin: 0.5rem 0 0;
  font-size: 0.84rem;
  color: var(--muted-strong);
}

.job-score-block {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.5rem;
}

.score-pill {
  border-radius: 0.75rem;
  padding: 0.35rem 0.7rem;
  border: 1px solid var(--border-subtle);
  background-color: var(--score-bg);
  display: inline-flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.05rem;
}

.score-value {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--score-text);
}
.score-label {
  font-size: 0.7rem;
  color: var(--muted);
}

/* Responsive */
@media (max-width: 900px) {
  .layout {
    grid-template-columns: minmax(0, 1fr);
  }

  .hero-copy {
    margin-bottom: 0.25rem;
  }
}

@media (max-width: 640px) {
  .app-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.6rem;
  }

  .app-main {
    padding-inline: 1rem;
  }

  .job-card-main {
    grid-template-columns: auto minmax(0, 1fr);
  }

  .job-score-block {
    grid-column: 1 / -1;
    flex-direction: row;
    justify-content: space-between;
    margin-top: 0.5rem;
  }
}
</style>
