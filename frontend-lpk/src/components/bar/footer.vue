<script setup>
import { PhoneIcon } from "@heroicons/vue/24/outline";

const handleWaLink = () => {
  window.open("https://wa.me/62", "_blank");
};
</script>

<template>
  <footer class="fixed bottom-0 left-0 right-0 bg-white shadow-lg font-['Montserrat'] z-40"
  style="background-image: url(/image_asset/); background-attachment: fixed;">
    
    <!-- Main Footer Content -->
    <div class="px-2 py-2 md:px-6 md:py-4 w-full">
      <div
        class="max-w-full mx-auto flex flex-col md:flex-row items-center justify-between gap-4 md:gap-4"
      >
        <!-- Left Side: Logo and Text -->
        <div class="flex items-center gap-3 justify-center md:justify-start shrink-0">
          <img src="/image_asset/IMM.svg" alt="IMM Logo" class="w-17 h-auto" />
          <div class="h-6 w-0.5 bg-[#cacaca] rounded-lg"></div>
          <div>
            <p
              class="text-md font-semibold text-[#523E95] m-0 text-center md:text-left"
            >
              Layanan Permintaan Kendaraan
            </p>
          </div>
        </div>

        <!-- Right Side: Phone Icon with Contact -->
        <div
          class="flex items-center gap-2 md:gap-3 justify-center md:justify-end shrink-0"
        >
          <PhoneIcon class="w-4 md:w-4 h-4 md:h-4 text-[#3b82f6] shrink-0" />
          <p class="text-xs md:text-sm font-semilight text-[#000000] m-0">
            Admin
          </p>
          <a
            href="#"
            @click.prevent="handleWaLink"
            class="text-xs md:text-sm text-[#646cff] font-medium hover:underline m-0"
            style="text-decoration: underline; text-decoration-thickness: 1px; text-underline-offset: 3px;"
          >
            0812-3456-7890
          </a>
        </div>
      </div>
    </div>

    <!-- Divider Line -->
    <div class="border-t border-[#e5e7eb]"></div>

    <!-- Copyright Section -->
    <div class="px-2 py-2 md:px-6 md:py-3">
      <div class="flex items-center justify-between gap-3">

        <!-- Center/Right: Copyright Text -->
        <p class="text-right text-xs font-light text-[#646464] m-0 flex-1">
          © PT. Indominco Mandiri. All right reserved
        </p>
      </div>
    </div>
  </footer>
</template>
