<script>
import { ref } from "vue";
import Footer from "../bar/footer.vue";
import Header from "../bar/header.vue";
import { MagnifyingGlassIcon, CheckIcon, XMarkIcon } from "@heroicons/vue/24/outline";

export default {
  components: {
    Footer,
    Header,
    MagnifyingGlassIcon,
    CheckIcon,
    XMarkIcon
  },
  setup() {
    const s = ref("");
    
    const tiketDiterima = ref([
      {
        id: "GA - TR - 1",
        keperluan: "Perjalanan Bisnis",
        nama: "John Doe",
        waktuPeminjaman: "12/12/2026, 12:00:00",
        waktuSelesai: "13/12/2026, 12:00:00",
      },
    ]);

    const tiketDitolak = ref([
      {
        id: "GA - TR - 2",
        keperluan: "Perjalanan Pribadi",
        nama: "Bob Wilson",
        waktuPeminjaman: "14/12/2026, 10:00:00",
        waktuSelesai: "14/12/2026, 16:00:00",
        alasan: "Kendaraan tidak tersedia",
      },
    ]);

    const hasilPencarian = ref([]);
    const statusPencarian = ref(""); // 'diterima', 'ditolak', atau ''

    const cariTiket = () => {
      if (!s.value.trim()) {
        hasilPencarian.value = [];
        statusPencarian.value = "";
        return;
      }

      const searchTerm = s.value.trim().toUpperCase();

      // Cari di tiket diterima
      const diterimaDitemukan = tiketDiterima.value.filter(
        (tiket) => tiket.id.toUpperCase().includes(searchTerm)
      );

      // Cari di tiket ditolak
      const ditolakDitemukan = tiketDitolak.value.filter(
        (tiket) => tiket.id.toUpperCase().includes(searchTerm)
      );

      if (diterimaDitemukan.length > 0) {
        hasilPencarian.value = diterimaDitemukan;
        statusPencarian.value = "diterima";
      } else if (ditolakDitemukan.length > 0) {
        hasilPencarian.value = ditolakDitemukan;
        statusPencarian.value = "ditolak";
      } else {
        hasilPencarian.value = [];
        statusPencarian.value = "tidak_ditemukan";
      }
    };

    return { s, tiketDiterima, tiketDitolak, hasilPencarian, statusPencarian, cariTiket };
  },
};
</script>

<template>
  <div class="min-h-screen flex flex-col font-['Montserrat'] bg-zinc-50">
    <Header />
    <div
      class="w-screen h-screen fixed inset-0 flex items-center justify-center bg-center bg-cover bg-no-repeat"
      style="background-image: url(&quot;/image_asset/backgrond.png&quot;)"
    >
      <div
        class="w-full flex flex-col gap-3 sm:gap-4 max-h-screen overflow-y-auto px-4 py-8 pr-2 pt-20 pb-30"
      >
        <div
          class="max-w-4xl justify-center mx-auto flex flex-col gap-2 sm:gap-4"
        >
          <!-- Konten Pengumuman -->
          <div
            class="p-4 md:p-8 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl mx-auto text-left">
            <h1
              class="font-left text-xl sm:text-2xl font-bold mb-1 text-black leading-tight">
              Tiket Kendaraan
            </h1>
            <p class="text-zinc-800 text-xs sm:text-sm leading-snug">
              Silakan cari tiket Anda melalui kolom pencarian di bawah ini menggunakan nama tiket yang telah dikirimkan ke email Gmail Anda.
            </p>
          </div>

          <!-- Konten Pencarian -->
           <div
            class="p-4 md:p-6 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl text-left">
            <h3
              class="font-left text-xl sm:text-xl font-bold mb-3 text-black leading-tight">
              Tujuan Perjalanan
            </h3>
            <div class="h-0.5 w-full bg-[#7800b5] rounded-lg mb-4"></div>
            <div class="flex flex-col gap-3">
                <div class="flex flex-row gap-3">
                    <input
                    v-model="s"
                    type="text"
                    placeholder="Masukkan tiket"
                    class="w-full px-3 py-2 border border-zinc-300 rounded-lg focus:outline-none focus:border-purple-600 text-sm text-black"
                    />
                        
                    <button
                    @click="cariTiket"
                    :disabled="!s"
                    class="px-5 sm:px-5 py-2 sm:py-2 bg-purple-600 hover:bg-purple-700 text-white font-bold text-base sm:text-base rounded-lg shadow-md transition-all flex items-center justify-center w-auto sm:w-auto disabled:bg-gray-400 disabled:cursor-not-allowed">
                    Cari
                    <MagnifyingGlassIcon class="ml-4 shrink-0 w-5 h-5 text-[#ffffff]" />
                    </button>
                </div>

                <!-- Konten Tiket Terima -->
                <div class="flex flex-col gap-3" v-if="hasilPencarian.length > 0">
                    <!-- Tiket Diterima -->
                    <div v-if="statusPencarian === 'diterima'" v-for="tiket in hasilPencarian" :key="tiket.id" class="p-4 md:p-4 bg-[#C4FFC0] rounded-2xl shadow-sm border border-[#1AA13C] max-w-3xl text-left">
                        <div class="flex flex-row gap-3 items-center">
                            <div class="flex flex-col gap-1 flex-1">
                                <span class="text-sm font-bold text-black">Nomor Tiket : {{ tiket.id }}</span>
                                <span class="text-xs text-black">Keperluan : {{ tiket.keperluan }}</span>
                                <span class="text-xs text-black">Nama: {{ tiket.nama }}</span>
                                <span class="text-xs text-black">Waktu Peminjaman : {{ tiket.waktuPeminjaman }}</span>
                                <span class="text-xs text-black">Waktu Selesai :  {{ tiket.waktuSelesai }}</span>
                            </div>
                            <div class="ml-auto flex items-center flex-col gap-1">
                                <div class="px-4 py-2 bg-green-600 text-white font-bold text-sm rounded-lg shadow-md">
                                    <CheckIcon class="w-10 h-10 text-white mx-auto mb-1"/>
                                    <p class="text-xs">Disetujui</p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Tiket Ditolak -->
                    <div v-if="statusPencarian === 'ditolak'" v-for="tiket in hasilPencarian" :key="tiket.id" class="p-4 md:p-4 bg-[#FFC4C4] rounded-2xl shadow-sm border border-[#C41E1E] max-w-3xl text-left">
                        <div class="flex flex-row gap-3 items-center">
                            <div class="flex flex-col gap-1 flex-1">
                                <span class="text-sm font-bold text-black">Nomor Tiket : {{ tiket.id }}</span>
                                <span class="text-xs text-black">Keperluan : {{ tiket.keperluan }}</span>
                                <span class="text-xs text-black">Nama: {{ tiket.nama }}</span>
                                <span class="text-xs text-black">Waktu Peminjaman : {{ tiket.waktuPeminjaman }}</span>
                                <span class="text-xs text-black">Waktu Selesai :  {{ tiket.waktuSelesai }}</span>
                                <span class="text-xs text-red-700 font-semibold">Alasan Penolakan : {{ tiket.alasan }}</span>
                            </div>
                            <div class="ml-auto flex items-center flex-col gap-1">
                                <div class="px-4 py-2 bg-red-600 text-white font-bold text-sm rounded-lg shadow-md">
                                  <XMarkIcon class="w-10 h-10 text-white mx-auto mb-1"/>
                                    <p class="text-xs">Ditolak</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Pesan Tidak Ditemukan -->
                <div v-if="statusPencarian === 'tidak_ditemukan'" class="p-4 md:p-6 bg-yellow-50 rounded-2xl shadow-sm border border-yellow-300 max-w-3xl text-center">
                    <p class="text-sm font-semibold text-yellow-800">Tiket dengan nomor "{{ s }}" tidak ditemukan</p>
                </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer />
  </div>
</template>
