<template>
    <div>
      <input v-model="searchQuery" @input="fetchData" placeholder="Search..." />
      <select v-model="rowsPerPage" @change="fetchData">
        <option value="10">10</option>
        <option value="15">15</option>
        <option value="20">20</option>
        <option value="all">All</option>
      </select>
      <table>
        <thead>
          <tr>
            <th @click="sort('id')">ID</th>
            <th>Name</th>
            <th @click="sort('email')">Email</th>
            <th>Body</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="comment in paginatedComments" :key="comment.id">
            <td>{{ comment.id }}</td>
            <td>{{ comment.name }}</td>
            <td>{{ comment.email }}</td>
            <td>{{ comment.body }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      <div v-if="loading">Loading...</div>
    </div>
  </template>
  
  <script>
  import { ref, computed, onMounted, watch } from 'vue';
  import axios from 'axios';
  
  export default {
    setup() {
      const comments = ref([]);
      const loading = ref(false);
      const searchQuery = ref('');
      const rowsPerPage = ref('10');
      const currentPage = ref(1);
      const sortKey = ref('');
      const sortAsc = ref(true);
  
      const fetchData = async () => {
        loading.value = true;
        try {
          const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
          comments.value = response.data;
        } finally {
          loading.value = false;
        }
      };
  
      const sortedComments = computed(() => {
        const data = comments.value.filter(comment =>
          comment.body.toLowerCase().includes(searchQuery.value.toLowerCase())
        );
        if (sortKey.value) {
          data.sort((a, b) => {
            if (a[sortKey.value] < b[sortKey.value]) return sortAsc.value ? -1 : 1;
            if (a[sortKey.value] > b[sortKey.value]) return sortAsc.value ? 1 : -1;
            return 0;
          });
        }
        return data;
      });
  
      const paginatedComments = computed(() => {
        const start = (currentPage.value - 1) * rowsPerPage.value;
        const end = start + rowsPerPage.value;
        return rowsPerPage.value === 'all'
          ? sortedComments.value
          : sortedComments.value.slice(start, end);
      });
  
      const totalPages = computed(() =>
        rowsPerPage.value === 'all' ? 1 : Math.ceil(sortedComments.value.length / rowsPerPage.value)
      );
  
      const sort = (key) => {
        if (sortKey.value === key) {
          sortAsc.value = !sortAsc.value;
        } else {
          sortKey.value = key;
          sortAsc.value = true;
        }
      };
  
      const prevPage = () => {
        if (currentPage.value > 1) currentPage.value--;
      };
  
      const nextPage = () => {
        if (currentPage.value < totalPages.value) currentPage.value++;
      };
  
      onMounted(fetchData);
      watch([searchQuery, rowsPerPage], fetchData);
  
      return {
        comments,
        loading,
        searchQuery,
        rowsPerPage,
        currentPage,
        sortKey,
        sortAsc,
        sortedComments,
        paginatedComments,
        totalPages,
        fetchData,
        sort,
        prevPage,
        nextPage,
      };
    },
  };
  </script>
  
  <style>
  /* Add your styles here */
  </style>
  