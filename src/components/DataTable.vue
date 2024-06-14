<template>
  <div>
    <input v-model="searchText" @input="debounceSearch" placeholder="Search comments" />
    <select v-model="itemsPerPage" @change="updateTable">
      <option v-for="option in perPageOptions" :key="option" :value="option">{{ option }}</option>
    </select>
    <table>
      <thead>
        <tr>
          <th @click="sortBy('id')">ID</th>
          <th @click="sortBy('email')">Email</th>
          <th>Comment</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="comment in paginatedComments" :key="comment.id">
          <td>{{ comment.id }}</td>
          <td>{{ comment.email }}</td>
          <td>{{ comment.body }}</td>
          <td>
            <button @click="editComment(comment.id)">Edit</button>
            <button @click="deleteComment(comment.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div>
      <button @click="changePage(page)" v-for="page in totalPages" :key="page">{{ page }}</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

export default {
  name: 'DataTable',
  setup() {
    const comments = ref([]);
    const searchText = ref('');
    const currentPage = ref(1);
    const itemsPerPage = ref(10);
    const sortDirection = ref({ id: 'asc', email: 'asc' });
    const loading = ref(false);

    const perPageOptions = [10, 15, 20, 'all'];

    const fetchData = async () => {
      loading.value = true;
      const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
      comments.value = response.data;
      loading.value = false;
      updateTable();
    };

    const totalPages = computed(() => {
      return Math.ceil(filteredComments.value.length / itemsPerPage.value);
    });

    const filteredComments = computed(() => {
      return comments.value.filter(comment =>
        comment.body.includes(searchText.value) || 
        comment.email.includes(searchText.value)
      );
    });

    const paginatedComments = computed(() => {
      if (itemsPerPage.value === 'all') {
        return filteredComments.value;
      }
      const start = (currentPage.value - 1) * itemsPerPage.value;
      const end = start + parseInt(itemsPerPage.value);
      return filteredComments.value.slice(start, end);
    });

    const changePage = (page) => {
      currentPage.value = page;
    };

    const updateTable = () => {
      currentPage.value = 1;
    };

    const debounceSearch = debounce(() => {
      updateTable();
    }, 300);

    const sortBy = (column) => {
      sortDirection.value[column] = sortDirection.value[column] === 'asc' ? 'desc' : 'asc';
      comments.value.sort((a, b) => {
        if (a[column] < b[column]) return sortDirection.value[column] === 'asc' ? -1 : 1;
        if (a[column] > b[column]) return sortDirection.value[column] === 'asc' ? 1 : -1;
        return 0;
      });
      updateTable();
    };

    const editComment = (id) => {
      alert(`Edit comment with ID: ${id}`);
    };

    const deleteComment = (id) => {
      comments.value = comments.value.filter(comment => comment.id !== id);
      updateTable();
    };

    onMounted(fetchData);

    return {
      comments,
      searchText,
      currentPage,
      itemsPerPage,
      sortDirection,
      paginatedComments,
      totalPages,
      perPageOptions,
      changePage,
      updateTable,
      debounceSearch,
      sortBy,
      editComment,
      deleteComment,
      loading,
    };
  }
};

function debounce(func, wait) {
  let timeout;
  return function(...args) {
    const context = this;
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(context, args), wait);
  };
}
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}
th, td {
  padding: 8px 12px;
  border: 1px solid #ddd;
  text-align: left;
}
th {
  cursor: pointer;
}
</style>
