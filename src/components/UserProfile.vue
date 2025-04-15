<template>
  <div class="profile-container">
    <!-- Banner -->
    <div class="profile-banner">
      <div class="profile-content">
        <!-- Profile Image -->
        <div class="profile-image-wrapper">
          <img :src="userData.profilePicture" alt="Profile" class="profile-image" />
        </div>
        <!-- Name and Status -->
        <div class="profile-details">
<div class="profile-text">
  <h1 class="profile-name">{{ userData.name }}</h1>
  <p class="last-online">Last seen: {{ userData.lastOnline }}</p>

</div>          <div class="profile-actions">
            <button class="btn btn-primary">
              <SendIcon class="icon" /> Send Message
            </button>
            <button class="btn btn-outline">
              <UserPlusIcon class="icon" /> Add Friend
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref } from 'vue';
import { Send as SendIcon, UserPlus as UserPlusIcon } from 'lucide-vue-next';

const userData = ref({
  name: 'John Doe',
  lastOnline: '2 days ago',
  profilePicture: require('../assets/profile-picture.png'),
});
</script>

<style scoped>.profile-container {
  position: relative;
  padding-bottom: 80px;
}

.profile-banner {
  background-color: #2db8dc;
  padding: 30px;
  position: relative;
  border-radius: 0;
  color: white;
}

.profile-content {
  display: flex;
  align-items: center;
  gap: 40px;
  position: relative;
}

.profile-image-wrapper {
  position: absolute;
  bottom: -75px;
  left: 40px;
  width: 150px;
  height: 200px;
  border-radius: 20px;
  overflow: hidden;
  background-color: white;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
  border: 4px solid #fff;
}

.profile-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.profile-text {
  margin-right: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.profile-details {
  margin-left: 200px;
  display: flex;
  flex-direction: row;
  justify-content: center;
}

.profile-name {
  font-size: 2.2rem;
  font-weight: 700;
}

.last-online {
  font-size: 1.0rem;
  color: #d0f0f8;
  margin: 6px 0 20px;
}

.profile-actions {
  display: flex;
  gap: 12px;
  margin: 0 0 0 20px;
  flex-direction: column;
  justify-content: center;
  height: 20%;
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  font-size: 1rem;
  font-weight: 500;
  border-radius: 9999px;
  cursor: pointer;
  transition: all 0.25s ease-in-out;
}

.btn-primary {
  background-color: white;
  color: #2db8dc;
  border: none;
}

.btn-primary:hover {
  background-color: #e0f7fa;
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.btn-outline {
  background-color: transparent;
  border: 2px solid white;
  color: white;
}

.btn-outline:hover {
  background-color: rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.icon {
  width: 18px;
  height: 18px;
}
/* Responsive Styles */
@media (max-width: 768px) {
  .profile-content {
    flex-direction: column;
    align-items: center;
    gap: 20px;
    padding-top: 120px; /* create space for image */
  }

  .profile-image-wrapper {
    position: relative;
    bottom: auto;
    left: auto;
    margin-bottom: 20px;
    margin-top: -100px; /* to overlap slightly like on desktop */
  }

  .profile-details {
    margin-left: 0;
    align-items: center;
    text-align: center;
    flex-direction: column;
  }

  .profile-text {
    margin-right: 0;
  }

  .profile-actions {
    margin: 16px 0 0;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
  }

  .btn {
    width: 100%;
    max-width: 280px;
    justify-content: center;
  }
}


</style>
