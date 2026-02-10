<script>
import { ref } from "vue";
import Footer from "../bar/footer.vue";
import Header from "../bar/header.vue";
import { CheckIcon } from "@heroicons/vue/24/solid";

export default {
  components: {
    Footer,
    Header,
    CheckIcon,
  },
  setup() {
    const s = ref(false);
    const formData = ref({
      inputText: "",
      vehicleType: "",
      date: "",
      time: "",
      agreeTerms: false,
    });
    return { s, formData };
  },
};
</script>

<template>
  <div class="min-h-screen flex flex-col font-['Montserrat'] bg-zinc-50">
    <Header/>
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
            class="p-4 md:p-8 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl mx-auto text-left"
          >
            <h1
              class="font-left text-xl sm:text-2xl font-bold mb-1 text-black leading-tight"
            >
              Selamat datang
            </h1>
            <p class="text-zinc-800 text-xs sm:text-sm leading-snug">
              Mohon mengisi formulir permintaan layanan kendaraan hari ini sebelum melakukan peminjaman atau menggunakan kendaraan layanan pool maupun kendaraan operasional dengan izin khusus PT Indominco Mandiri.
            </p>
          </div>

          <!-- KONTEN PERTANYAAN AKTIF -->

          <!-- Konten Input text -->
          <div
            class="p-4 md:p-6 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl text-left">
            <h3
              class="font-left text-xl sm:text-xl font-bold mb-2 text-black leading-tight">
              Input Text
            </h3>
            <div class="h-0.5 w-full bg-[#7800b5] rounded-lg mb-4"></div>
            <input
              v-model="formData.inputText"
              type="text"
              placeholder="Input text"
              class="w-full px-3 py-2 border border-zinc-300 rounded-lg focus:outline-none focus:border-purple-600 text-sm text-black"
            />
          </div>

          <!-- Konten Multiple Choice -->
           <div
            class="p-4 md:p-6 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl text-left">
            <h3
              class="font-left text-xl sm:text-xl font-bold mb-2 text-black leading-tight">
              Multiple Choice
            </h3>
            <div class="h-0.5 w-full bg-[#7800b5] rounded-lg mb-4"></div>
            <div class="flex flex-col gap-3">
              <label class="flex items-center gap-2 cursor-pointer">
                <input
                  v-model="formData.vehicleType"
                  type="radio"
                  name="vehicleType"
                  value="pool"
                  class="w-4 h-4 cursor-pointer accent-purple-600 appearance-none border-2 border-gray-400 rounded-full bg-gray-200 checked:bg-purple-600 checked:border-purple-600"
                />
                <span class="text-sm text-zinc-700">Satu</span>
              </label>
              <label class="flex items-center gap-2 cursor-pointer">
                <input
                  v-model="formData.vehicleType"
                  type="radio"
                  name="vehicleType"
                  value="operasional"
                  class="w-4 h-4 cursor-pointer accent-purple-600 appearance-none border-2 border-gray-400 rounded-full bg-gray-200 checked:bg-purple-600 checked:border-purple-600"
                />
                <span class="text-sm text-zinc-700">Dua</span>
              </label>
            </div>
          </div>

          <!-- Konten Date -->
          <div
            class="p-4 md:p-6 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl text-left">
            <h3
              class="font-left text-xl sm:text-xl font-bold mb-2 text-black leading-tight">
              Tanggal dan Waktu
            </h3>
            <div class="h-0.5 w-full bg-[#7800b5] rounded-lg mb-4"></div>
            <div class="flex flex-col sm:flex-column gap-3">
                <p class="text-sm text-zinc-700">Tanggal : </p>
              <input
                v-model="formData.date"
                type="datetime-local"
                class="flex-1 px-3 py-2 border border-zinc-300 rounded-lg focus:outline-none focus:border-purple-600 text-sm"
              />
              <p class="text-sm text-zinc-700">Tanggal : </p>
              <input
                v-model="formData.time"
                type="datetime-local"
                class="flex-1 px-3 py-2 border border-zinc-300 rounded-lg focus:outline-none focus:border-purple-600 text-sm"
              />
            </div>
          </div>

          <!-- Konten Ketentuan -->
          <div
            class="p-4 md:p-8 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl mx-auto text-left">
            <h1
              class="font-left text-xl sm:text-2xl font-bold mb-1 text-black leading-tight">
              Ketentuan :
            </h1>
            <ul class="flex flex-col gap-2 mt-2">
              <li
                class="text-zinc-800 text-xs sm:text-sm leading-snug list-disc ml-5">
                Pengguna bertanggung jawab atas segala kerusakan, kehilangan, pelanggaran lalu lintas, maupun klaim dari pihak ketiga apabila terjadi kecelakaan.
              </li>
              <li
                class="text-zinc-800 text-xs sm:text-sm leading-snug list-disc ml-5">
                Penggunaan kendaraan untuk kepentingan pribadi, termasuk biaya bahan bakar (BBM), menjadi tanggung jawab pribadi pengguna.
              </li>
              <li
                class="text-zinc-800 text-xs sm:text-sm leading-snug list-disc ml-5">
                Kendaraan tidak diperkenankan dioperasikan oleh pihak yang tidak memiliki izin resmi atau SIMPER yang masih berlaku.
              </li>
            </ul>
          </div>

          <!-- Konten Checkbox -->
          <div
            class="p-4 md:p-6 bg-white rounded-2xl shadow-sm border border-zinc-200 max-w-3xl text-left">
            <h3
              class="font-left text-xl sm:text-xl font-bold mb-2 text-black leading-tight">
              Persetujuan
            </h3>
            <div class="h-0.5 w-full bg-[#7800b5] rounded-lg mb-4"></div>
            <label class="flex items-center gap-3 cursor-pointer">
              <div class="relative mt-1">
                <input
                  v-model="formData.agreeTerms"
                  type="checkbox"
                  class="w-5 h-5 cursor-pointer appearance-none border-2 border-gray-300 rounded bg-white checked:bg-green-600 checked:border-green-600 transition-colors"
                />
                <CheckIcon v-if="formData.agreeTerms" class="w-4 h-4 text-white absolute top-0.5 left-0.5 pointer-events-none" />
              </div>
              <span class="text-xs sm:text-sm text-center text-zinc-700 leading-relaxed">
                Saya telah membaca dan menyetujui semua ketentuan yang berlaku
              </span>
            </label>
          </div>

          <!-- Tombol Submit -->
          <div class="flex justify-end mb-2">
            <button
              @click=""
              :disabled="s"
              class="px-10 sm:px-10 py-2.5 sm:py-3 bg-purple-600 hover:bg-purple-700 text-white font-bold text-xs sm:text-sm rounded-lg shadow-md transition-all active:scale-95 disabled:bg-zinc-400 disabled:shadow-sm flex items-center justify-center w-auto sm:w-auto"
            >
              Kirim
            </button>
          </div>
        </div>
      </div>
    </div>
    <Footer />
  </div>
</template>
