
<template>
     <!-- Header -->
     <header class="header">
        <div class="header-content">
          <div class="logo-container">
            <img src="../assets/logo.png" alt="KiraTech Logo" class="logo" />
          </div>
          
          <div class="header-icons">
            <button class="icon-button">
              <i class="notification-icon"></i>
            </button>
            <button class="icon-button">
              <i class="settings-icon"></i>
            </button>
            <button class="icon-button">
              <i class="logout-icon"></i>
            </button>
          </div>
        </div>
      </header>
</template>
<script lang="ts" setup>

</script>
<style>

/* Header Styles */
.header {
    background-color: #ffffff;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }
  
  .header-content {
    max-width: 1280px;
    margin: 0 auto;
    padding: 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .logo-container {
    display: flex;
    align-items: center;
  }
  
  .logo {
    height: 3.5rem;
    width: auto;
  }
  
  .company-name {
    margin-left: 0.75rem;
    font-size: 1.5rem;
    font-weight: 700;
    color: #111827;
  }
  
  .header-icons {
    display: flex;
    gap: 1rem;
  }
  
</style>