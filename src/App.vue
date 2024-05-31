<template>
  <div class="form-container">
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        <strong>{{ article.title }}</strong
        ><br />
        <p>{{ article.content }}</p>
        <br />
        <button @click="edit(article)">Edit</button>
        <button @click="remove(article.id)">Delete</button>
      </li>
    </ul>
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    })

    const articles = ref([])

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles')
        articles.value = response.data
      } catch (error) {
        console.error('Error Loading Articles:', error)
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles'
        const method = form.id ? 'put' : 'post'
        const response = await axios[method](url, { title: form.title, content: form.content })

        if (form.id) {
          // Update existing article
          articles.value = articles.value.map((article) =>
            article.id === form.id ? response.data : article
          )
        } else {
          // Add new article
          articles.value.push(response.data)
        }

        // Reset form
        form.id = null
        form.title = ''
        form.content = ''
      } catch (error) {
        console.error('Error Saving Article:', error)
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`)
        articles.value = articles.value.filter((article) => article.id !== id)
      } catch (error) {
        console.error('Error Deleting Article:', error)
      }
    }

    function edit(article) {
      form.id = article.id
      form.title = article.title
      form.content = article.content
    }

    onMounted(load)

    return { form, articles, save, edit, remove, load }
  }
}
</script>
