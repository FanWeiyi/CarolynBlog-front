<template>
  <div class="tech-container">
    <!-- <h1 class="title">发布博客</h1> -->

    <!-- <section class="tech-block">
      <h2>🌐 前端技术</h2>
      <ul>
        <li><strong>HTML5 + CSS3</strong>：掌握语义化标签、Flex 布局、响应式设计。</li>
        <li><strong>JavaScript</strong>：熟悉 ES6+ 语法、事件处理、DOM 操作、异步编程（Promise、async/await）。</li>
        <li><strong>Vue3 + TypeScript</strong>：使用 Composition API、ref、reactive、组件通信、Pinia 状态管理。</li>
        <li><strong>Axios</strong>：进行 HTTP 请求封装与错误处理。</li>
        <li><strong>TailwindCSS</strong>：快速构建美观 UI，提高开发效率。</li>
      </ul>
    </section>

    <section class="tech-block">
      <h2>🛠️ 后端技术</h2>
      <ul>
        <li><strong>Spring Boot</strong>：搭建 RESTful API，整合 MyBatis、Spring MVC。</li>
        <li><strong>MyBatis + MySQL</strong>：熟悉 CRUD、动态 SQL、分页查询、表关联。</li>
        <li><strong>Spring Security</strong>：实现用户认证与权限控制。</li>
        <li><strong>JWT</strong>：实现无状态登录机制。</li>
      </ul>
    </section>

    <section class="tech-block">
      <h2>📦 项目管理 & 部署</h2>
      <ul>
        <li><strong>Git</strong>：版本控制、分支管理、协作开发。</li>
        <li><strong>Vite</strong>：前端项目构建工具，配合 Vue3 使用。</li>
        <li><strong>Nginx + 阿里云服务器</strong>：部署前后端项目、配置 SSL 证书、域名绑定。</li>
      </ul>
    </section>

    <section class="tech-block">
      <h2>📚 其他技术 & 实践</h2>
      <ul>
        <li><strong>微信小程序</strong>：完成虚拟试衣间功能，接入抠图 API、合成接口。</li>
        <li><strong>AI 应用</strong>：调用百炼 API，实现图文生成功能。</li>
        <li><strong>ESP8266 + 单片机</strong>：串口通信，控制 LED 灯，实现小程序远程控制。</li>
        <li><strong>软著 & 项目汇报</strong>：具备项目总结、文档撰写、PPT 演示能力。</li>
      </ul>
    </section> -->
     <BolgEdit />

    <div class="upload">
      <h2 style="color: red;">上传文件</h2>
      <input type="file" style="color: rgb(72, 63, 63);" @change="handleFileChange" />
      <select v-model="fileType">
        <option value="image">图片</option>
        <option value="video">视频</option>
        <option value="blog">博客文档</option>
        <option value="zip">压缩包</option>
      </select>
      <button @click="uploadFile">上传</button>
      <p v-if="message">{{ message }}</p>
    </div>

    <div class="file-view">
      <h2 style="color: red;">我的文件</h2>
      <div class="tabs">
        <button @click="loadFiles('image')">图片</button>
        <button @click="loadFiles('video')">视频</button>
        <button @click="loadFiles('blog')">博客</button>
        <button @click="loadFiles('zip')">压缩包</button>
      </div>

      <div v-if="fileType === 'image'" class="image-list">
        <div v-for="file in files" :key="file.url" class="file-card">
          <img :src="getUrl(file.url)" :alt="file.name" width="200" />
          <p>{{ file.name }}</p>
          <a :href="getUrl(file.url)" download>下载</a>
        </div>
      </div>

      <div v-else-if="fileType === 'video'" class="video-list">
        <div v-for="file in files" :key="file.url" class="file-card">
          <video :src="getUrl(file.url)" controls width="400"></video>
          <p>{{ file.name }}</p>
          <a :href="getUrl(file.url)" download>下载</a>
        </div>
      </div>

      <ul v-else>
        <li v-for="file in files" :key="file.url">
          <a :href="getUrl(file.url)" target="_blank">{{ file.name }}</a>
          <a :href="getUrl(file.url)" download style="margin-left: 10px;">下载</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import BolgEdit from '@/components/bolgEdit.vue'

import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import axios from '@/api'

const authStore = useAuthStore()
const router = useRouter()

const files = ref<{ name: string; url: string }[]>([])
const selectedFile = ref<File | null>(null)
const fileType = ref('image')
const message = ref('')
const uploadUrl = ref('')

const handleLogout = () => {
  authStore.clearAuth()
  router.push('/login')
}

const handleFileChange = (e: Event) => {
  const target = e.target as HTMLInputElement
  if (target.files && target.files.length > 0) {
    selectedFile.value = target.files[0]
  }
}

const uploadFile = async () => {
  if (!selectedFile.value) {
    message.value = '请先选择文件'
    return
  }

  const formData = new FormData()
  formData.append('file', selectedFile.value)
  formData.append('userId', authStore.userId || '')
  formData.append('fileType', fileType.value)

  try {
    const res = await axios.post('/upload/userfile', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    message.value = res.data
    uploadUrl.value = '' // 上传成功后不立即预览
    loadFiles(fileType.value) // 自动刷新列表
  } catch (error) {
    console.error('上传失败', error)
    message.value = '上传失败，请检查网络或服务器'
  }
}

const loadFiles = async (type: string) => {
  fileType.value = type
  const res = await axios.get('/user/files', {
    params: { userId: authStore.userId, fileType: type }
  })
  files.value = res.data
}

const getUrl = (url: string) => {
  return url.startsWith('http') ? url : '/' + url
}
</script>

<style scoped>
.user-home {
  margin: 100px auto;
  max-width: 500px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}
.upload-section {
  margin-top: 20px;
}
/* .input[type="file"] {
  margin-bottom: 10px;
  color: rgb(72, 63, 63);
} */
.upload-section input[type="file"] {
  margin-bottom: 10px;
  color: rgb(72, 63, 63);
}
.upload {
  max-width: 400px;
  margin: 50px auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
}
.image-list,
.video-list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}
.file-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  /* color: rgb(72, 63, 63); */
}
.file-view {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  /* color: rgb(72, 63, 63); */
}

</style>


<style scoped>
.tech-container {
  max-width: 960px;
  margin: 0 auto;
  padding: 40px 16px;
}
.title {
  font-size: 2.25rem;
  font-weight: bold;
  text-align: center;
  margin-bottom: 40px;
  color: #1e293b;
}
.tech-block {
  margin-bottom: 32px;
  background: #f8fafc;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}
.tech-block h2 {
  font-size: 1.5rem;
  margin-bottom: 16px;
  color: #2563eb;
}
.tech-block ul {
  list-style-type: disc;
  padding-left: 20px;
}
.tech-block li {
  margin-bottom: 8px;
  line-height: 1.6;
}
</style>

