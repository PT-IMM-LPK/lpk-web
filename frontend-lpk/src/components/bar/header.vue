<script setup>
import { useRouter } from "vue-router";
import { computed, ref } from "vue";

const router = useRouter();
const isMenuOpen = ref(false);
const hoverState = ref({});

const handleLogPermintaan = () => {
  router.push({ name: "log-permintaan" });
  isMenuOpen.value = false;
};

const handleLogin = () => {
  router.push({ name: "login-admin" });
  isMenuOpen.value = false;
};

const handleForm = () => {
  router.push({ name: "form-lpk" });
  isMenuOpen.value = false;
};

const handleMonitor = () => {
  // Check if user is logged in
  const authToken = localStorage.getItem("authToken");
  if (!authToken) {
    // Redirect to login if not authenticated
    router.push({ name: "login-admin" });
    isMenuOpen.value = false;
    return;
  }
  router.push({ name: "data-monitor" });
  isMenuOpen.value = false;
};

const currentRoute = computed(() => router.currentRoute.value.name);

const getButtonClass = (routeName) => {
  const isActive = currentRoute.value === routeName;
  return isActive
    ? "text-sm font-bold transition-colors"
    : "text-sm font-light transition-colors";
};

const getButtonColor = (routeName) => {
  const isActive = currentRoute.value === routeName;
  return isActive ? "#523E95" : "#646464";
};

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};
</script>

<template>
  <header
    class="fixed top-0 left-0 w-full z-50 bg-white/70 backdrop-blur-lg shadow-md px-4 md:px-6 py-3 md:py-4 flex justify-between items-center"
  >
    <div class="flex items-center gap-2 md:gap-3 min-w-0 flex-1">
      <img
        src="/image_asset/2_Ptimm.png"
        alt="Logo"
        class="h-7 md:h-8 w-auto shrink-0"
      />

      <!-- HR vertikal valid -->
      <div class="h-6 md:h-8 w-0.5 bg-[#cacaca] rounded-lg shrink-0"></div>

      <span
        class="font-['Montserrat'] font-semibold text-md md:text-md text-[#523E95] truncate"
      >
        Layanan Permintaan Kendaraan
      </span>
    </div>

    <!-- Desktop Navigation -->
    <nav class="hidden lg:flex items-center gap-4 xl:gap-6 ml-auto">
      <button
        @click="handleLogPermintaan"
        @mouseenter="hoverState['log-permintaan'] = true"
        @mouseleave="hoverState['log-permintaan'] = false"
        :class="getButtonClass('log-permintaan')"
        :style="{ color: getButtonColor('log-permintaan') }"
        class="text-xs lg:text-sm transition-all duration-200 whitespace-nowrap"
      >
        Log Permintaan
      </button>

      <div class="h-5 w-px bg-gray-600"></div>
      <button
        @click="handleForm"
        :class="getButtonClass('form-lpk')"
        :style="{ color: getButtonColor('form-lpk') }"
        class="text-xs lg:text-sm transition-all duration-200 whitespace-nowrap"
      >
        Form Permintaan
      </button>

      <div class="h-5 w-px bg-gray-600"></div>
      <button
        @click="handleMonitor"
        :class="getButtonClass('data-monitor')"
        :style="{ color: getButtonColor('data-monitor') }"
        class="text-xs lg:text-sm transition-all duration-200 whitespace-nowrap"
      >
        Dashboard
      </button>

      <div class="h-5 w-px bg-gray-600"></div>
      <button
        @click="handleLogin"
        :class="getButtonClass('login-admin')"
        :style="{ color: getButtonColor('login-admin') }"
        class="text-xs lg:text-sm transition-all duration-200 whitespace-nowrap"
      >
        Login
      </button>
    </nav>

    <!-- Mobile Hamburger Button -->
    <button
      @click="toggleMenu"
      class="lg:hidden p-2 hover:bg-gray-100 rounded-lg transition-colors duration-200 ml-auto"
    >
      <div v-if="!isMenuOpen" class="space-y-1.5">
        <div class="w-5 h-0.5 bg-[#523E95] transition-all duration-300"></div>
        <div class="w-5 h-0.5 bg-[#523E95] transition-all duration-300"></div>
        <div class="w-5 h-0.5 bg-[#523E95] transition-all duration-300"></div>
      </div>
      <div v-else class="text-xl text-[#523E95] font-bold">✕</div>
    </button>
  </header>

  <!-- Mobile Menu -->
  <transition
    enter-active-class="transition ease-out duration-200"
    enter-from-class="opacity-0 -translate-y-1"
    enter-to-class="opacity-100 translate-y-0"
    leave-active-class="transition ease-in duration-150"
    leave-from-class="opacity-100 translate-y-0"
    leave-to-class="opacity-0 -translate-y-1"
  >
    <nav
      v-if="isMenuOpen"
      class="lg:hidden fixed top-16 left-0 w-full bg-white/95 backdrop-blur-lg shadow-md z-40 border-b border-gray-200"
    >
      <div class="px-4 py-3 space-y-1">
        <button
          @click="handleLogPermintaan"
          class="block w-full text-left px-3 py-2.5 text-sm font-medium text-gray-500 rounded-md hover:bg-gray-50 hover:text-gray-700 transition-colors duration-200"
        >
          Log Permintaan
        </button>
        <button
          @click="handleLogin"
          class="block w-full text-left px-3 py-2.5 text-sm font-medium text-gray-500 rounded-md hover:bg-gray-50 hover:text-gray-700 transition-colors duration-200"
        >
          Login
        </button>
        <button
          @click="handleForm"
          class="block w-full text-left px-3 py-2.5 text-sm font-medium text-gray-500 rounded-md hover:bg-gray-50 hover:text-gray-700 transition-colors duration-200"
        >
          Form Permintaan
        </button>
      </div>
    </nav>
  </transition>
</template>
