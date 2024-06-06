<template>
  <!-- Main container for the data table and its controls -->
  <div class="data-table-container">
    <!-- Controls for search and rows per page selection -->
    <div class="controls">
      <!-- Search input field, bound to searchQuery and triggers handleSearch on input -->
      <input v-model="searchQuery" @input="handleSearch" placeholder="Search..." class="search-input" />
      <!-- Select dropdown for rows per page, bound to rowsPerPage and triggers handleRowsPerPageChange on change -->
      <select v-model="rowsPerPage" @change="handleRowsPerPageChange" class="rows-select">
        <option value="10">10</option>
        <option value="15">15</option>
        <option value="20">20</option>
        <option value="all">All</option>
      </select>
    </div>
    <!-- Data table to display comments -->
    <table class="data-table">
      <thead>
        <tr>
          <!-- Sortable column headers for ID and Email -->
          <th @click="sort('id')">ID</th>
          <th>Name</th>
          <th @click="sort('email')">Email</th>
          <th>Body</th>
        </tr>
      </thead>
      <tbody>
        <!-- Render paginated comments -->
        <tr v-for="comment in paginatedComments" :key="comment.id">
          <td>{{ comment.id }}</td>
          <td>{{ comment.name }}</td>
          <td>{{ comment.email }}</td>
          <td>{{ comment.body }}</td>
        </tr>
      </tbody>
    </table>
    <!-- Pagination controls -->
    <div class="pagination-controls">
      <!-- Previous page button, disabled on the first page -->
      <button @click="prevPage" :disabled="currentPage === 1" class="pagination-button">Previous</button>
      <!-- Next page button, disabled on the last page -->
      <button @click="nextPage" :disabled="currentPage === totalPages" class="pagination-button">Next</button>
    </div>
    <!-- Loading indicator -->
    <div v-if="loading" class="loading">Loading...</div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const comments = ref([]); // Holds the comments data
    const loading = ref(false); // Indicates if data is being fetched
    const searchQuery = ref(''); // Holds the search query
    const rowsPerPage = ref('10'); // Holds the number of rows to display per page
    const currentPage = ref(1); // Current page number
    const sortKey = ref(''); // Key to sort by
    const sortAsc = ref(true); // Indicates if sorting is ascending

    // Function to fetch data from the API
    const fetchData = async () => {
      loading.value = true;
      try {
        const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
        comments.value = response.data;
      } finally {
        loading.value = false;
      }
    };

    // Computed property to get sorted and filtered comments
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

    // Computed property to get paginated comments
    const paginatedComments = computed(() => {
      if (rowsPerPage.value === 'all') {
        return sortedComments.value;
      }
      const start = (currentPage.value - 1) * parseInt(rowsPerPage.value, 10);
      const end = start + parseInt(rowsPerPage.value, 10);
      return sortedComments.value.slice(start, end);
    });

    // Computed property to get total number of pages
    const totalPages = computed(() => {
      if (rowsPerPage.value === 'all') {
        return 1;
      }
      return Math.ceil(sortedComments.value.length / parseInt(rowsPerPage.value, 10));
    });

    // Function to sort by a given key
    const sort = (key) => {
      if (sortKey.value === key) {
        sortAsc.value = !sortAsc.value;
      } else {
        sortKey.value = key;
        sortAsc.value = true;
      }
    };

    // Function to go to the previous page
    const prevPage = () => {
      if (currentPage.value > 1) currentPage.value--;
    };

    // Function to go to the next page
    const nextPage = () => {
      if (currentPage.value < totalPages.value) currentPage.value++;
    };

    // Function to handle search input
    const handleSearch = () => {
      currentPage.value = 1;
      fetchData();
    };

    // Function to handle change in rows per page
    const handleRowsPerPageChange = () => {
      currentPage.value = 1;
      fetchData();
    };

    // Fetch data when component is mounted
    onMounted(fetchData);

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
      handleSearch,
      handleRowsPerPageChange,
    };
  },
};
</script>

<style scoped>
/* Container for the data table */
.data-table-container {
  font-family: Arial, sans-serif;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

/* Controls section */
.controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

/* Styling for search input and rows per page select dropdown */
.search-input,
.rows-select {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Styling for the data table */
.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.data-table th,
.data-table td {
  padding: 12px;
  border: 1px solid #ccc;
  text-align: left;
}

/* Styling for table headers */
.data-table th {
  background-color: #f2f2f2;
  cursor: pointer;
}

/* Styling for alternating table rows */
.data-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

/* Pagination controls section */
.pagination-controls {
  display: flex;
  justify-content: space-between;
}

/* Styling for pagination buttons */
.pagination-button {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

/* Disabled state for pagination buttons */
.pagination-button:disabled {
  background-color: #ccc;
}

/* Loading indicator styling */
.loading {
  text-align: center;
  padding: 20px;
  font-size: 18px;
}
</style>
