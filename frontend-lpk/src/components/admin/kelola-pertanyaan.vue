<script setup>
import { ref, provide } from "vue";
import Aside from "../bar/aside.vue";
import HeaderAdmin from "../bar/header-admin.vue";
import {
  TrashIcon,
  PlusIcon,
  XMarkIcon,
  ChevronDownIcon,
} from "@heroicons/vue/24/outline";

// Reactive state
const isMobileMenuOpen = ref(false);
const tambahPertanyaan = ref(false);
const editPertanyaan = ref(false);
const pertanyaanYangDiedit = ref(null);

// Form state untuk tambah
const formLabel = ref("");
const formTipe = ref("text");
const formPilihanList = ref([""]);

// Form state untuk edit
const formEditLabel = ref("");
const formEditTipe = ref("text");
const formEditPilihanList = ref([""]);

// Daftar pertanyaan (setiap pertanyaan satu kotak)
const pertanyaanList = ref([
  { id: 1, label: "Nama Lengkap", tipe: "text", pilihan_list: [] },
  { id: 2, label: "Tanggal Lahir", tipe: "date", pilihan_list: [] },
  {
    id: 3,
    label: "Jenis Kelamin",
    tipe: "multiple",
    pilihan_list: ["Laki-laki", "Perempuan"],
  },
]);

const getTipeLabel = (tipe) => {
  switch (tipe) {
    case "text":
      return "Input Text";
    case "date":
      return "Tanggal";
    case "multiple":
      return "Pilihan Ganda";
    default:
      return "Input Text";
  }
};

// Functions
const openTambahPertanyaan = () => {
  tambahPertanyaan.value = true;
  formLabel.value = "";
  formTipe.value = "text";
  formPilihanList.value = [""];
};

const closeTambahPertanyaan = () => {
  tambahPertanyaan.value = false;
};

const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
};

const openEditPertanyaan = (pertanyaan) => {
  editPertanyaan.value = true;
  pertanyaanYangDiedit.value = pertanyaan;
  formEditLabel.value = pertanyaan.label;
  formEditTipe.value = pertanyaan.tipe;
  formEditPilihanList.value = pertanyaan.pilihan_list?.length
    ? [...pertanyaan.pilihan_list]
    : [""];
};

const closeEditPertanyaan = () => {
  editPertanyaan.value = false;
  pertanyaanYangDiedit.value = null;
};

const simpanEditPertanyaan = () => {
  if (!formEditLabel.value.trim()) {
    alert("Label pertanyaan tidak boleh kosong!");
    return;
  }

  if (formEditTipe.value === "multiple") {
    const validOptions = formEditPilihanList.value.filter((opt) => opt.trim());
    if (validOptions.length < 2) {
      alert("Pilihan ganda minimal 2 opsi!");
      return;
    }
  }

  const index = pertanyaanList.value.findIndex(
    (p) => p.id === pertanyaanYangDiedit.value.id,
  );
  if (index > -1) {
    pertanyaanList.value[index] = {
      id: pertanyaanYangDiedit.value.id,
      label: formEditLabel.value,
      tipe: formEditTipe.value,
      pilihan_list:
        formEditTipe.value === "multiple"
          ? formEditPilihanList.value.filter((o) => o.trim())
          : [],
    };
  }

  closeEditPertanyaan();
  alert("Pertanyaan berhasil diupdate!");
};

const hapusPertanyaan = (pertanyaan) => {
  if (confirm("Hapus pertanyaan ini?")) {
    const index = pertanyaanList.value.findIndex((p) => p.id === pertanyaan.id);
    if (index > -1) pertanyaanList.value.splice(index, 1);
  }
};

const simpanPertanyaan = () => {
  if (!formLabel.value.trim()) {
    alert("Label pertanyaan tidak boleh kosong!");
    return;
  }

  if (formTipe.value === "multiple") {
    const validOptions = formPilihanList.value.filter((opt) => opt.trim());
    if (validOptions.length < 2) {
      alert("Pilihan ganda minimal 2 opsi!");
      return;
    }
  }

  const newPertanyaan = {
    id: Date.now(),
    label: formLabel.value,
    tipe: formTipe.value,
    pilihan_list:
      formTipe.value === "multiple"
        ? formPilihanList.value.filter((o) => o.trim())
        : [],
  };

  pertanyaanList.value.push(newPertanyaan);
  closeTambahPertanyaan();
  alert("Pertanyaan berhasil ditambahkan!");
};

provide("isMobileMenuOpen", isMobileMenuOpen);
provide("toggleMobileMenu", toggleMobileMenu);
</script>

<template>
  <div class="h-screen flex flex-col font-['Montserrat']">
    <div class="flex flex-1 overflow-hidden">
      <Aside />

      <div class="flex flex-col flex-1 min-w-0 overflow-hidden">
        <HeaderAdmin />

        <main class="bg-[#EFEFEF] flex-1 flex flex-col p-4 overflow-hidden">
          <div
            class="bg-white rounded-xl shadow p-6 flex-1 flex flex-col overflow-hidden min-h-0"
          >
            <!-- Header -->
            <div class="flex items-center justify-between mb-6 shrink-0">
              <div>
                <h1 class="text-xl font-bold text-black">Kelola Pertanyaan</h1>
                <p class="text-sm text-black mt-1">
                  Buat dan kelola daftar pertanyaan
                </p>
              </div>
              <button
                @click="openTambahPertanyaan"
                class="flex items-center gap-2 px-5 py-2.5 rounded-lg text-white bg-[#6444C6] hover:bg-[#5c3db8] text-sm font-medium"
              >
                <PlusIcon class="w-4 h-4" />
                Tambah Pertanyaan
              </button>
            </div>

            <!-- Content -->
            <div class="flex-1 overflow-y-auto space-y-3">
              <div
                v-for="(pertanyaan, idx) in pertanyaanList"
                :key="pertanyaan.id"
                class="bg-gray-50 rounded-lg border border-gray-200 p-4"
              >
                <div class="flex justify-between items-start">
                  <div class="flex-1">
                    <div class="flex items-center gap-3 mb-2">
                      <span class="text-sm font-bold text-black"
                        >{{ idx + 1 }}.</span
                      >
                      <span class="text-sm font-semibold text-black">{{
                        pertanyaan.label
                      }}</span>
                      <span
                        class="text-xs text-black bg-gray-200 px-2 py-0.5 rounded"
                      >
                        {{ getTipeLabel(pertanyaan.tipe) }}
                      </span>
                    </div>

                    <!-- Tampilkan opsi jika multiple -->
                    <div
                      v-if="
                        pertanyaan.tipe === 'multiple' &&
                        pertanyaan.pilihan_list.length > 0
                      "
                      class="ml-6"
                    >
                      <p class="text-xs text-black mb-1">Opsi:</p>
                      <div class="flex flex-wrap gap-2">
                        <span
                          v-for="(opt, i) in pertanyaan.pilihan_list"
                          :key="i"
                          class="text-xs text-black bg-white border border-gray-300 px-2 py-1 rounded"
                        >
                          {{ opt }}
                        </span>
                      </div>
                    </div>
                  </div>

                  <div class="flex gap-2 ml-4">
                    <button
                      @click="openEditPertanyaan(pertanyaan)"
                      class="px-3 py-1.5 text-sm font-medium bg-[#6444C6] text-white rounded-lg hover:bg-[#5c3db8]"
                    >
                      Edit
                    </button>
                    <button
                      @click="hapusPertanyaan(pertanyaan)"
                      class="px-3 py-1.5 text-sm font-medium bg-red-500 text-white rounded-lg hover:bg-red-600"
                    >
                      Hapus
                    </button>
                  </div>
                </div>
              </div>

              <!-- Empty state -->
              <div v-if="pertanyaanList.length === 0" class="text-center py-16">
                <p class="text-lg font-medium text-black">
                  Belum ada pertanyaan
                </p>
                <p class="text-sm text-black mt-1">
                  Klik tombol "Tambah Pertanyaan" untuk membuat pertanyaan baru
                </p>
              </div>
            </div>

            <!-- Modal Tambah -->
            <div
              v-if="tambahPertanyaan"
              class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
              @click.self="closeTambahPertanyaan"
            >
              <div
                class="bg-white rounded-xl w-full max-w-lg max-h-[90vh] overflow-y-auto shadow-lg"
              >
                <!-- Header Modal -->
                <div
                  class="flex justify-between items-center p-5 border-b border-gray-200"
                >
                  <h2 class="text-lg font-bold text-black">
                    Tambah Pertanyaan
                  </h2>
                  <button
                    @click="closeTambahPertanyaan"
                    class="p-2 hover:bg-gray-100 rounded-lg"
                  >
                    <XMarkIcon class="w-5 h-5 text-black" />
                  </button>
                </div>

                <div class="p-5 space-y-4">
                  <!-- Label Pertanyaan -->
                  <div>
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Label Pertanyaan</label
                    >
                    <input
                      type="text"
                      v-model="formLabel"
                      placeholder="Contoh: Nama Lengkap"
                      class="w-full p-3 border border-gray-300 rounded-lg text-sm text-black focus:outline-none focus:border-[#6444C6]"
                    />
                  </div>

                  <!-- Tipe Jawaban -->
                  <div>
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Tipe Jawaban</label
                    >
                    <div class="relative">
                      <select
                        v-model="formTipe"
                        class="w-full p-3 border border-gray-300 rounded-lg text-sm text-black appearance-none focus:outline-none focus:border-[#6444C6]"
                      >
                        <option value="text">Input Text</option>
                        <option value="multiple">Pilihan Ganda</option>
                        <option value="date">Tanggal</option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-3.5 w-4 h-4 text-black pointer-events-none"
                      />
                    </div>
                  </div>

                  <!-- Multiple Choice Options -->
                  <div
                    v-if="formTipe === 'multiple'"
                    class="bg-gray-50 rounded-lg p-4 border border-gray-200"
                  >
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Daftar Opsi Pilihan</label
                    >
                    <div class="space-y-2">
                      <div
                        v-for="(opt, i) in formPilihanList"
                        :key="i"
                        class="flex gap-2"
                      >
                        <input
                          type="text"
                          v-model="formPilihanList[i]"
                          :placeholder="'Opsi ' + (i + 1)"
                          class="flex-1 p-2 border border-gray-300 rounded-lg text-sm text-black focus:outline-none focus:border-[#6444C6]"
                        />
                        <button
                          v-if="formPilihanList.length > 1"
                          @click="formPilihanList.splice(i, 1)"
                          class="px-3 text-red-500 hover:bg-red-50 rounded-lg text-lg"
                        >
                          ×
                        </button>
                      </div>
                      <button
                        @click="formPilihanList.push('')"
                        class="text-sm text-[#6444C6] font-medium hover:underline"
                      >
                        + Tambah opsi
                      </button>
                    </div>
                  </div>

                  <!-- Actions -->
                  <div class="flex gap-3 pt-3">
                    <button
                      @click="closeTambahPertanyaan"
                      class="flex-1 py-2.5 border border-gray-300 text-black rounded-lg text-sm font-medium hover:bg-gray-50"
                    >
                      Batal
                    </button>
                    <button
                      @click="simpanPertanyaan"
                      class="flex-1 py-2.5 bg-[#6444C6] text-white rounded-lg text-sm font-medium hover:bg-[#5c3db8]"
                    >
                      Simpan
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <!-- Modal Edit -->
            <div
              v-if="editPertanyaan"
              class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
              @click.self="closeEditPertanyaan"
            >
              <div
                class="bg-white rounded-xl w-full max-w-lg max-h-[90vh] overflow-y-auto shadow-lg"
              >
                <!-- Header Modal -->
                <div
                  class="flex justify-between items-center p-5 border-b border-gray-200"
                >
                  <h2 class="text-lg font-bold text-black">Edit Pertanyaan</h2>
                  <button
                    @click="closeEditPertanyaan"
                    class="p-2 hover:bg-gray-100 rounded-lg"
                  >
                    <XMarkIcon class="w-5 h-5 text-black" />
                  </button>
                </div>

                <div class="p-5 space-y-4">
                  <!-- Label Pertanyaan -->
                  <div>
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Label Pertanyaan</label
                    >
                    <input
                      type="text"
                      v-model="formEditLabel"
                      placeholder="Contoh: Nama Lengkap"
                      class="w-full p-3 border border-gray-300 rounded-lg text-sm text-black focus:outline-none focus:border-[#6444C6]"
                    />
                  </div>

                  <!-- Tipe Jawaban -->
                  <div>
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Tipe Jawaban</label
                    >
                    <div class="relative">
                      <select
                        v-model="formEditTipe"
                        class="w-full p-3 border border-gray-300 rounded-lg text-sm text-black appearance-none focus:outline-none focus:border-[#6444C6]"
                      >
                        <option value="text">Input Text</option>
                        <option value="multiple">Pilihan Ganda</option>
                        <option value="date">Tanggal</option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-3.5 w-4 h-4 text-black pointer-events-none"
                      />
                    </div>
                  </div>

                  <!-- Multiple Choice Options -->
                  <div
                    v-if="formEditTipe === 'multiple'"
                    class="bg-gray-50 rounded-lg p-4 border border-gray-200"
                  >
                    <label class="block text-sm font-semibold text-black mb-2"
                      >Daftar Opsi Pilihan</label
                    >
                    <div class="space-y-2">
                      <div
                        v-for="(opt, i) in formEditPilihanList"
                        :key="i"
                        class="flex gap-2"
                      >
                        <input
                          type="text"
                          v-model="formEditPilihanList[i]"
                          :placeholder="'Opsi ' + (i + 1)"
                          class="flex-1 p-2 border border-gray-300 rounded-lg text-sm text-black focus:outline-none focus:border-[#6444C6]"
                        />
                        <button
                          v-if="formEditPilihanList.length > 1"
                          @click="formEditPilihanList.splice(i, 1)"
                          class="px-3 text-red-500 hover:bg-red-50 rounded-lg text-lg"
                        >
                          ×
                        </button>
                      </div>
                      <button
                        @click="formEditPilihanList.push('')"
                        class="text-sm text-[#6444C6] font-medium hover:underline"
                      >
                        + Tambah opsi
                      </button>
                    </div>
                  </div>

                  <!-- Actions -->
                  <div class="flex gap-3 pt-3">
                    <button
                      @click="closeEditPertanyaan"
                      class="flex-1 py-2.5 border border-gray-300 text-black rounded-lg text-sm font-medium hover:bg-gray-50"
                    >
                      Batal
                    </button>
                    <button
                      @click="simpanEditPertanyaan"
                      class="flex-1 py-2.5 bg-[#6444C6] text-white rounded-lg text-sm font-medium hover:bg-[#5c3db8]"
                    >
                      Update
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  </div>
</template>
