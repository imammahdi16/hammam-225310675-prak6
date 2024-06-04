<template>
  <div class="open">
    <div class="app-wrapper">
      <div class="app">
        <div
          class="container d-flex flex-column align-items-center justify-content-center min-vh-100"
        >
          <div class="cool-title">
            <h1 class="text-primary font-weight-bold mb-4">Artikel</h1>
          </div>

          <form
            @submit.prevent="save"
            class="w-100 max-w-md bg-white p-4 rounded shadow"
          >
            <div
              class="input-container mb-3"
              :class="{ focused: isFocusedTitle, 'has-value': form.title }"
            >
              <input
                type="text"
                v-model="form.title"
                class="form-control"
                placeholder=" "
                required
                @focus="isFocusedTitle = true"
                @blur="isFocusedTitle = !form.title && false"
              />
              <label class="form-label">Judul artikel</label>
            </div>
            <div
              class="input-container mb-3"
              :class="{ focused: isFocusedContent, 'has-value': form.content }"
            >
              <textarea
                v-model="form.content"
                class="form-control"
                placeholder=" "
                required
                @focus="isFocusedContent = true"
                @blur="isFocusedContent = !form.content && false"
              ></textarea>
              <label class="form-label">Deskripsi Artikel</label>
            </div>
            <button type="submit" class="btn btn-primary w-100 mb-3">
              Save
            </button>
          </form>
          <div class="article-container w-100 max-w-md mt-4">
            <transition-group name="list" tag="ul" class="list-group">
              <li
                v-for="article in articles"
                :key="article.id"
                class="list-group-item d-flex justify-content-between align-items-start mb-2"
              >
                <div class="data">
                  <h5>{{ article.title }}</h5>
                  <p>{{ article.content }}</p>
                </div>
                <div>
                  <button
                    @click="edit(article)"
                    class="btn btn-warning btn-sm me-2"
                  >
                    Edit
                  </button>
                  <button
                    @click="remove(article.id)"
                    class="btn btn-danger btn-sm"
                  >
                    Delete
                  </button>
                </div>
              </li>
              <div v-if="!articles.length" class="no-articles">
                No articles available.
              </div>
            </transition-group>
          </div>
          <button @click="load" class="btn btn-success mt-4">Load</button>
          <transition name="fade">
            <div v-if="errorMessage" class="alert alert-danger mt-3">
              {{ errorMessage }}
            </div>
          </transition>
          <transition name="fade">
            <div v-if="successMessage" class="alert alert-success mt-3">
              {{ successMessage }}
            </div>
          </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);
    const errorMessage = ref("");
    const successMessage = ref("");
    const isFocusedTitle = ref(false);
    const isFocusedContent = ref(false);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
        errorMessage.value = "";
      } catch (error) {
        console.error("Error loading articles:", error);
        errorMessage.value = "Gagal load articles";
        clearMessages();
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === form.id ? response.data : article
          );
          successMessage.value = "Data berhasil diubah";
        } else {
          articles.value.push(response.data);
          successMessage.value = "Data berhasil ditambahkan";
        }
        form.id = null;
        form.title = "";
        form.content = "";
        errorMessage.value = "";
      } catch (error) {
        console.error("Error saving article:", error);
        errorMessage.value = "Gagal menyimpan artikel";
      } finally {
        clearMessages();
      }
    }

    async function remove(articleId) {
      try {
        await axios.delete(`http://localhost:3000/articles/${articleId}`);
        articles.value = articles.value.filter(
          (article) => article.id !== articleId
        );
        successMessage.value = "Data berhasil dihapus";
        errorMessage.value = "";
        clearMessages();
      } catch (error) {
        console.error("Error deleting article:", error);
        errorMessage.value = "Gagal menghapus artikel";
        clearMessages();
      }
    }

    async function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function clearMessages() {
      setTimeout(() => {
        successMessage.value = "";
        errorMessage.value = "";
      }, 2000);
    }

    onMounted(load);

    return {
      form,
      articles,
      save,
      edit,
      remove,
      load,
      errorMessage,
      successMessage,
      isFocusedTitle,
      isFocusedContent,
    };
  },
};
</script>

<style>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 2;
  }
}

@keyframes zoom {
  from {
    transform: scale(1);
  }
  to {
    transform: scale(0.8);
  }
}

.app {
  max-width: 100%;
  height: 100%;
  margin: auto;
  border: 5px solid gray;
  border-radius: 20px;
  padding: 20px;
  background-color: #000;
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  text-align: center;
  color: white;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  animation: fadeIn 1s ease forwards, zoom 1s ease forwards;
  overflow: hidden;
}

.article-container {
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #007bff;
  border-radius: 10px;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.9);
}

.list-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 0;
  margin: 0;
}

.list-group-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 15px;
  border: 1px solid #ced4da;
  border-radius: 5px;
  background-color: #f8f9fa;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  word-wrap: break-word;
}

.data {
  flex-grow: 1;
}

.data p {
  max-width: 300px;
  overflow: hidden;
  text-overflow: ellipsis;
  word-wrap: break-word;
  text-align: left;
  color: #000;
}

.data h5 {
  max-width: 300px;
  margin: 0;
  color: #000;
  text-align: left;
}

.no-articles {
  color: black;
}

.cool-title {
  font-size: 2.5rem;
  font-weight: bold;
  color: #007bff;
  text-align: center;
  margin-bottom: 2rem;
  position: relative;
  overflow: hidden;
}

.form-control:focus {
  outline: none;
  box-shadow: none;
}

.input-container {
  position: relative;
  margin-bottom: 1.5rem;
}

.form-control {
  width: 100%;
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  color: #000;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  box-shadow: none;
  outline: none;
}

.form-label {
  position: absolute;
  top: 0.25rem;
  left: 0.75rem;
  transition: top 0.1s, left 0.1s, font-size 0.1s, background-color 0.1s,
    padding 0.1s, color 0.1s, transform 0.3s;
  color: #000;
}

.input-container.focused .form-label,
.input-container.has-value .form-label {
  top: -0.5rem;
  left: 1rem;
  font-size: 0.75rem;
  background-color: #fff;
  padding: 0.2em;
  color: #000;
  transform: translateX(0.5rem);
}

.list-group-item {
  animation: moveUp 0.5s ease forwards;
}

.list-enter-from {
  opacity: 0;
  transform: scale(0.6);
}

.list-enter-to {
  opacity: 1;
  transform: scale(1);
}

.list-enter-active {
  transition: all 0.4s ease;
}

.list-leave-to {
  opacity: 0;
  transform: scale(0.6);
}

.list-leave-active {
  transition: all 1s ease;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.edit-enter-from {
  opacity: 0;
  transform: scale(0.6);
}

.edit-enter-to {
  opacity: 1;
  transform: scale(1);
}

.edit-enter-active {
  transition: all 0.4s ease;
}

.edit-leave-to {
  opacity: 0;
  transform: scale(0.6);
}

.edit-leave-active {
  transition: all 1s ease;
}

.slide-in {
  animation: slideIn 0.5s ease-in-out;
  font-size: 2rem;
  color: #fff;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}
</style>
