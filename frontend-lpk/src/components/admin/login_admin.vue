<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { EyeIcon, EyeSlashIcon } from "@heroicons/vue/24/solid";
import { PhoneIcon } from "@heroicons/vue/24/outline";
import Header from "../bar/header.vue";

// API Configuration
const API_BASE_URL = "http://localhost:3000/api";

const router = useRouter();
const showPassword = ref(false);
const showForgotPasswordModal = ref(false);
const loading = ref(false);
const errorMessage = ref("");

const phoneNumber = ref("");
const password = ref("");

const handleLogin = async () => {
  // Validasi input
  if (!phoneNumber.value.trim()) {
    errorMessage.value = "Nomor telepon wajib diisi";
    return;
  }
  if (!password.value.trim()) {
    errorMessage.value = "Password wajib diisi";
    return;
  }

  loading.value = true;
  errorMessage.value = "";

  try {
    const response = await fetch(`${API_BASE_URL}/auth/login`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        nomorTelepon: phoneNumber.value,
        password: password.value,
      }),
    });

    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.message || "Login gagal");
    }

    // Simpan token dan user data ke localStorage
    localStorage.setItem("authToken", result.data.token);
    localStorage.setItem("userData", JSON.stringify(result.data.user));

    // Redirect ke dashboard
    router.push({ name: "data-monitor" });
  } catch (error) {
    console.error("Login error:", error);
    errorMessage.value = error.message || "Terjadi kesalahan saat login";
  } finally {
    loading.value = false;
  }
};

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

const openForgotPasswordModal = () => {
  showForgotPasswordModal.value = true;
};

const closeForgotPasswordModal = () => {
  showForgotPasswordModal.value = false;
};

const handleWaLink = () => {
  window.open("https://wa.me/6282254442400", "_blank");
};
</script>

<template>
  <div
    class="w-screen h-screen fixed inset-0 flex items-center justify-center bg-center bg-cover bg-no-repeat"
    style="background-image: url(&quot;/image_asset/backgrond.png&quot;)"
  >
    <Header />
    <div
      class="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-md mx-4 border border-gray-100"
    >
      <img src="/image_asset/IMM.svg" class="w-32 h-auto block mx-auto" />

      <div class="flex flex-col gap-3 text-center w-full">
        <div class="flex flex-col gap-1 mt-4">
          <p class="text-black text-sm font-bold">
            Layanan Permintaan Kendaraan
          </p>
          <p class="text-black text-sm -mt-1 mb-2 font-bold">
            Operasional PT Indominco Mandiri
          </p>
        </div>

        <div class="flex flex-col gap-4">
          <input
            id="phone_number"
            v-model="phoneNumber"
            type="tel"
            placeholder="Nomor Handphone"
            autocomplete="tel"
            class="px-3.75 py-3 border border-[#a1a1a1] bg-white rounded-lg text-[14px] text-[#333] transition-colors duration-300 focus:outline-none focus:border-[#646cff] focus:ring-3 focus:ring-[#646cff]/10"
          />
          <div class="relative flex flex-col">
            <input
              id="password"
              name="password"
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              placeholder="Password (nama + tanggal lahir)"
              autocomplete="current-password"
              class="px-3.75 py-3 border border-[#a1a1a1] bg-white rounded-lg text-[14px] text-[#333] transition-colors duration-300 focus:outline-none focus:border-[#646cff] focus:ring-3 focus:ring-[#646cff]/10"
            />
            <button
              type="button"
              @click="togglePasswordVisibility"
              class="absolute right-3 top-1/2 -translate-y-1/2 bg-none border-none p-0 cursor-pointer w-5 h-5 flex items-center justify-center transition-all duration-300 hover:opacity-70"
            >
              <EyeIcon
                v-if="showPassword"
                class="w-5 h-5 text-[#646cff] hover:text-[#535bf2]"
              />
              <EyeSlashIcon
                v-else
                class="w-5 h-5 text-[#646cff] hover:text-[#535bf2]"
              />
            </button>
          </div>
          <div class="flex justify-end mb-2.5">
            <a
              @click.prevent="openForgotPasswordModal"
              class="text-[13px] font-semibold text-[#646cff] transition-colors duration-300 hover:text-[#535bf2] hover:underline cursor-pointer"
            >
              Lupa Password?
            </a>
          </div>

          <!-- Error Message -->
          <div
            v-if="errorMessage"
            class="p-3 bg-red-50 border border-red-200 rounded-lg text-red-600 text-sm text-left"
          >
            {{ errorMessage }}
          </div>

          <!-- Sign In -->
          <div class="flex justify-center">
            <button
              @click="handleLogin"
              :disabled="loading"
              type="submit"
              class="w-fit px-25 py-3 bg-[#523E95] text-white rounded-xl text-[16px] font-semilight cursor-pointer transition-colors duration-300 hover:bg-[#43317d] disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {{ loading ? "Memuat..." : "Masuk" }}
            </button>
          </div>
          <p
            class="m-0 mt-4 text-left text-[#ff6464] text-xs font-semibold underline"
          >
            *Notes : Anda dapat mengakses monitor kendaraan tanpa perlu login
          </p>
        </div>
      </div>
      <!-- Forgot Password -->
      <div
        v-if="showForgotPasswordModal"
        class="fixed inset-0 bg-black/70 flex items-center justify-center z-1000"
      >
        <div
          class="bg-white p-10 rounded-lg text-center max-w-130 m-2.5 shadow-[0_4px_6px_rgba(0,0,0,0.1)]"
        >
          <h3
            class="text-[#333] text-[17px] font-regular mb-5 leading-normal text-left mt-0"
          >
            Silahkan hubungi Admin Transportasi Manajemen IMM agar kami dapat
            memberikan informasi lebih lanjut tentang password anda.
          </h3>

          <p
            class="text-[#333] text-[16px] mb-5 leading-normal text-left mt-5 flex items-center gap-3"
          >
            <PhoneIcon class="shrink-0 w-5 h-5 text-[#3b82f6]" />
            <span class="font-bold"> Admin :</span>
            <a
              href="#"
              @click.prevent="handleWaLink"
              class="text-[#646cff] text-[16px] font-medium underline decoration-1"
              style="
                text-decoration: underline;
                text-decoration-thickness: 1.2px;
                text-underline-offset: 3px;
              "
            >
              0812-3456-7890
            </a>
          </p>

          <button
            @click="closeForgotPasswordModal"
            class="px-7.5 py-3 mt-2.5 bg-[#646cff] text-white border-none rounded-md text-[16px] font-semibold cursor-pointer transition-colors duration-300 hover:bg-[#535bf2]"
          >
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
