<script setup>
import { ref, provide, onMounted, computed } from "vue";
import Aside from "../bar/aside.vue";
import HeaderAdmin from "../bar/header-admin.vue";
import {
  XMarkIcon,
  PencilSquareIcon,
  ChevronDownIcon,
} from "@heroicons/vue/24/solid";

// API Configuration
const API_BASE_URL = "http://localhost:3000/api";

// Mobile menu state
const isMobileMenuOpen = ref(false);
const showEditAkun = ref(false);
const loading = ref(false);
const saving = ref(false);
const errorMessage = ref("");

// User data state
const userData = ref({
  nomor: null,
  nama: "",
  email: "",
  nomorTelepon: "",
  tanggalLahir: "",
  departemen: null,
  role: "",
});

// Departemen options for dropdown
const departemenOptions = ref([]);

// State untuk form edit
const editFormData = ref({
  nama: "",
  nomorTelepon: "",
  email: "",
  tanggalLahir: "",
  departemenId: "",
});

// Computed properties for display
const adminName = computed(() => userData.value.nama || "Nama tidak tersedia");
const adminEmail = computed(
  () => userData.value.email || "Email tidak tersedia",
);
const adminNoTelepon = computed(
  () => userData.value.nomorTelepon || "Nomor tidak tersedia",
);
const adminTanggalLahir = computed(() => {
  if (!userData.value.tanggalLahir) return "Tanggal tidak tersedia";
  const date = new Date(userData.value.tanggalLahir);
  return date.toLocaleDateString("id-ID", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
  });
});
const adminDepartemen = computed(
  () =>
    userData.value.departemen?.namaDepartemen || "Departemen tidak tersedia",
);
const roleKerja = computed(() => {
  const role = userData.value.role;
  if (!role) return "Role tidak tersedia";

  const roleMap = {
    ADMIN: "Admin",
    SUPER_ADMIN: "Super Admin",
    KARYAWAN: "Karyawan",
  };
  return roleMap[role] || role;
});

// Get auth headers
const getAuthHeaders = () => {
  const token = localStorage.getItem("authToken");
  return {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  };
};

// Fetch user data from API
const fetchUserData = async () => {
  loading.value = true;
  errorMessage.value = "";

  try {
    // Get user ID from localStorage
    const storedUserData = localStorage.getItem("userData");
    if (!storedUserData) {
      errorMessage.value =
        "Data pengguna tidak ditemukan. Silakan login ulang.";
      return;
    }

    const parsedUserData = JSON.parse(storedUserData);
    const userId = parsedUserData.nomor;

    // Fetch fresh data from API
    const response = await fetch(`${API_BASE_URL}/pengguna/${userId}`, {
      headers: getAuthHeaders(),
    });

    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.message || "Gagal mengambil data pengguna");
    }

    userData.value = result.data.pengguna;

    // Also update localStorage with fresh data
    localStorage.setItem("userData", JSON.stringify(result.data.pengguna));
  } catch (error) {
    console.error("Error fetching user data:", error);
    // Fallback to localStorage data
    const storedUserData = localStorage.getItem("userData");
    if (storedUserData) {
      userData.value = JSON.parse(storedUserData);
    } else {
      errorMessage.value = error.message;
    }
  } finally {
    loading.value = false;
  }
};

// Fetch departemen options
const fetchDepartemen = async () => {
  try {
    const response = await fetch(`${API_BASE_URL}/departemen`, {
      headers: getAuthHeaders(),
    });
    const result = await response.json();
    if (result.success) {
      departemenOptions.value = result.data.departemen;
    }
  } catch (error) {
    console.error("Error fetching departemen:", error);
  }
};

// Toggle mobile menu
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
};

// Open edit modal and populate form
const openEditAkun = () => {
  editFormData.value = {
    nama: userData.value.nama || "",
    nomorTelepon: userData.value.nomorTelepon || "",
    email: userData.value.email || "",
    tanggalLahir: userData.value.tanggalLahir
      ? new Date(userData.value.tanggalLahir).toISOString().split("T")[0]
      : "",
    departemenId: userData.value.departemen?.nomor || "",
  };
  showEditAkun.value = true;
};

const closeEditAkun = () => {
  showEditAkun.value = false;
  errorMessage.value = "";
};

// Save profile changes
const saveProfile = async () => {
  saving.value = true;
  errorMessage.value = "";

  try {
    const response = await fetch(
      `${API_BASE_URL}/pengguna/${userData.value.nomor}`,
      {
        method: "PUT",
        headers: getAuthHeaders(),
        body: JSON.stringify({
          nama: editFormData.value.nama,
          nomorTelepon: editFormData.value.nomorTelepon,
          email: editFormData.value.email,
          tanggalLahir: editFormData.value.tanggalLahir,
          departemenId: editFormData.value.departemenId || null,
        }),
      },
    );

    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.message || "Gagal menyimpan perubahan");
    }

    // Refresh user data
    await fetchUserData();
    closeEditAkun();
    alert("Profil berhasil diperbarui!");
  } catch (error) {
    console.error("Error saving profile:", error);
    errorMessage.value = error.message;
  } finally {
    saving.value = false;
  }
};

// Lifecycle
onMounted(async () => {
  await Promise.all([fetchUserData(), fetchDepartemen()]);
});

provide("isMobileMenuOpen", isMobileMenuOpen);
provide("toggleMobileMenu", toggleMobileMenu);
</script>

<template>
  <div class="min-h-screen flex flex-col font-['Montserrat']">
    <div class="flex flex-1 overflow-hidden">
      <Aside />

      <div class="flex flex-col flex-1 overflow-hidden">
        <HeaderAdmin />

        <!-- Content -->
        <main class="bg-[#EFEFEF] flex-1 flex flex-col p-3 overflow-hidden">
          <div
            class="bg-white rounded-lg shadow-md gap-4 p-5 flex-1 flex flex-col overflow-hidden"
          >
            <div class="flex justify-end">
              <button
                @click="openEditAkun"
                class="px-7 py-1 text-xs md:text-sm font-semilight bg-linear-to-r from-[#A90CF8] to-[#9600E1] text-white rounded-md hover:opacity-90 transition"
              >
                Edit akun
              </button>
            </div>

            <!-- Nama dan Email -->
            <div>
              <p class="text-base font-regular text-gray-800 mb-1">Nama</p>
              <input
                :value="adminName"
                type="text"
                placeholder="Nama Lengkap"
                disabled
                class="w-full p-2 text-sm border border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>

            <div>
              <p class="text-base font-regular text-gray-800 mb-1">Email</p>
              <input
                :value="adminEmail"
                type="text"
                placeholder="email@example.com"
                disabled
                class="w-full p-2 border text-sm border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>

            <!-- Nomor Telepon dan Tanggal Lahir -->
            <div>
              <p class="text-base font-regular text-gray-800 mb-1">
                Nomor Telepon
              </p>
              <input
                :value="adminNoTelepon"
                type="text"
                placeholder="081xxxxxxxx"
                disabled
                class="w-full p-2 border text-sm border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>

            <div>
              <p class="text-base font-regular text-gray-800 mb-1">
                Tanggal Lahir
              </p>
              <input
                :value="adminTanggalLahir"
                type="text"
                placeholder="hh/bb/tttt"
                disabled
                class="w-full p-2 border text-sm border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>

            <!-- Departemen -->
            <div>
              <p class="text-base font-regular text-gray-800 mb-1">
                Departemen
              </p>
              <input
                :value="adminDepartemen"
                type="text"
                placeholder="Departemen"
                disabled
                class="w-full p-2 border text-sm border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>

            <!-- Role-->
            <div>
              <p class="text-base font-regular text-gray-800 mb-1">Role</p>
              <input
                :value="roleKerja"
                type="text"
                placeholder="Role"
                disabled
                class="w-full p-2 border text-sm border-[#C3C3C3] bg-[#EEEEEE] text-[#777777] rounded-md cursor-not-allowed"
              />
            </div>
          </div>

          <!-- Konten Edit akun -->
          <div
            v-if="showEditAkun"
            class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4"
          >
            <div
              class="bg-white rounded-lg w-full max-w-md md:max-w-xl max-h-[90vh] overflow-y-auto shadow-[0_4px_6px_rgba(0,0,0,0.1)] p-6 md:p-8"
            >
              <div
                class="flex justify-between items-center mb-4 pb-3 border-b border-gray-200"
              >
                <h2 class="text-lg md:text-xl font-semibold text-gray-900">
                  Edit Profil
                </h2>
                <button
                  @click="closeEditAkun"
                  class="shrink-0 p-1 hover:bg-gray-100 rounded-md transition"
                >
                  <XMarkIcon
                    class="w-6 h-6 text-gray-600 hover:text-gray-900"
                  />
                </button>
              </div>

              <div>
                <label class="block text-base font-medium text-black mb-2 mt-4"
                  >Nama Lengkap</label
                >
                <div class="relative">
                  <input
                    type="text"
                    v-model="editFormData.nama"
                    placeholder="Masukkan nama"
                    class="w-full p-2 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-sm focus:outline-none focus:border-[#A90CF8]"
                  />
                  <PencilSquareIcon
                    class="absolute right-3 top-2.5 w-5 h-5 text-[#C3C3C3]"
                  />
                </div>
              </div>
              <div>
                <label
                  class="block text-base font-medium text-gray-800 mb-2 mt-2"
                  >Nomor Handphone</label
                >
                <div class="relative">
                  <input
                    type="text"
                    v-model="editFormData.nomorTelepon"
                    placeholder="081xxxxxxxx"
                    class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-sm focus:outline-none focus:border-[#A90CF8]"
                  />
                  <PencilSquareIcon
                    class="absolute right-3 top-2.5 w-5 h-5 text-[#C3C3C3]"
                  />
                </div>
              </div>
              <div>
                <label
                  class="block text-base font-medium text-gray-800 mb-2 mt-2"
                  >Email</label
                >
                <div class="relative">
                  <input
                    type="email"
                    v-model="editFormData.email"
                    placeholder="email@example.com"
                    class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-sm focus:outline-none focus:border-[#A90CF8]"
                  />
                  <PencilSquareIcon
                    class="absolute right-3 top-2.5 w-5 h-5 text-[#C3C3C3]"
                  />
                </div>
              </div>
              <div>
                <label
                  class="block text-base font-medium text-gray-800 mb-2 mt-2"
                  >Tanggal Lahir</label
                >
                <input
                  type="date"
                  v-model="editFormData.tanggalLahir"
                  class="w-full p-2 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-sm focus:outline-none focus:border-[#A90CF8]"
                />
              </div>
              <div>
                <label
                  class="block text-base font-medium text-gray-800 mb-2 mt-2"
                  >Departemen</label
                >
                <div class="relative">
                  <select
                    v-model="editFormData.departemenId"
                    class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-sm focus:outline-none focus:border-[#A90CF8] focus:ring-2 focus:ring-[#A90CF8]/20 appearance-none cursor-pointer"
                  >
                    <option value="">Pilih Departemen</option>
                    <option
                      v-for="dept in departemenOptions"
                      :key="dept.nomor"
                      :value="dept.nomor"
                    >
                      {{ dept.namaDepartemen }}
                    </option>
                  </select>
                  <ChevronDownIcon
                    class="absolute right-3 top-2.5 w-5 h-5 text-[#C3C3C3] pointer-events-none"
                  />
                </div>
              </div>

              <!-- Error Message -->
              <div
                v-if="errorMessage"
                class="mt-4 p-3 bg-red-100 border border-red-300 rounded-md"
              >
                <p class="text-sm text-red-700">{{ errorMessage }}</p>
              </div>

              <div class="flex justify-end gap-3 mt-6">
                <button
                  @click="saveProfile"
                  :disabled="saving"
                  class="px-8 md:px-10 py-2 text-sm md:text-base bg-linear-to-r from-[#A90CF8] to-[#9600E1] text-white rounded-xl hover:opacity-90 transition font-regular disabled:opacity-50 disabled:cursor-not-allowed"
                >
                  {{ saving ? "Menyimpan..." : "Simpan" }}
                </button>
                <button
                  @click="closeEditAkun"
                  :disabled="saving"
                  class="px-6 md:px-6 py-2 text-sm md:text-base border border-gray-300 bg-red-600 text-white rounded-xl hover:bg-red-700 transition font-regular disabled:opacity-50"
                >
                  Batal
                </button>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  </div>
</template>
