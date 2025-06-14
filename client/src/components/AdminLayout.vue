<template>
  <div class="admin-layout">
    <!-- Mobile Header with Menu Toggle -->
    <div class="mobile-admin-header md:hidden flex items-center justify-between p-4 bg-[#1C1B1F] border-b border-[#3D3A42] shadow-lg">
      <h1 class="text-xl font-bold text-white flex items-center gap-2">
        <scissors-icon :size="20" class="text-[#D4AF37]" /> 
        <span class="text-[#D4AF37] font-black">Tailorly</span>
        <span class="text-white ml-1">Admin</span>
      </h1>
      <button 
        class="menu-toggle-btn"
        @click="toggleSidebar"
        aria-label="Toggle menu"
      >
        <div class="hamburger" :class="{ 'is-active': sidebarOpen }">
          <span class="bg-[#D4AF37]"></span>
          <span class="bg-[#D4AF37]"></span>
          <span class="bg-[#D4AF37]"></span>
        </div>
      </button>
    </div>
    
    <!-- Admin Sidebar -->
    <aside 
      class="admin-sidebar"
      :class="{ 'open': sidebarOpen }"
    >
      <div class="sidebar-header hidden md:flex items-center p-5 border-b border-[#3D3A42] bg-[#18171A]">
        <div class="flex items-center gap-2">
          <scissors-icon :size="22" class="text-[#D4AF37]" />
          <h1 class="text-xl font-bold">
            <span class="text-[#D4AF37]">Tailorly</span>
            <span class="text-white ml-1">Admin</span>
          </h1>
        </div>
      </div>
      
      <div class="sidebar-content py-6">
        <ul class="space-y-2">
          <li v-for="item in menuItems" :key="item.id">
            <a 
              href="#" 
              class="sidebar-link flex items-center p-3 mx-3 rounded-md transition-all duration-200 hover:bg-[#252429] hover:text-[#D4AF37]"
              :class="{ 'active bg-[#252429] text-[#D4AF37] border-l-4 border-[#D4AF37]': activeSection === item.id }"
              @click.prevent="showSection(item.id)"
            >
              <el-icon class="mr-3" :class="{ 'text-[#D4AF37]': activeSection === item.id, 'text-[#C4C4C4]': activeSection !== item.id }"><component :is="item.icon" /></el-icon>
              <span class="font-medium">{{ item.name }}</span>
            </a>
          </li>
        </ul>
      </div>
      
      <!-- Logout Button -->
      <div class="sidebar-footer p-4 border-t border-[#3D3A42] mt-auto">
        <button 
          class="logout-btn flex items-center justify-center w-full p-3 rounded-md bg-[#D4AF37] text-[#1C1B1F] hover:bg-[#E5C158] transition-all duration-200 shadow-md"
          @click="logout"
        >
          <el-icon class="mr-2"><SwitchButton /></el-icon>
          <span class="font-medium">Logout</span>
        </button>
      </div>
    </aside>
    
    <!-- Admin Content -->
    <div class="admin-content bg-[#1C1B1F]">
      <!-- Content Header -->
      <div class="content-header p-4 md:p-6 border-b border-[#3D3A42] flex items-center justify-between">
        <h2 class="text-xl md:text-2xl font-bold text-white">
          <span class="text-[#D4AF37]">{{ activeSection.charAt(0).toUpperCase() + activeSection.slice(1) }}</span>
        </h2>
        <div class="flex items-center gap-3">
          <span class="text-[#C4C4C4] hidden md:inline-block">{{ new Date().toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }) }}</span>
        </div>
      </div>
      
      <!-- Content Sections -->
      <div 
        v-for="item in menuItems" 
        :key="item.id"
        class="admin-content-section p-4 md:p-6"
        :class="{ 'hidden': activeSection !== item.id }"
        :id="'section-' + item.id"
      >
        <slot :name="item.id"></slot>
      </div>
    </div>
    
    <!-- Mobile Sidebar Overlay -->
    <div 
      v-if="windowWidth <= 768" 
      class="sidebar-overlay"
      :class="{ 'active': sidebarOpen }"
      @click="closeSidebar"
    ></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { useUserStore } from '../stores/user';
import { SwitchButton } from '@element-plus/icons-vue';

// Props
const props = defineProps({
  defaultSection: {
    type: String,
    default: 'dashboard'
  }
});

// Menu items
const menuItems = [
  { id: 'dashboard', name: 'Dashboard', icon: 'DataAnalysis' },
  { id: 'orders', name: 'Orders', icon: 'List' },
  { id: 'clients', name: 'Clients', icon: 'User' },
  { id: 'inventory', name: 'Inventory', icon: 'Box' },
  { id: 'settings', name: 'Settings', icon: 'Setting' }
];

// Active section
const activeSection = ref(props.defaultSection);

// Sidebar state
const sidebarOpen = ref(false);

// Router and store
const router = useRouter();
const userStore = useUserStore();

// Window width for responsive behavior
const windowWidth = ref(window.innerWidth);

// Toggle sidebar
const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value;
  
  // Prevent body scrolling when sidebar is open on mobile
  if (sidebarOpen.value && windowWidth.value <= 768) {
    document.body.classList.add('overflow-hidden');
  } else {
    document.body.classList.remove('overflow-hidden');
  }
};

// Close sidebar
const closeSidebar = () => {
  sidebarOpen.value = false;
  document.body.classList.remove('overflow-hidden');
};

// Show section
const showSection = (sectionId) => {
  activeSection.value = sectionId;
  
  // Navigate to the corresponding route
  router.push(`/admin/${sectionId}`);
  
  // Close sidebar on mobile after selection
  if (windowWidth.value <= 768) {
    closeSidebar();
  }
};

// Update window width on resize
const handleResize = () => {
  windowWidth.value = window.innerWidth;
  
  // Auto-close sidebar on larger screens
  if (windowWidth.value > 768) {
    document.body.classList.remove('overflow-hidden');
  }
};

// Add and remove event listeners
onMounted(() => {
  window.addEventListener('resize', handleResize);
});

// Logout function
const logout = async () => {
  try {
    await userStore.logout();
    router.push('/login');
  } catch (error) {
    console.error('Logout error:', error);
  }
};

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  document.body.classList.remove('overflow-hidden');
});
</script>

<style scoped>
.admin-layout {
  display: flex;
  min-height: 100vh;
  position: relative;
  background-color: #1C1B1F;
  color: #FFFFFF;
}

.admin-sidebar {
  width: 250px;
  background-color: #1C1B1F;
  border-right: 1px solid #3D3A42;
  height: 100vh;
  position: sticky;
  top: 0;
  z-index: 10;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.3);
}

.admin-content {
  flex: 1;
  background-color: #1C1B1F;
  min-height: 100vh;
  color: #FFFFFF;
}

.sidebar-link {
  color: #C4C4C4;
  font-weight: 500;
  margin-bottom: 0.25rem;
  transition: all 0.3s ease;
}

.sidebar-link:hover {
  background-color: #252429;
  color: #D4AF37;
  transform: translateX(3px);
}

.sidebar-link.active {
  background-color: #252429;
  color: #D4AF37;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  border-left: 4px solid #D4AF37;
}

/* Mobile styles */
@media (max-width: 768px) {
  .admin-layout {
    display: block;
  }
  
  .admin-sidebar {
    position: fixed;
    left: -280px;
    top: 0;
    bottom: 0;
    width: 280px;
    z-index: 100;
    transition: left 0.3s ease;
    padding-top: 60px; /* Space for mobile header */
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow-y: auto;
    background-color: #1C1B1F;
    border-right: 1px solid #3D3A42;
  }
  
  .admin-sidebar.open {
    left: 0;
    box-shadow: 0 5px 25px rgba(0, 0, 0, 0.5);
  }
  
  .admin-content {
    margin-left: 0;
    width: 100%;
    padding: 0;
    padding-top: 60px; /* Space for mobile header */
  }
  
  .sidebar-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.7);
    backdrop-filter: blur(3px);
    z-index: 90;
    visibility: hidden;
    opacity: 0;
    transition: all 0.3s ease;
  }
  
  .sidebar-overlay.active {
    visibility: visible;
    opacity: 1;
  }
  
  .mobile-admin-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 80;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    background-color: #18171A; /* Slightly darker than main background */
    border-bottom: 1px solid #3D3A42;
  }
  
  /* Hamburger button */
  .menu-toggle-btn {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.5rem;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: all 0.3s ease;
  }
  
  .menu-toggle-btn:hover {
    background-color: rgba(212, 175, 55, 0.1);
  }
  
  .hamburger {
    width: 24px;
    height: 20px;
    position: relative;
    transform: rotate(0deg);
    transition: .5s ease-in-out;
  }
  
  .hamburger span {
    display: block;
    position: absolute;
    height: 2px;
    width: 100%;
    background: #D4AF37; /* Gold color for hamburger lines */
    border-radius: 2px;
    opacity: 1;
    left: 0;
    transform: rotate(0deg);
    transition: .25s ease-in-out;
  }
  
  .hamburger span:nth-child(1) {
    top: 0px;
  }
  
  .hamburger span:nth-child(2) {
    top: 8px;
    width: 75%;
  }
  
  .hamburger span:nth-child(3) {
    top: 16px;
  }
  
  .hamburger.is-active span {
    background: #D4AF37; /* Gold color for active state */
  }
  
  .hamburger.is-active span:nth-child(1) {
    top: 8px;
    transform: rotate(135deg);
    width: 100%;
  }
  
  .hamburger.is-active span:nth-child(2) {
    opacity: 0;
    left: -60px;
  }
  
  .hamburger.is-active span:nth-child(3) {
    top: 8px;
    transform: rotate(-135deg);
    width: 100%;
  }
  
  /* Content section styling */
  .content-header {
    padding: 1rem;
    border-bottom: 1px solid #3D3A42;
  }
  
  .admin-content-section {
    padding: 1rem;
  }
}
</style>
