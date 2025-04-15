<template>
  <div class="user-list-container">
    <main class="main-content">
      <!-- This is for the Profile Section -->
      <transition name="fade">
        <div v-if="selectedUser" class="profile-section">
          <div class="profile-header">
            <div class="profile-header-content">
              <button @click="clearSelectedUser" class="back-button">
                <i class="back-icon"></i>
              </button>
              
              <div class="profile-info">
                <img :src="selectedUser.picture.large" alt="Profile Picture" class="profile-picture" />
                <h2 class="user-name">{{ selectedUser.name.first }} {{ selectedUser.name.last }}</h2>
                <p class="last-online">Last online: 2 days ago</p>
                
                <div class="profile-actions">
                  <button class="action-button">
                    <i class="message-icon"></i>
                    Send Message
                  </button>
                  <button class="action-button">
                    <i class="add-friend-icon"></i>
                    Add Friend
                  </button>
                </div>
              </div>
            </div>
          </div>
          
          <div class="user-details">
            <div class="detail-header">
              <div class="detail-cell">Date</div>
              <div class="detail-cell">Name</div>
              <div class="detail-cell">Gender</div>
              <div class="detail-cell">Country</div>
              <div class="detail-cell">Email</div>
            </div>
            
            <div class="detail-row">
              <div class="detail-cell">{{ formatDate(selectedUser.registered.date) }}</div>
              <div class="detail-cell">{{ selectedUser.name.first }} {{ selectedUser.name.last }}</div>
              <div class="detail-cell capitalize">{{ selectedUser.gender }}</div>
              <div class="detail-cell">{{ selectedUser.location.country }}</div>
              <div class="detail-cell">{{ selectedUser.email }}</div>
            </div>
          </div>
        </div>
      </transition>

      <!-- This is for Searching and Controls -->
      <div class="controls-section">
        <div class="search-container">
          <i class="search-icon"></i>
          <input
            v-model="searchQuery"
            type="text"
            class="search-input"
            placeholder="Search users..."
          />
        </div>
        
        <div class="items-per-page">
          <label class="items-label">Show:</label>
          <select v-model="itemsPerPage" class="items-select">
            <option>5</option>
            <option>10</option>
            <option>20</option>
          </select>
        </div>
      </div>

      <!-- This is for Users Table -->
      <div class="users-table">
        <div v-if="loading" class="loading-container">
          <div class="loader"></div>
        </div>
        
        <template v-else>
          <div class="table-header">
            <div class="table-cell">Date</div>
            <div class="table-cell">Name</div>
            <div class="table-cell">Gender</div>
            <div class="table-cell">Country</div>
            <div class="table-cell">Email</div>
          </div>
          
          <transition-group name="slide-fade">
            <div 
              v-for="user in paginatedUsers" 
              :key="user.login.uuid"
              class="table-row"
              @click="selectUser(user)"
            >
              <div class="table-cell">{{ formatDate(user.registered.date) }}</div>
              <div class="table-cell">{{ user.name.first }} {{ user.name.last }}</div>
              <div class="table-cell capitalize">{{ user.gender }}</div>
              <div class="table-cell">{{ user.location.country }}</div>
              <div class="table-cell">{{ user.email }}</div>
            </div>
          </transition-group>
          
          <div v-if="filteredUsers.length === 0" class="no-results">
            No users found matching your search criteria.
          </div>

          <!-- Pagination -->
          <div class="pagination-container">
            <div class="pagination-info">
              <p class="results-text">
                Showing
                <span class="results-number">{{ startIndex + 1 }}</span>
                to
                <span class="results-number">{{ Math.min(endIndex, filteredUsers.length) }}</span>
                of
                <span class="results-number">{{ filteredUsers.length }}</span>
                results
              </p>
            </div>
            <div class="pagination-controls">
              <button
                @click="prevPage"
                :disabled="currentPage === 1"
                class="pagination-button prev"
                :class="{'disabled': currentPage === 1}"
              >
                <span class="sr-only">Prev</span>
                <i class="prev-icon"></i>
              </button>
              
              <template v-for="page in displayedPages" :key="page">
                <button
                  v-if="page !== '...'"
                  @click="currentPage = page"
                  class="pagination-page"
                  :class="{'active': currentPage === page}"
                >
                  {{ page }}
                </button>
                <span
                  v-else
                  class="pagination-ellipsis"
                >
                  ...
                </span>
              </template>
              
              <button
                @click="nextPage"
                :disabled="currentPage === totalPages"
                class="pagination-button next"
                :class="{'disabled': currentPage === totalPages}"
              >
                <span class="sr-only">Next</span>
                <i class="next-icon"></i>
              </button>
            </div>
          </div>
        </template>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue';
import axios from 'axios';

export default {
  name: 'UserList',
  setup() {
    const users = ref([]);
    const loading = ref(true);
    const currentPage = ref(1);
    const itemsPerPage = ref(10);
    const searchQuery = ref('');
    const selectedUser = ref(null);

    // Fetch users from API
    const fetchUsers = async () => {
      loading.value = true;
      try {
        const response = await axios.get('https://randomuser.me/api/?results=20');
        users.value = response.data.results;
      } catch (error) {
        console.error('Error fetching user data:', error);
      } finally {
        loading.value = false;
      }
    };

    // Filter users based on search query
    const filteredUsers = computed(() => {
      if (!searchQuery.value) return users.value;
      
      const query = searchQuery.value.toLowerCase();
      return users.value.filter(user => {
        const fullName = `${user.name.first} ${user.name.last}`.toLowerCase();
        const email = user.email.toLowerCase();
        const country = user.location.country.toLowerCase();
        
        return fullName.includes(query) || 
               email.includes(query) || 
               country.includes(query);
      });
    });

    // Calculate pagination values
    const totalPages = computed(() => {
      return Math.ceil(filteredUsers.value.length / itemsPerPage.value);
    });

    const startIndex = computed(() => {
      return (currentPage.value - 1) * itemsPerPage.value;
    });

    const endIndex = computed(() => {
      return startIndex.value + itemsPerPage.value;
    });

    const paginatedUsers = computed(() => {
      return filteredUsers.value.slice(startIndex.value, endIndex.value);
    });

    // Calculate which pages to display in pagination
    const displayedPages = computed(() => {
      if (totalPages.value <= 7) {
        return Array.from({ length: totalPages.value }, (_, i) => i + 1);
      }
      
      let pages = [];
      
      // Always show first page
      pages.push(1);
      
      // Show ellipsis if needed
      if (currentPage.value > 3) {
        pages.push('...');
      }
      
      // Calculate start and end of page range around current page
      let start = Math.max(2, currentPage.value - 1);
      let end = Math.min(totalPages.value - 1, currentPage.value + 1);
      
      // Add pages in range
      for (let i = start; i <= end; i++) {
        pages.push(i);
      }
      
      // Show ellipsis if needed
      if (currentPage.value < totalPages.value - 2) {
        pages.push('...');
      }
      
      // Always show last page
      if (totalPages.value > 1) {
        pages.push(totalPages.value);
      }
      
      return pages;
    });

    // Pagination methods
    const nextPage = () => {
      if (currentPage.value < totalPages.value) {
        currentPage.value++;
      }
    };

    const prevPage = () => {
      if (currentPage.value > 1) {
        currentPage.value--;
      }
    };

    // Format date
    const formatDate = (dateString) => {
      const date = new Date(dateString);
      return `${date.getDate()} ${date.toLocaleString('default', { month: 'short' })} ${date.getFullYear()}`;
    };

    // Select user to view details
    const selectUser = (user) => {
      selectedUser.value = user;
    };

    // Clear selected user
    const clearSelectedUser = () => {
      selectedUser.value = null;
    };

    // Reset to first page when search query or items per page changes
    watch([searchQuery, itemsPerPage], () => {
      currentPage.value = 1;
    });

    // Fetch users on component mount
    onMounted(() => {
      fetchUsers();
    });

    return {
      users,
      loading,
      currentPage,
      itemsPerPage,
      searchQuery,
      selectedUser,
      filteredUsers,
      paginatedUsers,
      totalPages,
      displayedPages,
      startIndex,
      endIndex,
      nextPage,
      prevPage,
      selectUser,
      clearSelectedUser,
      formatDate
    };
  }
}
</script>

<style>
/* General Styles */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background-color: #f3f4f6;
  color: #1f2937;
  line-height: 1.5;
}

.user-list-container {
  width: 100%;
  min-height: 100vh;
}

.icon-button {
  background: none;
  border: none;
  padding: 0.5rem;
  border-radius: 9999px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-button:hover {
  background-color: #f3f4f6;
}

/* Main Content Styles */
.main-content {
  max-width: 1280px;
  margin: 0 auto;
  padding: 1.5rem 1rem;
}

/* Profile Section Styles */
.profile-section {
  margin-bottom: 2rem;
}

.profile-header {
  background-color: #2db8dc;
  border-radius: 0.5rem 0.5rem 0 0;
  padding: 1.5rem;
  color: white;
  position: relative;
  overflow: hidden;
}

.profile-header-content {
  display: flex;
  align-items: center;
}

.back-button {
  position: absolute;
  top: 1.5rem;
  left: 1.5rem;
  background-color: rgba(255, 255, 255, 0.2);
  border: none;
  padding: 0.25rem;
  border-radius: 9999px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.profile-info {
  margin: 0 auto;
  text-align: center;
}

.profile-picture {
  width: 6rem;
  height: 6rem;
  border-radius: 9999px;
  border: 4px solid white;
  margin: 0 auto;
}

.user-name {
  margin-top: 1rem;
  font-size: 1.5rem;
  font-weight: 700;
}

.last-online {
  font-size: 0.875rem;
  opacity: 0.8;
}

.profile-actions {
  display: flex;
  justify-content: center;
  gap: 0.75rem;
  margin-top: 1rem;
}

.action-button {
  background-color: white;
  color: #4DB6AC;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  border: none;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  transition: background-color 0.2s;
}

.action-button:hover {
  background-color: #f9fafb;
}

.user-details {
  background-color: white;
  border-radius: 0 0 0.5rem 0.5rem;
  overflow: hidden;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.detail-header, .detail-row {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
}

.detail-header {
  background-color: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.detail-cell {
  padding: 0.75rem 1rem;
  font-size: 0.875rem;
}

.detail-header .detail-cell {
  color: #6b7280;
  font-weight: 500;
}

/* Controls Section Styles */
.controls-section {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

@media (min-width: 640px) {
  .controls-section {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
}

.search-container {
  position: relative;
  max-width: 24rem;
  width: 100%;
}

.search-input {
  width: 100%;
  padding: 0.5rem 0.75rem 0.5rem 2.5rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  line-height: 1.25rem;
}

.search-input:focus {
  outline: none;
  border-color: #4DB6AC;
  box-shadow: 0 0 0 1px #4DB6AC;
}

.search-icon {
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  width: 1.25rem;
  height: 1.25rem;
  color: #9ca3af;
}

.items-per-page {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.items-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: #4b5563;
}

.items-select {
  padding: 0.5rem 2rem 0.5rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  background-color: white;
  font-size: 0.875rem;
  cursor: pointer;
}

/* Users Table Styles */
.users-table {
  background-color: white;
  border-radius: 0.5rem;
  overflow: hidden;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 3rem 0;
}

.loader {
  border: 0.25rem solid #e5e7eb;
  border-top: 0.25rem solid #4DB6AC;
  border-radius: 50%;
  width: 3rem;
  height: 3rem;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.table-header {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  background-color: #f9fafb;
  font-size: 0.875rem;
  font-weight: 500;
  color: #6b7280;
  border-bottom: 1px solid #e5e7eb;
}

.table-row {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  font-size: 0.875rem;
  border-bottom: 1px solid #e5e7eb;
  cursor: pointer;
  transition: background-color 0.2s;
}

.table-row:hover {
  background-color: #f9fafb;
}

.table-row:last-child {
  border-bottom: none;
}

.table-cell {
  padding: 0.75rem 1rem;
}

.capitalize {
  text-transform: capitalize;
}

.no-results {
  padding: 3rem 0;
  text-align: center;
  color: #6b7280;
}

/* Pagination Styles */
.pagination-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.75rem 1rem;
  background-color: #f9fafb;
  border-top: 1px solid #e5e7eb;
}

@media (min-width: 640px) {
  .pagination-container {
    flex-direction: row;
    justify-content: space-between;
  }
}

.pagination-info {
  margin-bottom: 1rem;
}

@media (min-width: 640px) {
  .pagination-info {
    margin-bottom: 0;
  }
}

.results-text {
  font-size: 0.875rem;
  color: #6b7280;
}

.results-number {
  font-weight: 500;
  color: #374151;
}

.pagination-controls {
  display: flex;
  align-items: center;
}

.pagination-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  padding: 0;
  border: 1px solid #d1d5db;
  background-color: white;
  cursor: pointer;
}

.pagination-button.prev {
  border-radius: 0.375rem 0 0 0.375rem;
}

.pagination-button.next {
  border-radius: 0 0.375rem 0.375rem 0;
}

.pagination-button.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pagination-page {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border: 1px solid #d1d5db;
  border-left: none;
  background-color: white;
  font-size: 0.875rem;
  color: #374151;
  cursor: pointer;
}

.pagination-page.active {
  background-color: #eef7f6;
  border-color: #4DB6AC;
  color: #4DB6AC;
  position: relative;
  z-index: 10;
}

.pagination-ellipsis {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border: 1px solid #d1d5db;
  border-left: none;
  background-color: white;
  font-size: 0.875rem;
  color: #6b7280;
}

/* Animations */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(20px);
  opacity: 0;
}

/* Icons */
.back-icon, .message-icon, .add-friend-icon, .search-icon, .notification-icon, .settings-icon, .add-icon, .prev-icon, .next-icon {
  display: inline-block;
  width: 1.5rem;
  height: 1.5rem;
  background-color: currentColor;
  mask-size: cover;
  -webkit-mask-size: cover;
}

.back-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M10 19l-7-7m0 0l7-7m-7 7h18' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M10 19l-7-7m0 0l7-7m-7 7h18' /%3E%3C/svg%3E");
}

.message-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z' /%3E%3C/svg%3E");
}

.add-friend-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z' /%3E%3C/svg%3E");
}

.search-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z' /%3E%3C/svg%3E");
}

.notification-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9' /%3E%3C/svg%3E");
}

.settings-icon {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.732c1.5-.5 3.2.5 3.2 2.4v1.6c0 1.9-1.7 2.9-3.2 2.4a1.724 1.724 0 00-2.573 1.732c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.732c-1.5-.5-3.2.5-3.2 2.4v1.6c0 1.9 1.7 2.9 3.2 2.4a1.724 1.724 0 002.573-1.732c-.426-1.756-.426-3 .426-4z' /%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='currentColor'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.732c1.5-.5 3.2.5 3.2 2.4v1.6c0 1.9-1.7 2.9-3.2 2.4a1.724 1.724 0 00-2.573 1.732c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.732c-1.5-.5-3.2.5-3.2 2.4v1.6c0 1.9 1.7 2.9 3.2 2.4a1.724 1.724 0 002.573-1.732c-.426-1.756-.426-3 .426-4z' /%3E%3C/svg%3E");
}
</style>