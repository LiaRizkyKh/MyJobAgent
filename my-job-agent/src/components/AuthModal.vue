<!-- src/components/AuthModal.vue -->
<script setup>
import { computed, ref, watch } from 'vue';

const props = defineProps({
  visible: { type: Boolean, default: false },
  mode: { type: String, default: 'login' } // 'login' | 'signup'
});

const emit = defineEmits(['close', 'switch-mode', 'success']);

const email = ref('');
const password = ref('');
const name = ref('');

watch(
  () => props.visible,
  (val) => {
    if (val) {
      email.value = '';
      password.value = '';
      name.value = '';
    }
  }
);

const isLogin = computed(() => props.mode === 'login');

function handleSubmit() {
  if (!email.value || !password.value) {
    alert('Email dan password wajib diisi.');
    return;
  }

  if (!isLogin.value && !name.value) {
    alert('Nama wajib diisi untuk pendaftaran.');
    return;
  }

  // simulasi call API
  emit('success', { email: email.value });
}

function handleGoogleLogin() {
  alert('Simulasi login dengan Google. Nanti bisa dihubungkan ke OAuth Google.');
  emit('success', { email: 'google-user@example.com' });
}
</script>

<template>
  <Teleport to="body">
    <div v-if="visible" class="modal-backdrop" @click.self="emit('close')">
      <div class="modal">
        <div class="modal-header">
          <h3>{{ isLogin ? 'Masuk ke My Job Agent' : 'Daftar Akun My Job Agent' }}</h3>
          <button class="close-btn" type="button" @click="emit('close')">✕</button>
        </div>

        <div class="modal-body">
          <button type="button" class="btn btn-google" @click="handleGoogleLogin">
            <span class="g-icon">G</span>
            Login dengan akun Google
          </button>

          <div class="divider">
            <span>atau</span>
          </div>

          <form @submit.prevent="handleSubmit" class="auth-form">
            <div v-if="!isLogin" class="form-group">
              <label for="nameInput">Nama Lengkap</label>
              <input
                id="nameInput"
                v-model="name"
                type="text"
                placeholder="Nama lengkap"
              />
            </div>

            <div class="form-group">
              <label for="emailInput">Email</label>
              <input
                id="emailInput"
                v-model="email"
                type="email"
                placeholder="nama@email.com"
              />
            </div>

            <div class="form-group">
              <label for="passwordInput">Password</label>
              <input
                id="passwordInput"
                v-model="password"
                type="password"
                placeholder="Password"
              />
            </div>

            <button type="submit" class="btn btn-primary full-width">
              {{ isLogin ? 'Masuk' : 'Daftar' }}
            </button>
          </form>
        </div>

        <div class="modal-footer">
          <p v-if="isLogin">
            Belum punya akun?
            <button type="button" class="link-btn" @click="emit('switch-mode', 'signup')">
              Daftar di sini
            </button>
          </p>
          <p v-else>
            Sudah punya akun?
            <button type="button" class="link-btn" @click="emit('switch-mode', 'login')">
              Masuk di sini
            </button>
          </p>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  inset: 0;
  background-color: rgba(15, 23, 42, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 50;
}

.modal {
  width: 100%;
  max-width: 420px;
  background-color: var(--card-bg);
  color: var(--text);
  border-radius: 0.9rem;
  border: 1px solid var(--border-strong);
  box-shadow: var(--modal-shadow);
  padding: 1.2rem 1.2rem 0.9rem;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.75rem;
}

.modal-header h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
}

.close-btn {
  border: none;
  background: transparent;
  cursor: pointer;
  color: var(--muted);
  font-size: 1rem;
}

.modal-body {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.form-group label {
  font-size: 0.84rem;
}

.form-group input {
  border-radius: 0.6rem;
  border: 1px solid var(--border-subtle-strong);
  background-color: var(--input-bg);
  padding: 0.5rem 0.7rem;
  font-size: 0.9rem;
  color: var(--text);
  outline: none;
}

.form-group input::placeholder {
  color: var(--muted);
}

.form-group input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1px var(--primary-soft);
}

/* Google button */
.btn {
  border-radius: 999px;
  padding: 0.55rem 1.2rem;
  font-size: 0.9rem;
  font-weight: 500;
  border: none;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
}

.btn-primary {
  background-color: var(--primary);
  color: #fff;
}
.btn-primary:hover {
  background-color: var(--primary-strong);
}

.btn-google {
  width: 100%;
  background-color: #ffffff;
  color: #111827;
  border-radius: 999px;
  border: 1px solid #e5e7eb;
}
.btn-google:hover {
  background-color: #f3f4f6;
}

.g-icon {
  width: 1.4rem;
  height: 1.4rem;
  border-radius: 50%;
  border: 1px solid #d4d4d8;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
}

.divider {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 0.78rem;
  color: var(--muted);
}
.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background-color: var(--border-subtle);
}

/* Footer */
.modal-footer {
  margin-top: 0.6rem;
  font-size: 0.8rem;
  text-align: center;
  color: var(--muted);
}
.link-btn {
  border: none;
  padding: 0;
  background: none;
  color: var(--primary);
  cursor: pointer;
}
.link-btn:hover {
  text-decoration: underline;
}

.full-width {
  width: 100%;
}
</style>
