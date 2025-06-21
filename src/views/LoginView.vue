<template>
  <div class="login">
    <h2>登录</h2>
    <form @submit.prevent="handleLogin">
      <input v-model="username" placeholder="用户名" class="border p-2 w-full mb-4" />
      <input v-model="password" type="password" placeholder="密码" class="border p-2 w-full mb-4" />
      <button type="submit">登录</button>
    </form>
    <p v-if="error">{{ error }}</p>
    <router-link to="/register">没有账号？去注册</router-link>

    <!-- ✅ 显示 token（从 pinia 中） -->
    <p v-if="authStore.token" style="color: rgb(72, 63, 63);">当前 Token：{{ authStore.token }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from '@/api'
import { isAxiosError } from 'axios'
import { useAuthStore } from '@/stores/auth'

const username = ref('')
const password = ref('')
const error = ref('')
const router = useRouter()
const authStore = useAuthStore()

const handleLogin = async () => {
  try {
    const res = await axios.post<{ token: string; userId: number }>('/login', {
      username: username.value,
      password: password.value,
    })
    authStore.setAuth(res.data.token, res.data.userId)  // ✅ 设置 token + userId
    router.push('/user')
  } catch (err: unknown) {
    if (isAxiosError(err)) {
      error.value = err.response?.data?.msg || '账号或密码错误'
    } else {
      error.value = '网络异常或未知错误'
    }
  }
}
</script>

<style scoped>
.login {
  /* max-width: 300px;
  margin: 100px auto; */
  max-width: 500px;
  margin: 100px auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
  /* color: rgb(72, 63, 63) */
}
</style>

<style scoped>
.p-6 {
  background-color: #ffffff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  border-radius: 12px;
  margin-top: 2rem;
}
h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #1f2937;
  text-align: center;
  margin-bottom: 1.5rem;
}
input,
textarea {
  border: 1px solid #d1d5db;
  border-radius: 0.5rem;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  width: 100%;
  transition: border-color 0.2s, box-shadow 0.2s;
}
input:focus,
textarea:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.3);
}
.ql-editor {
  min-height: 200px;
  padding: 1rem;
  font-size: 1rem;
  line-height: 1.75;
  color: #374151;
}
.ql-container {
  border-radius: 0.5rem;
}
.text-green-600 {
  word-break: break-all;
  font-size: 0.875rem;
}
button {
  margin-top: 1.5rem;
  /* margin: auto; */
  background-color: #e959f9;
  padding: 0.6rem 1.5rem;
  font-size: 1rem;
  font-weight: 500;
  border-radius: 0.5rem;
  transition: all 0.2s;
}
button:hover {
  background-color: #d22e99;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(59, 130, 246, 0.3);
}
.bg-red-500 {
  background-color: #ef4444;
}
.bg-red-500:hover {
  background-color: #dc2626;
}
</style>

