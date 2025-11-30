<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import AuthModal from './components/AuthModal.vue';
// API endpoint - use environment variable or fallback to production URL
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'https://myjobmatch-backend-355671284742.asia-southeast2.run.app';

const cvFile = ref(null);
const cvText = ref(''); // optional CV text input
const jobQuery = ref('');

const showAuthModal = ref(false);
const authMode = ref('login');
const isLoggedIn = ref(false);
const userEmail = ref('');
const pendingSearch = ref(false);

// theme
const theme = ref('dark');

function applyTheme(value) {
  document.documentElement.setAttribute('data-theme', value);
}

/* === interaktif background === */
const appRootRef = ref(null);

function handlePointerMove(event) {
  const root = appRootRef.value;
  if (!root) return;
  const rect = root.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  root.style.setProperty('--cursor-x', `${x}px`);
  root.style.setProperty('--cursor-y', `${y}px`);
}

onMounted(() => {
  const current = document.documentElement.getAttribute('data-theme');
  theme.value = current === 'light' ? 'light' : 'dark';
  applyTheme(theme.value);

  window.addEventListener('pointermove', handlePointerMove);
});

onBeforeUnmount(() => {
  window.removeEventListener('pointermove', handlePointerMove);
});

watch(theme, (val) => {
  applyTheme(val);
});

function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark';
}

/* === STATE HASIL PENCARIAN === */

const hasSearched = ref(false);
const jobResults = ref([]); // <- array of job objects
const isLoading = ref(false);

/*
  Bentuk setiap item di jobResults:
  {
    id,
    rank,
    title,
    company,
    location,
    employmentType,   // dari work_type
    workMode,         // dari site_setting
    matchScore,       // dari match_score
    description,      // dari match_reason (fallback description)
    url
  }
*/

function mapWorkType(type) {
  switch (type) {
    case 'full_time':
      return 'Full-time';
    case 'part_time':
      return 'Part-time';
    case 'contract':
      return 'Contract';
    case 'internship':
      return 'Internship';
    default:
      return 'Other';
  }
}

function transformApiResponseToJobs(apiResponse) {
  if (!apiResponse || !Array.isArray(apiResponse.results)) return [];

  const preferredLocation =
    apiResponse.profile?.preferred_locations?.[0] || '';
  const preferredRemote =
    apiResponse.profile?.preferred_remote_modes?.[0] || '';

  return apiResponse.results.map((job, index) => {
    const location =
      job.location || preferredLocation || 'Lokasi tidak disebutkan';

    const workMode =
      job.site_setting && job.site_setting !== 'Unknown'
        ? job.site_setting
        : preferredRemote || 'Flexible';

    return {
      id: index,
      rank: index + 1,
      title: job.title,
      company: job.company,
      location,
      employmentType: mapWorkType(job.work_type),
      workMode,
      matchScore: job.match_score ?? null,
      description: job.match_reason || job.description,
      url: job.url
    };
  });
}

/* === Fetch jobs from real API with file upload === */
async function fetchJobsFromApi() {
  const formData = new FormData();
  
  // Add CV file if selected
  if (cvFile.value) {
    formData.append('cv_file', cvFile.value);
  }
  
  // Add CV text if provided (or use job query as fallback)
  const textContent = cvText.value.trim() || jobQuery.value.trim();
  if (textContent) {
    formData.append('cv_text', textContent);
  }
  
  try {
    const response = await fetch(`${API_BASE_URL}/api/search-jobs`, {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
      },
      body: formData,
    });
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.detail || `HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    jobResults.value = transformApiResponseToJobs(data);
    hasSearched.value = true;
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}

/* === HANDLER FORM & FILE === */

async function handleSearchClick() {
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

  try {
    isLoading.value = true;
    await fetchJobsFromApi();
  } catch (err) {
    console.error(err);
    alert('Gagal memuat rekomendasi pekerjaan.');
  } finally {
    isLoading.value = false;
  }
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

/* === AUTH MODAL === */

function openAuth(mode) {
  authMode.value = mode;
  showAuthModal.value = true;
}

async function handleAuthSuccess(payload) {
  isLoggedIn.value = true;
  userEmail.value = payload.email;
  showAuthModal.value = false;

  if (pendingSearch.value) {
    pendingSearch.value = false;
    // setelah login, langsung jalankan pencarian yang tertunda
    try {
      isLoading.value = true;
      await fetchJobsFromApi();
    } catch (err) {
      console.error(err);
      alert('Gagal memuat rekomendasi pekerjaan.');
    } finally {
      isLoading.value = false;
    }
  }
}
</script>


<template>
  <div class="app-root" ref="appRootRef">
    <!-- HEADER -->
    <header class="page-header">
      <div class="flex items-center justify-between">
        <!-- Brand -->
        <div class="flex items-center">
          <div
            class="flex h-9 w-9 items-center justify-center rounded-xl bg-blue-100 text-xs font-bold text-blue-900 dark:bg-blue-500/15 dark:text-blue-100"
          >
            MJA
          </div>
        </div>

        <!-- Right controls -->
        <div class="flex items-center gap-3">
          <!-- Theme toggle -->
          <!-- <button
            type="button"
            class="inline-flex h-8 w-8 items-center justify-center rounded-full border border-slate-200 bg-white text-sm shadow-sm hover:bg-slate-50 dark:border-slate-600 dark:bg-slate-800 dark:hover:bg-slate-700"
            @click="toggleTheme"
            :aria-label="theme === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'"
          >
            <span v-if="theme === 'dark'">🌙</span>
            <span v-else>☀️</span>
          </button> -->

          <!-- Login / user pill -->
          <!-- <button
            v-if="!isLoggedIn"
            type="button"
            class="inline-flex items-center rounded-full border border-slate-300 px-4 py-1.5 text-sm font-medium text-slate-700 hover:bg-slate-50 dark:border-slate-600 dark:text-slate-100 dark:hover:bg-slate-800"
            @click="openAuth('login')"
          >
            Masuk
          </button> -->

          <!-- <div
            v-else
            class="flex items-center gap-2 rounded-full border border-slate-300 bg-white px-2 py-1 text-xs text-slate-700 shadow-sm dark:border-slate-600 dark:bg-slate-900 dark:text-slate-100"
          >
            <span
              class="flex h-7 w-7 items-center justify-center rounded-full bg-blue-600 text-xs font-semibold text-white"
            >
              {{ userEmail.charAt(0).toUpperCase() }}
            </span>
            <span class="max-w-[10rem] truncate">
              {{ userEmail }}
            </span>
          </div> -->
        </div>
      </div>
    </header>

    <!-- MAIN -->
    <main class="app-main">
      <!-- HERO + FORM di tengah, satu kolom -->
      <section class="hero-stack">
        <div class="hero-block">
          <!-- tagline yang dicoret hitam di screenshot dihapus -->
          <h2 class="hero-title">
            Cari pekerjaan dengan CV dan kata kunci dalam satu langkah.
          </h2>
          <p class="hero-sub">
            Unggah CV-mu, ketik jenis pekerjaan yang kamu minati, dan biarkan My Job Agent
            mengolahnya menjadi rekomendasi lowongan yang relevan.
          </p>
        </div>

        <div class="search-panel">
          <form @submit.prevent="handleSearchClick" class="search-form">
            <!-- Upload CV -->
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
                  <!-- <span class="file-button">Pilih File</span> -->
                  <span class="file-name">
                    {{ cvFile ? cvFile.name : 'Pilih file' }}
                  </span>

                </span>
              </div>
            </div>

            <!-- Job query -->
            <!-- <div class="form-group">
              <label for="jobQuery">Pekerjaan yang Anda cari</label>
              <input
                id="jobQuery"
                v-model="jobQuery"
                type="text"
                class="text-input bg-[#f7ffff]"
                placeholder="Pekerjaan apa yang anda cari?"
              />
            </div> -->

            <button type="submit" class="btn btn-primary full-width mt-3">
              Mulai Cari Pekerjaan
            </button>
            <p class="sub-note">
              *Untuk menyimpan hasil, Anda perlu login ke My Job Agent.
            </p>
          </form>
        </div>

      </section>

      <!-- RESULTS -->
      <section
        v-if="hasSearched"
        class="results-section"
      >
        <div class="results-header">
          <div>
            <h3>Top Matches</h3>
            <p>{{ jobResults.length }} found</p>
          </div>
          <span class="powered-by">Powered by My Job Agent</span>
        </div>

        <div class="results-list">
          <article
            v-for="job in jobResults"
            :key="job.id"
            class="job-card"
          >
            <!-- header row -->
            <div class="job-card-header">
              <!-- rank bulat -->
              <div class="job-rank">
                <span>#{{ job.rank }}</span>
              </div>

              <!-- title + company + meta -->
              <div class="job-main">
                <h4 class="job-title">
                  {{ job.title }}
                </h4>
                <p class="job-company">
                  {{ job.company }}
                </p>

                <div class="job-meta">
                  <span
                    v-if="job.location"
                    class="meta-item"
                  >
                    <span class="meta-icon">📍</span>
                    <span>{{ job.location }}</span>
                  </span>

                  <span
                    v-if="job.location && job.employmentType"
                    class="meta-separator"
                  >
                    •
                  </span>

                  <span
                    v-if="job.employmentType"
                    class="meta-item"
                  >
                    {{ job.employmentType }}
                  </span>

                  <span
                    v-if="job.workMode"
                    class="meta-chip"
                  >
                    {{ job.workMode }}
                  </span>
                </div>
              </div>

              <!-- score + button -->
              <div class="job-side">
                <div
                  v-if="job.matchScore !== null && job.matchScore !== undefined"
                  class="score-pill"
                >
                  <span class="score-value">
                    {{ job.matchScore }}%
                  </span>
                  <span class="score-label">
                    MATCH
                  </span>
                </div>

                <a
                  :href="job.url"
                  target="_blank"
                  rel="noreferrer"
                  class="btn btn-outline view-job-btn"
                >
                  View Job
                  <span aria-hidden="true">↗</span>
                </a>
              </div>
            </div>

            <!-- description / match_reason -->
            <div
              v-if="job.description"
              class="job-desc-box"
            >
              <div class="job-desc-icon">
                ✓
              </div>
              <p class="job-desc-text">
                “{{ job.description }}”
              </p>
            </div>
          </article>
        </div>
      </section>

    </main>

    <!-- MODAL AUTH -->
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
  position: relative;
  overflow: hidden;
}

/* background interaktif yang mengikuti cursor */
.app-root::before {
  content: '';
  position: absolute;
  inset: -200px;
  background: radial-gradient(
    circle at var(--cursor-x, 50%) var(--cursor-y, 30%),
    rgba(37, 99, 235, 0.16),
    transparent 60%
  );
  pointer-events: none;
  z-index: 0;
}

.app-root::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
      to bottom,
      rgba(15, 23, 42, 0.04),
      transparent 30%,
      rgba(15, 23, 42, 0.03)
    );
  pointer-events: none;
  z-index: 0;
}

/* semua konten di atas layer background */
.app-root > * {
  position: relative;
  z-index: 1;
}

/* HEADER WRAPPER (padding 1 halaman) */
.page-header {
  padding: 1.25rem 2.5rem 0;
}

/* MAIN: hero di tengah, dengan padding 1 halaman */
.app-main {
  flex: 1;
  padding: 2rem 2.5rem 2.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* wrapper hero + form */
.hero-stack {
  width: 100%;
  max-width: 720px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  text-align: center;
  gap: 1.5rem;
  margin-top: min(8vh, 3rem);
}

/* hero text */
.hero-block {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.hero-title {
  margin: 0;
  font-size: 1.9rem;
  line-height: 1.25;
}

.hero-sub {
  margin: 0;
  font-size: 0.95rem;
  color: var(--muted);
}

/* search panel di bawah hero text */
.search-panel {
  background-color: var(--card-bg);
  border-radius: 1.5rem;
  border: 1px solid var(--border-strong);
  box-shadow: var(--card-shadow);
  padding: 1.5rem 1.5rem 1.25rem;
  text-align: left;
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
/* file input */
.file-input-wrapper {
  position: relative;
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle-strong);
  background-color: #f7ffff !important;
  overflow: hidden;
  box-shadow: 0 18px 40px rgba(37, 99, 235, 0.16); /* sama feel dengan button */
}

.file-input {
  position: absolute;
  inset: 0;
  opacity: 0;
  cursor: pointer;
  background-color: #f7ffff !important;
}

/* sekarang rata kiri & style mirip text-input */
.file-display {
  display: flex;
  align-items: center;
  justify-content: flex-start;         /* <— rata kiri */
  width: 100%;
  padding: 0.6rem 0.8rem;
  gap: 0.6rem;
  background-color: var(--input-bg);   /* sama seperti text-input */
}

.file-button {
  border-radius: 999px;
  border: 1px solid var(--primary);
  padding: 0.2rem 0.9rem;
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

/* text input: disamakan background & feel-nya */
.text-input {
  border-radius: 0.9rem;
  border: 1px solid var(--border-subtle-strong);
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


/* buttons */
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

/* primary: putih dengan shadow biru, text hitam */
.btn-primary {
  background-color: #ffffff;
  color: #111827; /* text hitam */
  border: 1px solid rgba(37, 99, 235, 0.4);
  box-shadow: 0 18px 45px rgba(37, 99, 235, 0.25);
}
.btn-primary:hover {
  transform: translateY(-1px);
}

/* ghost & outline tetap */
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
/* RESULTS – layout utama */
.results-section {
  width: 100%;
  max-width: 1040px;
  margin: 2.75rem auto 0;
  padding-bottom: 2.5rem;
}

.results-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 0.9rem;
}

.results-header h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
}

.results-header p {
  margin: 0.15rem 0 0;
  font-size: 0.82rem;
  color: var(--muted);
}

.powered-by {
  font-size: 0.78rem;
  color: var(--muted);
}

.results-list {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
}

/* Kartu job mirip screenshot */
.job-card {
  background-color: var(--card-bg);
  border-radius: 1.1rem;
  border: 1px solid var(--border-subtle);
  box-shadow: 0 18px 35px rgba(15, 23, 42, 0.08);
  padding: 1.1rem 1.3rem 1rem;
}

/* header row di dalam kartu */
.job-card-header {
  display: grid;
  grid-template-columns: auto minmax(0, 1.5fr) auto;
  column-gap: 1.1rem;
  row-gap: 0.35rem;
  align-items: flex-start;
}

/* rank bulat */
.job-rank span {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.1rem;
  height: 2.1rem;
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  font-size: 0.86rem;
  font-weight: 600;
  color: var(--primary);
  background-color: var(--card-bg-soft);
}

/* info utama */
.job-main {
  min-width: 0;
}

.job-title {
  margin: 0;
  font-size: 0.98rem;
  font-weight: 600;
}

.job-company {
  margin: 0.15rem 0 0.25rem;
  font-size: 0.86rem;
  color: var(--muted);
}

/* baris meta: lokasi, jenis kerja, site setting */
.job-meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.8rem;
  color: var(--muted-strong);
}

.meta-item {
  display: inline-flex;
  align-items: center;
  gap: 0.2rem;
}

.meta-icon {
  font-size: 0.9rem;
}

.meta-separator {
  color: var(--border-subtle-strong);
}

.meta-chip {
  border-radius: 999px;
  padding: 0.08rem 0.55rem;
  border: 1px solid var(--primary-soft-border);
  background-color: var(--primary-soft);
  color: var(--primary-strong);
  font-size: 0.78rem;
  font-weight: 500;
}

/* kolom kanan: score + tombol */
.job-side {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.5rem;
}

.score-pill {
  border-radius: 0.9rem;
  padding: 0.4rem 0.75rem;
  border: 1px solid rgba(16, 185, 129, 0.25);
  background-color: rgba(16, 185, 129, 0.08);
  display: inline-flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.05rem;
}

.score-value {
  font-size: 0.95rem;
  font-weight: 600;
  color: #059669; /* hijau ke arah tailwind emerald */
}

.score-label {
  font-size: 0.7rem;
  font-weight: 500;
  color: rgba(5, 150, 105, 0.9);
}

.view-job-btn {
  padding-inline: 1rem;
  font-size: 0.84rem;
}

/* box deskripsi di bawah (match_reason/description) */
.job-desc-box {
  margin-top: 0.9rem;
  border-radius: 0.75rem;
  background-color: var(--card-bg-soft);
  border: 1px solid var(--border-subtle);
  padding: 0.6rem 0.75rem;
  display: flex;
  align-items: flex-start;
  gap: 0.6rem;
}

.job-desc-icon {
  width: 1.4rem;
  height: 1.4rem;
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  color: var(--muted-strong);
  flex-shrink: 0;
}

.job-desc-text {
  margin: 0;
  font-size: 0.82rem;
  color: var(--muted-strong);
}

/* responsive: kartu stack saat layar kecil */
@media (max-width: 640px) {
  .job-card-header {
    grid-template-columns: auto minmax(0, 1fr);
  }

  .job-side {
    grid-column: 1 / -1;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin-top: 0.5rem;
  }

  .view-job-btn {
    padding-inline: 0.9rem;
  }
}


/* Responsive */
@media (max-width: 900px) {
  .page-header {
    padding: 1rem 1.5rem 0;
  }

  .app-main {
    padding: 1.75rem 1.5rem 2rem;
  }
}

@media (max-width: 640px) {
  .page-header {
    padding: 1rem 1rem 0;
  }

  .app-main {
    padding: 1.5rem 1rem 2rem;
  }

  .search-panel {
    padding-inline: 1.1rem;
  }

  /* .job-card-main {
    grid-template-columns: auto minmax(0, 1fr);
  }

  .job-score-block {
    grid-column: 1 / -1;
    flex-direction: row;
    justify-content: space-between;
    margin-top: 0.5rem;
  } */
}
</style>
