<script setup>
import { ref, provide, computed } from "vue";
import Aside from "../bar/aside.vue";
import HeaderAdmin from "../bar/header-admin.vue";
import {
  MagnifyingGlassIcon,
  Bars3BottomLeftIcon,
  ArrowUpTrayIcon,
  TrashIcon,
  CheckIcon,
  CheckCircleIcon,
  XCircleIcon,
  XMarkIcon,
  ChevronDownIcon,
} from "@heroicons/vue/24/outline";

const isMobileMenuOpen = ref(false);
const searchQuery = ref("");
const currentPage = ref(1);
const selectedRowIds = ref([]);
const itemsPerPage = 10;
const showFilter = ref(false);
const filterDate = ref({
  startDate: "",
  endDate: "",
});
const filterPersetujuan = ref({
  Head: false,
  Transportasi: false,
  GA: false,
  GS: false,
});
const filterDepartemen = ref("");
const filterLokasi = ref("");
const filterKeperluan = ref("");
const filterLayanan = ref("");

// State untuk Approval Modals
const showPersetujuanTerima = ref(false);
const showPersetujuanTolak = ref(false);
const approvalContext = ref({
  type: null, // 'head', 'trans', 'ga', 'gs'
  action: null, // 'approve', 'reject'
  row: null,
});

// State untuk Detail Modal
const showDetail = ref(false);
const currentDetailRow = ref(null);

const approvalTypeLabel = computed(() => {
  const labels = {
    head: "Head Departemen",
    trans: "Transportasi",
    ga: "GA",
    gs: "GS",
  };
  return labels[approvalContext.value.type] || "";
});

const approvalActionLabel = computed(() => {
  return approvalContext.value.action === "approve"
    ? "Persetujuan"
    : "Penolakan";
});

// Mock data untuk testing
const tableData = ref([
  {
    id: 1,
    nomorTiket: "GA-TR-01",
    nik: "12345678901234567890",
    nama: "John Doe",
    email: "john.doe@company.com",
    departemen: "IT",
    lokasi: "Desabinaan",
    keperluan: "Dinas",
    layanan: "Izin Khusus Kendaraan operasional",
    waktuPeminjaman: "2024-01-21 10:30:45",
    waktuSelesai: "2024-01-21 12:30:45",
    headApproval: null,
    headApprovalTime: "",
    transApproval: null,
    transApprovalTime: "",
    gaApproval: null,
    gaApprovalTime: "",
    gsApproval: null,
    gsApprovalTime: "",
  },
]);

// Filtered data based on search and filters
const filteredTableData = computed(() => {
  let filtered = tableData.value;

  // Apply search filter
  if (searchQuery.value) {
    filtered = filtered.filter(
      (item) =>
        item.nama.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        item.nomorTiket.toLowerCase().includes(searchQuery.value.toLowerCase()),
    );
  }

  // Apply date range filter
  if (filterDate.value.startDate || filterDate.value.endDate) {
    filtered = filtered.filter((item) => {
      const itemDate = new Date(item.waktuPeminjaman.split(" ")[0]);
      if (filterDate.value.startDate) {
        const startDate = new Date(filterDate.value.startDate);
        if (itemDate < startDate) return false;
      }
      if (filterDate.value.endDate) {
        const endDate = new Date(filterDate.value.endDate);
        if (itemDate > endDate) return false;
      }
      return true;
    });
  }

  // Apply persetujuan filter
  const hasPersetujuanFilter =
    filterPersetujuan.value.Head ||
    filterPersetujuan.value.Transportasi ||
    filterPersetujuan.value.GA ||
    filterPersetujuan.value.GS;

  if (hasPersetujuanFilter) {
    filtered = filtered.filter((item) => {
      const matchHead =
        !filterPersetujuan.value.Head || item.headApproval === true;
      const matchTrans =
        !filterPersetujuan.value.Transportasi || item.transApproval === true;
      const matchGA = !filterPersetujuan.value.GA || item.gaApproval === true;
      const matchGS = !filterPersetujuan.value.GS || item.gsApproval === true;

      return matchHead && matchTrans && matchGA && matchGS;
    });
  }

  // Apply departemen filter
  if (filterDepartemen.value) {
    filtered = filtered.filter(
      (item) => item.departemen === filterDepartemen.value,
    );
  }

  // Apply lokasi filter
  if (filterLokasi.value) {
    filtered = filtered.filter((item) => item.lokasi === filterLokasi.value);
  }

  // Apply keperluan filter
  if (filterKeperluan.value) {
    filtered = filtered.filter(
      (item) => item.keperluan === filterKeperluan.value,
    );
  }

  // Apply layanan filter
  if (filterLayanan.value) {
    filtered = filtered.filter((item) => item.layanan === filterLayanan.value);
  }

  return filtered;
});

// Paginated data
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredTableData.value.slice(start, end);
});

// Total pages
const totalPages = computed(() => {
  return Math.ceil(filteredTableData.value.length / itemsPerPage);
});

// Start and end index for pagination display
const startIndex = computed(() => {
  return (currentPage.value - 1) * itemsPerPage + 1;
});

const endIndex = computed(() => {
  return Math.min(
    currentPage.value * itemsPerPage,
    filteredTableData.value.length,
  );
});

// Select all checkbox
const selectAllChecked = computed(() => {
  if (paginatedData.value.length === 0) return false;
  return paginatedData.value.every((item) =>
    selectedRowIds.value.includes(item.id),
  );
});

// Helper functions
const isRowSelected = (id) => {
  return selectedRowIds.value.includes(id);
};

const selectRow = (id) => {
  if (isRowSelected(id)) {
    selectedRowIds.value = selectedRowIds.value.filter((rowId) => rowId !== id);
  } else {
    selectedRowIds.value.push(id);
  }
};

const toggleSelectAll = () => {
  if (selectAllChecked.value) {
    paginatedData.value.forEach((item) => {
      selectedRowIds.value = selectedRowIds.value.filter(
        (id) => id !== item.id,
      );
    });
  } else {
    paginatedData.value.forEach((item) => {
      if (!isRowSelected(item.id)) {
        selectedRowIds.value.push(item.id);
      }
    });
  }
};

const handleApprovalClick = (row, type, action) => {
  approvalContext.value = {
    type: type,
    action: action,
    row: row,
  };

  if (action === "approve") {
    showPersetujuanTerima.value = true;
  } else {
    showPersetujuanTolak.value = true;
  }
};

const confirmApproval = () => {
  const row = approvalContext.value.row;
  const type = approvalContext.value.type;
  const action = approvalContext.value.action;
  const now = new Date();
  const timestamp = now.toISOString().slice(0, 16); // Format: YYYY-MM-DDTHH:mm

  if (type === "head") {
    row.headApproval = action === "approve" ? true : false;
    row.headApprovalTime = timestamp;
  } else if (type === "trans") {
    row.transApproval = action === "approve" ? true : false;
    row.transApprovalTime = timestamp;
  } else if (type === "ga") {
    row.gaApproval = action === "approve" ? true : false;
    row.gaApprovalTime = timestamp;
  } else if (type === "gs") {
    row.gsApproval = action === "approve" ? true : false;
    row.gsApprovalTime = timestamp;
  }

  closeApprovalModals();
};

const closeApprovalModals = () => {
  showPersetujuanTerima.value = false;
  showPersetujuanTolak.value = false;
  approvalContext.value = {
    type: null,
    action: null,
    row: null,
  };
};

const openDetail = (row) => {
  currentDetailRow.value = { ...row };
  showDetail.value = true;
};

const closeDetail = () => {
  showDetail.value = false;
  currentDetailRow.value = null;
};

const handleEdit = (id) => {
  console.log("Edit item:", id);
};

const previousPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

// Function untuk toggle mobile menu dari header
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
};

// Function untuk membuka filter
const openFilter = () => {
  showFilter.value = true;
};

const closeFilter = () => {
  showFilter.value = false;
};

const applyFilter = () => {
  // Reset to first page when filter is applied
  currentPage.value = 1;
  showFilter.value = false;
};

const resetFilter = () => {
  filterDate.value.startDate = "";
  filterDate.value.endDate = "";
  filterPersetujuan.value.Head = false;
  filterPersetujuan.value.Transportasi = false;
  filterPersetujuan.value.GA = false;
  filterPersetujuan.value.GS = false;
  filterDepartemen.value = "";
  filterLokasi.value = "";
  filterKeperluan.value = "";
  filterLayanan.value = "";
  currentPage.value = 1;
};

// Get unique values from table data for dropdown options
const uniqueDepartemen = computed(() => {
  return [...new Set(tableData.value.map((item) => item.departemen))];
});

const uniqueLokasi = computed(() => {
  return [...new Set(tableData.value.map((item) => item.lokasi))];
});

const uniqueKeperluan = computed(() => {
  return [...new Set(tableData.value.map((item) => item.keperluan))];
});

const uniqueLayanan = computed(() => {
  return [...new Set(tableData.value.map((item) => item.layanan))];
});

// Provide untuk digunakan di header
provide("isMobileMenuOpen", isMobileMenuOpen);
provide("toggleMobileMenu", toggleMobileMenu);
</script>

<template>
  <div class="h-screen flex flex-col font-['Montserrat']">
    <div class="flex flex-1 overflow-hidden">
      <!-- Aside Sidebar - Push content style -->
      <Aside />

      <!-- Main Content Area -->
      <div class="flex flex-col flex-1 min-w-0 overflow-hidden">
        <HeaderAdmin />

        <!-- Content -->
        <main class="bg-[#EFEFEF] flex-1 flex flex-col p-3 overflow-hidden">
          <div
            class="bg-white rounded-lg shadow-md p-5 flex-1 flex flex-col overflow-y-auto"
          >
            <!-- Toolbar -->
            <div
              class="flex items-center gap-3 pb-4 border-b shrink-0 flex-none justify-end"
            >
              <!-- Search Input with Icon Inside -->
              <div class="relative flex min-w-56">
                <MagnifyingGlassIcon
                  class="w-4 h-4 text-gray-400 absolute left-3 top-1/2 -translate-y-1/2 pointer-events-none"
                />
                <input
                  v-model="searchQuery"
                  @input="currentPage = 1"
                  type="text"
                  placeholder="Cari..."
                  class="w-full pl-9 pr-3 py-2 border border-gray-300 rounded-md text-sm text-gray-600 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
              </div>

              <!-- Filter Button -->
              <button
                @click="openFilter"
                class="flex items-center gap-2 px-3 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50 transition text-sm"
              >
                <Bars3BottomLeftIcon class="w-4 h-4" />
                <span>Filter</span>
              </button>

              <!-- Export Button -->
              <button
                class="flex items-center gap-2 px-3 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50 transition text-sm"
              >
                <ArrowUpTrayIcon class="w-4 h-4" />
                <span>Export</span>
              </button>

              <!-- Delete Button -->
              <button
                :disabled="selectedRowIds.length === 0"
                class="flex items-center gap-2 px-3 py-2 rounded-md transition text-sm"
                :class="
                  selectedRowIds.length > 0
                    ? 'bg-red-100 text-red-700 border border-red-300 hover:bg-red-200'
                    : 'bg-gray-100 text-gray-400 border border-gray-300 cursor-not-allowed'
                "
              >
                <TrashIcon class="w-4 h-4" />
                <span>Hapus</span>
              </button>
            </div>
            <!-- Tabel -->
            <div
              class="flex flex-col gap-4 bg-gray-50 p-1 rounded-lg border border-gray-200 min-h-0"
            >
              <div class="overflow-auto rounded-lg border bg-white max-h-full">
                <table class="w-full border-collapse">
                  <thead>
                    <tr class="border-b-2 border-gray-400 bg-gray-50">
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center font-semibold text-gray-700 whitespace-nowrap min-w-12 border-r border-gray-300"
                      >
                        <div class="relative w-5 h-5">
                          <input
                            type="checkbox"
                            :checked="selectAllChecked"
                            @change="toggleSelectAll"
                            class="w-5 h-5 cursor-pointer rounded-md border-2 appearance-none bg-white border-gray-600 checked:bg-blue-500 checked:border-blue-500"
                            style="
                              appearance: none;
                              -webkit-appearance: none;
                              -moz-appearance: none;
                            "
                            title="Pilih semua / Batal pilih semua"
                          />
                          <!-- Check Icon -->
                          <CheckIcon
                            v-if="selectAllChecked"
                            class="absolute inset-0 m-auto w-3.5 h-3.5 text-white pointer-events-none"
                          />
                        </div>
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-24 border-r border-gray-300"
                      >
                        Nomor Tiket
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-28 border-r border-gray-300"
                      >
                        NIK
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-28 border-r border-gray-300"
                      >
                        Nama
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-32 border-r border-gray-300"
                      >
                        Email
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-24 border-r border-gray-300"
                      >
                        Departemen
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-24 border-r border-gray-300"
                      >
                        Lokasi
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-20 border-r border-gray-300"
                      >
                        Keperluan
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-20 border-r border-gray-300"
                      >
                        Layanan
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-40 border-r border-gray-300"
                      >
                        Waktu Peminjaman
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-40 border-r border-gray-300"
                      >
                        Waktu Selesai
                      </th>
                      <th
                        colspan="4"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-full border-b border-gray-300 border-r"
                      >
                        Persetujuan
                      </th>
                      <th
                        rowspan="2"
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-16"
                      >
                        Detail
                      </th>
                    </tr>
                    <tr class="border-b-2 border-gray-400 bg-gray-50">
                      <th
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-20 border-r border-gray-300"
                      >
                        Head Dept
                      </th>
                      <th
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-24 border-r border-gray-300"
                      >
                        Transportasi
                      </th>
                      <th
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-20 border-r border-gray-300"
                      >
                        GA
                      </th>
                      <th
                        class="px-4 py-3 text-center text-sm font-semibold text-gray-700 whitespace-nowrap min-w-20 border-r border-gray-300"
                      >
                        GS
                      </th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr
                      v-for="row in paginatedData"
                      :key="row.id"
                      class="border-b border-gray-200 hover:bg-gray-50 transition cursor-pointer"
                      :class="{ 'bg-blue-50': isRowSelected(row.id) }"
                    >
                      <td
                        class="px-4 py-3 whitespace-nowrap min-w-12 text-center border-r border-gray-300"
                      >
                        <div class="relative w-5 h-5 mx-auto">
                          <input
                            type="checkbox"
                            :checked="isRowSelected(row.id)"
                            @change="selectRow(row.id)"
                            @click.stop
                            class="w-5 h-5 cursor-pointer rounded-md border-2 appearance-none bg-white border-gray-600 checked:bg-blue-500 checked:border-blue-500"
                          />
                          <!-- Check Icon -->
                          <CheckIcon
                            v-if="isRowSelected(row.id)"
                            class="absolute inset-0 m-auto w-3.5 h-3.5 text-white pointer-events-none"
                          />
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-24 text-center border-r border-gray-300"
                      >
                        {{ row.nomorTiket }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-28 text-center border-r border-gray-300"
                      >
                        {{ row.nik }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-28 text-center border-r border-gray-300"
                      >
                        {{ row.nama }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-32 text-center border-r border-gray-300"
                      >
                        {{ row.email }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-24 text-center border-r border-gray-300"
                      >
                        {{ row.departemen }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-24 text-center border-r border-gray-300"
                      >
                        <span
                          class="px-2 py-1 rounded-md text-xs font-semibold"
                          :class="
                            row.lokasi === 'Desabinaan'
                              ? 'bg-green-100 text-green-700'
                              : 'bg-blue-100 text-blue-700'
                          "
                        >
                          {{ row.lokasi }}
                        </span>
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <span
                          class="px-2 py-1 rounded-md text-xs font-semibold"
                          :class="
                            row.keperluan === 'Dinas'
                              ? 'bg-blue-100 text-blue-700'
                              : 'bg-green-100 text-green-700'
                          "
                        >
                          {{ row.keperluan }}
                        </span>
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <span
                          class="px-2 py-1 rounded-md text-xs font-semibold"
                          :class="
                            row.layanan === 'Izin Khusus Kendaraan operasional'
                              ? 'bg-blue-100 text-blue-700'
                              : 'bg-green-100 text-green-700'
                          "
                        >
                          {{ row.layanan }}
                        </span>
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-40 text-center border-r border-gray-300"
                      >
                        {{ row.waktuPeminjaman }}
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-40 text-center border-r border-gray-300"
                      >
                        {{ row.waktuSelesai }}
                      </td>
                      <td
                        class="px-4 py-3 whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <div class="flex items-center justify-center gap-2">
                          <button
                            @click="handleApprovalClick(row, 'head', 'approve')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.headApproval === true
                                ? 'text-green-500 bg-green-100 border-green-500'
                                : 'text-gray-400 hover:text-green-500 border-gray-300 hover:border-green-400',
                            ]"
                            title="Approve"
                          >
                            <CheckCircleIcon class="w-6 h-6" />
                          </button>
                          <button
                            @click="handleApprovalClick(row, 'head', 'reject')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.headApproval === false
                                ? 'text-red-500 bg-red-100 border-red-500'
                                : 'text-gray-400 hover:text-red-500 border-gray-300 hover:border-red-400',
                            ]"
                            title="Decline"
                          >
                            <XCircleIcon class="w-6 h-6" />
                          </button>
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 whitespace-nowrap min-w-24 text-center border-r border-gray-300"
                      >
                        <div class="flex items-center justify-center gap-2">
                          <button
                            @click="
                              handleApprovalClick(row, 'trans', 'approve')
                            "
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.transApproval === true
                                ? 'text-green-500 bg-green-100 border-green-500'
                                : 'text-gray-400 hover:text-green-500 border-gray-300 hover:border-green-400',
                            ]"
                            title="Approve"
                          >
                            <CheckCircleIcon class="w-6 h-6" />
                          </button>
                          <button
                            @click="handleApprovalClick(row, 'trans', 'reject')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.transApproval === false
                                ? 'text-red-500 bg-red-100 border-red-500'
                                : 'text-gray-400 hover:text-red-500 border-gray-300 hover:border-red-400',
                            ]"
                            title="Decline"
                          >
                            <XCircleIcon class="w-6 h-6" />
                          </button>
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <div class="flex items-center justify-center gap-2">
                          <button
                            @click="handleApprovalClick(row, 'ga', 'approve')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.gaApproval === true
                                ? 'text-green-500 bg-green-100 border-green-500'
                                : 'text-gray-400 hover:text-green-500 border-gray-300 hover:border-green-400',
                            ]"
                            title="Approve"
                          >
                            <CheckCircleIcon class="w-6 h-6" />
                          </button>
                          <button
                            @click="handleApprovalClick(row, 'ga', 'reject')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.gaApproval === false
                                ? 'text-red-500 bg-red-100 border-red-500'
                                : 'text-gray-400 hover:text-red-500 border-gray-300 hover:border-red-400',
                            ]"
                            title="Decline"
                          >
                            <XCircleIcon class="w-6 h-6" />
                          </button>
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <div class="flex items-center justify-center gap-2">
                          <button
                            @click="handleApprovalClick(row, 'gs', 'approve')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.gsApproval === true
                                ? 'text-green-500 bg-green-100 border-green-500'
                                : 'text-gray-400 hover:text-green-500 border-gray-300 hover:border-green-400',
                            ]"
                            title="Approve"
                          >
                            <CheckCircleIcon class="w-6 h-6" />
                          </button>
                          <button
                            @click="handleApprovalClick(row, 'gs', 'reject')"
                            :class="[
                              'p-1.5 rounded-lg transition border-2',
                              row.gsApproval === false
                                ? 'text-red-500 bg-red-100 border-red-500'
                                : 'text-gray-400 hover:text-red-500 border-gray-300 hover:border-red-400',
                            ]"
                            title="Decline"
                          >
                            <XCircleIcon class="w-6 h-6" />
                          </button>
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 text-gray-800 text-xs font-medium whitespace-nowrap min-w-20 text-center border-r border-gray-300"
                      >
                        <button
                          type="button"
                          @click="openDetail(row)"
                          class="px-3 py-1 rounded-md text-xs font-semibold bg-gray-200 text-gray-700 hover:bg-gray-300 active:bg-gray-400 transition"
                        >
                          Detail
                        </button>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>

              <!-- Konten Filter -->
              <div
                v-if="showFilter"
                class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4"
              >
                <div
                  class="bg-white rounded-lg w-full max-w-md md:max-w-lg max-h-[90vh] overflow-y-auto shadow-[0_4px_6px_rgba(0,0,0,0.1)] p-6 md:p-8"
                >
                  <div
                    class="flex justify-between items-center mb-1 pb-3 border-b border-gray-200"
                  >
                    <h2 class="text-lg md:text-xl font-semibold text-gray-900">
                      Filter Data
                    </h2>
                    <button
                      @click="closeFilter"
                      class="shrink-0 p-1 hover:bg-gray-100 rounded-md transition"
                    >
                      <XMarkIcon
                        class="w-6 h-6 text-gray-600 hover:text-gray-900"
                      />
                    </button>
                  </div>

                  <!-- Filter Date -->
                  <div class="grid grid-cols-2 gap-4">
                    <div>
                      <label
                        class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                        >Waktu Peminjaman</label
                      >
                      <input
                        v-model="filterDate.startDate"
                        type="date"
                        class="w-full p-2 text-xs border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8]"
                      />
                    </div>
                    <div>
                      <label
                        class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                        >Waktu Selesai</label
                      >
                      <input
                        v-model="filterDate.endDate"
                        type="date"
                        class="w-full p-2 text-xs border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8]"
                      />
                    </div>
                  </div>

                  <!-- Filter Persetujuan -->
                  <div>
                    <label
                      class="block text-sm font-medium text-black mb-2 mt-4"
                      >Persetujuan</label
                    >
                    <div class="flex flex-wrap gap-4">
                      <div class="flex items-center">
                        <input
                          v-model="filterPersetujuan.Head"
                          type="checkbox"
                          id="head"
                          class="w-4 h-4 cursor-pointer rounded border-2 border-gray-300"
                          style="
                            appearance: none;
                            -webkit-appearance: none;
                            -moz-appearance: none;
                          "
                          :style="
                            filterPersetujuan.Head
                              ? {
                                  backgroundColor: '#3B82F6',
                                  borderColor: '#1E40AF',
                                }
                              : {
                                  backgroundColor: 'white',
                                  borderColor: '#d1d5db',
                                }
                          "
                        />
                        <label
                          for="head"
                          class="ml-2 text-sm text-black cursor-pointer"
                          >Head</label
                        >
                      </div>
                      <div class="flex items-center">
                        <input
                          v-model="filterPersetujuan.Transportasi"
                          type="checkbox"
                          id="transportasi"
                          class="w-4 h-4 cursor-pointer rounded border-2 border-gray-300"
                          style="
                            appearance: none;
                            -webkit-appearance: none;
                            -moz-appearance: none;
                          "
                          :style="
                            filterPersetujuan.Transportasi
                              ? {
                                  backgroundColor: '#8B5CF6',
                                  borderColor: '#5B21B6',
                                }
                              : {
                                  backgroundColor: 'white',
                                  borderColor: '#d1d5db',
                                }
                          "
                        />
                        <label
                          for="transportasi"
                          class="ml-2 text-sm text-black cursor-pointer"
                          >Transportasi</label
                        >
                      </div>
                      <div class="flex items-center">
                        <input
                          v-model="filterPersetujuan.GA"
                          type="checkbox"
                          id="ga"
                          class="w-4 h-4 cursor-pointer rounded border-2 border-gray-300"
                          style="
                            appearance: none;
                            -webkit-appearance: none;
                            -moz-appearance: none;
                          "
                          :style="
                            filterPersetujuan.GA
                              ? {
                                  backgroundColor: '#EC4899',
                                  borderColor: '#9D174D',
                                }
                              : {
                                  backgroundColor: 'white',
                                  borderColor: '#d1d5db',
                                }
                          "
                        />
                        <label
                          for="ga"
                          class="ml-2 text-sm text-black cursor-pointer"
                          >GA</label
                        >
                      </div>
                      <div class="flex items-center">
                        <input
                          v-model="filterPersetujuan.GS"
                          type="checkbox"
                          id="gs"
                          class="w-4 h-4 cursor-pointer rounded border-2 border-gray-300"
                          style="
                            appearance: none;
                            -webkit-appearance: none;
                            -moz-appearance: none;
                          "
                          :style="
                            filterPersetujuan.GS
                              ? {
                                  backgroundColor: '#F59E0B',
                                  borderColor: '#B45309',
                                }
                              : {
                                  backgroundColor: 'white',
                                  borderColor: '#d1d5db',
                                }
                          "
                        />
                        <label
                          for="gs"
                          class="ml-2 text-sm text-black cursor-pointer"
                          >GS</label
                        >
                      </div>
                    </div>
                  </div>

                  <!-- Departemen -->
                  <div>
                    <label
                      class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                      >Departemen</label
                    >
                    <div class="relative">
                      <select
                        v-model="filterDepartemen"
                        class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8] appearance-none"
                      >
                        <option value="">Pilih Departemen</option>
                        <option
                          v-for="dept in uniqueDepartemen"
                          :key="dept"
                          :value="dept"
                        >
                          {{ dept }}
                        </option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-2.5 w-5 h-5 text-[#949494] pointer-events-none"
                      />
                    </div>
                  </div>

                  <!-- Lokasi -->
                  <div>
                    <label
                      class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                      >Lokasi</label
                    >
                    <div class="relative">
                      <select
                        v-model="filterLokasi"
                        class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8] appearance-none"
                      >
                        <option value="">Pilih Lokasi</option>
                        <option
                          v-for="lokasi in uniqueLokasi"
                          :key="lokasi"
                          :value="lokasi"
                        >
                          {{ lokasi }}
                        </option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-2.5 w-5 h-5 text-[#949494] pointer-events-none"
                      />
                    </div>
                  </div>

                  <!-- Keperluan -->
                  <div>
                    <label
                      class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                      >Keperluan</label
                    >
                    <div class="relative">
                      <select
                        v-model="filterKeperluan"
                        class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8] appearance-none"
                      >
                        <option value="">Pilih Keperluan</option>
                        <option
                          v-for="keperluan in uniqueKeperluan"
                          :key="keperluan"
                          :value="keperluan"
                        >
                          {{ keperluan }}
                        </option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-2.5 w-5 h-5 text-[#949494] pointer-events-none"
                      />
                    </div>
                  </div>

                  <!-- Layanan -->
                  <div>
                    <label
                      class="block text-sm font-medium text-gray-800 mb-2 mt-2"
                      >Layanan</label
                    >
                    <div class="relative">
                      <select
                        v-model="filterLayanan"
                        class="w-full p-2 pr-10 text-sm border border-[#C3C3C3] bg-white text-gray-700 rounded-md focus:outline-none focus:border-[#A90CF8] appearance-none"
                      >
                        <option value="">Pilih Layanan</option>
                        <option
                          v-for="layanan in uniqueLayanan"
                          :key="layanan"
                          :value="layanan"
                        >
                          {{ layanan }}
                        </option>
                      </select>
                      <ChevronDownIcon
                        class="absolute right-3 top-2.5 w-5 h-5 text-[#949494] pointer-events-none"
                      />
                    </div>
                  </div>

                  <div class="flex justify-center gap-3 mt-6">
                    <button
                      @click="
                        () => {
                          resetFilter();
                          closeFilter();
                        }
                      "
                      class="px-6 md:px-6 py-2 text-sm md:text-base border border-gray-300 text-gray-700 rounded-xl hover:bg-gray-50 transition font-regular"
                    >
                      Reset
                    </button>
                    <button
                      @click="closeFilter"
                      class="px-6 md:px-6 py-2 text-sm md:text-base border border-gray-300 text-gray-700 rounded-xl hover:bg-gray-50 transition font-regular"
                    >
                      Batal
                    </button>
                    <button
                      @click="applyFilter"
                      class="px-6 md:px-6 py-2 text-sm md:text-base bg-linear-to-r from-[#A90CF8] to-[#9600E1] text-white rounded-xl hover:opacity-90 transition font-regular"
                    >
                      Terapkan
                    </button>
                  </div>
                </div>
              </div>

              <!-- Persetujuan Terima -->
              <div
                v-if="showPersetujuanTerima"
                class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4"
              >
                <div
                  class="bg-white rounded-lg w-full max-w-md md:max-w-lg max-h-[90vh] overflow-y-auto shadow-[0_4px_6px_rgba(0,0,0,0.1)] p-6 md:p-8"
                >
                  <div
                    class="flex justify-between items-center mb-4 pb-3 border-b border-gray-200"
                  >
                    <h2 class="text-lg md:text-xl font-semibold text-gray-900">
                      Konfirmasi {{ approvalActionLabel }}
                    </h2>
                    <button
                      @click="closeApprovalModals"
                      class="shrink-0 p-1 hover:bg-gray-100 rounded-md transition"
                    >
                      <XMarkIcon
                        class="w-6 h-6 text-gray-600 hover:text-gray-900"
                      />
                    </button>
                  </div>

                  <div class="mb-6">
                    <p class="text-gray-700 text-base">
                      Apakah sebagai
                      <span class="font-semibold text-blue-600">{{
                        approvalTypeLabel
                      }}</span>
                      anda yakin untuk melakukan
                      <span class="font-semibold text-green-600">{{
                        approvalActionLabel
                      }}</span>
                      ?
                    </p>
                  </div>

                  <div class="flex justify-end gap-3">
                    <button
                      @click="closeApprovalModals"
                      class="px-6 py-2 text-sm md:text-base border border-gray-300 text-gray-700 rounded-xl hover:bg-gray-50 transition font-regular"
                    >
                      Batal
                    </button>
                    <button
                      @click="confirmApproval"
                      class="px-6 py-2 text-sm md:text-base bg-linear-to-r from-green-500 to-green-600 text-white rounded-xl hover:opacity-90 transition font-regular"
                    >
                      Ya, Setuju
                    </button>
                  </div>
                </div>
              </div>

              <!-- Persetujuan Tolak -->
              <div
                v-if="showPersetujuanTolak"
                class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4"
              >
                <div
                  class="bg-white rounded-lg w-full max-w-md md:max-w-lg max-h-[90vh] overflow-y-auto shadow-[0_4px_6px_rgba(0,0,0,0.1)] p-6 md:p-8"
                >
                  <div
                    class="flex justify-between items-center mb-4 pb-3 border-b border-gray-200"
                  >
                    <h2 class="text-lg md:text-xl font-semibold text-gray-900">
                      Konfirmasi {{ approvalActionLabel }}
                    </h2>
                    <button
                      @click="closeApprovalModals"
                      class="shrink-0 p-1 hover:bg-gray-100 rounded-md transition"
                    >
                      <XMarkIcon
                        class="w-6 h-6 text-gray-600 hover:text-gray-900"
                      />
                    </button>
                  </div>

                  <div class="mb-6">
                    <p class="text-gray-700 text-base">
                      Apakah sebagai
                      <span class="font-semibold text-blue-600">{{
                        approvalTypeLabel
                      }}</span>
                      anda yakin untuk melakukan
                      <span class="font-semibold text-red-600">{{
                        approvalActionLabel
                      }}</span>
                      ?
                    </p>
                  </div>

                  <div class="flex justify-end gap-3">
                    <button
                      @click="closeApprovalModals"
                      class="px-6 py-2 text-sm md:text-base border border-gray-300 text-gray-700 rounded-xl hover:bg-gray-50 transition font-regular"
                    >
                      Batal
                    </button>
                    <button
                      @click="confirmApproval"
                      class="px-6 py-2 text-sm md:text-base bg-linear-to-r from-red-500 to-red-600 text-white rounded-xl hover:opacity-90 transition font-regular"
                    >
                      Ya, Tolak
                    </button>
                  </div>
                </div>
              </div>

              <!-- Detail -->
              <div
                v-if="showDetail && currentDetailRow"
                class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 p-4"
              >
                <div
                  class="bg-white rounded-lg w-full max-w-md md:max-w-lg max-h-[90vh] overflow-y-auto shadow-[0_4px_6px_rgba(0,0,0,0.1)] p-6 md:p-8"
                >
                  <div
                    class="flex justify-between items-center mb-4 pb-3 border-b border-gray-200"
                  >
                    <h2 class="text-lg md:text-xl font-semibold text-gray-900">
                      Detail Persetujuan
                    </h2>
                    <button
                      @click="closeDetail"
                      class="shrink-0 p-1 hover:bg-gray-100 rounded-md transition"
                    >
                      <XMarkIcon
                        class="w-6 h-6 text-gray-600 hover:text-gray-900"
                      />
                    </button>
                  </div>

                  <div class="space-y-4">
                    <!-- Head Departemen -->
                    <div class="border border-gray-200 rounded-lg p-4">
                      <div class="flex justify-between items-start mb-2">
                        <label class="text-sm font-semibold text-gray-700"
                          >Head Dept</label
                        >
                        <span
                          :class="[
                            'text-sm font-semibold',
                            currentDetailRow.headApproval === null
                              ? 'text-gray-400'
                              : currentDetailRow.headApproval === true
                                ? 'text-green-600'
                                : 'text-red-600',
                          ]"
                        >
                          {{
                            currentDetailRow.headApproval === null
                              ? "Belum Dipilih"
                              : currentDetailRow.headApproval === true
                                ? "Disetujui"
                                : "Ditolak"
                          }}
                        </span>
                      </div>
                      <input
                        v-model="currentDetailRow.headApprovalTime"
                        type="datetime-local"
                        :disabled="currentDetailRow.headApproval === null"
                        class="w-full p-2 text-xs border border-gray-300 bg-white text-gray-700 rounded-md focus:outline-none disabled:bg-gray-100"
                      />
                    </div>

                    <!-- Transportasi -->
                    <div class="border border-gray-200 rounded-lg p-4">
                      <div class="flex justify-between items-start mb-2">
                        <label class="text-sm font-semibold text-gray-700"
                          >Transportasi</label
                        >
                        <span
                          :class="[
                            'text-sm font-semibold',
                            currentDetailRow.transApproval === null
                              ? 'text-gray-400'
                              : currentDetailRow.transApproval === true
                                ? 'text-green-600'
                                : 'text-red-600',
                          ]"
                        >
                          {{
                            currentDetailRow.transApproval === null
                              ? "Belum Dipilih"
                              : currentDetailRow.transApproval === true
                                ? "Disetujui"
                                : "Ditolak"
                          }}
                        </span>
                      </div>
                      <input
                        v-model="currentDetailRow.transApprovalTime"
                        type="datetime-local"
                        :disabled="currentDetailRow.transApproval === null"
                        class="w-full p-2 text-xs border border-gray-300 bg-white text-gray-700 rounded-md focus:outline-none disabled:bg-gray-100"
                      />
                    </div>

                    <!-- GA -->
                    <div class="border border-gray-200 rounded-lg p-4">
                      <div class="flex justify-between items-start mb-2">
                        <label class="text-sm font-semibold text-gray-700"
                          >GA</label
                        >
                        <span
                          :class="[
                            'text-sm font-semibold',
                            currentDetailRow.gaApproval === null
                              ? 'text-gray-400'
                              : currentDetailRow.gaApproval === true
                                ? 'text-green-600'
                                : 'text-red-600',
                          ]"
                        >
                          {{
                            currentDetailRow.gaApproval === null
                              ? "Belum Dipilih"
                              : currentDetailRow.gaApproval === true
                                ? "Disetujui"
                                : "Ditolak"
                          }}
                        </span>
                      </div>
                      <input
                        v-model="currentDetailRow.gaApprovalTime"
                        type="datetime-local"
                        :disabled="currentDetailRow.gaApproval === null"
                        class="w-full p-2 text-xs border border-gray-300 bg-white text-gray-700 rounded-md focus:outline-none disabled:bg-gray-100"
                      />
                    </div>

                    <!-- GS -->
                    <div class="border border-gray-200 rounded-lg p-4">
                      <div class="flex justify-between items-start mb-2">
                        <label class="text-sm font-semibold text-gray-700"
                          >GS</label
                        >
                        <span
                          :class="[
                            'text-sm font-semibold',
                            currentDetailRow.gsApproval === null
                              ? 'text-gray-400'
                              : currentDetailRow.gsApproval === true
                                ? 'text-green-600'
                                : 'text-red-600',
                          ]"
                        >
                          {{
                            currentDetailRow.gsApproval === null
                              ? "Belum Dipilih"
                              : currentDetailRow.gsApproval === true
                                ? "Disetujui"
                                : "Ditolak"
                          }}
                        </span>
                      </div>
                      <input
                        v-model="currentDetailRow.gsApprovalTime"
                        type="datetime-local"
                        :disabled="currentDetailRow.gsApproval === null"
                        class="w-full p-2 text-xs border border-gray-300 bg-white text-gray-700 rounded-md focus:outline-none disabled:bg-gray-100"
                      />
                    </div>
                  </div>

                  <div class="flex justify-end gap-3 mt-6">
                    <button
                      @click="closeDetail"
                      class="px-6 py-2 text-sm md:text-base border border-gray-300 text-gray-700 rounded-xl bg-red-100 hover:bg-red-200 transition font-regular"
                    >
                      Tutup
                    </button>
                  </div>
                </div>
              </div>

              <!-- Pagination -->
              <div
                class="flex flex-col md:flex-row justify-between md:justify-end items-center gap-3 md:gap-4 pt-3 md:pt-4 border-t border-gray-200"
              >
                <span
                  class="text-xs md:text-sm text-gray-700 font-medium order-2 md:order-1"
                >
                  {{ startIndex }} - {{ endIndex }} of
                  {{ filteredTableData.length }}
                </span>
                <div class="flex gap-2 order-1 md:order-2">
                  <button
                    @click="previousPage"
                    :disabled="currentPage === 1"
                    class="px-2 md:px-3 py-1 md:py-2 border border-gray-300 rounded-md text-gray-700 font-medium text-sm disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-100 transition"
                  >
                    &lt;
                  </button>
                  <button
                    @click="nextPage"
                    :disabled="currentPage === totalPages"
                    class="px-2 md:px-3 py-1 md:py-2 border border-gray-300 rounded-md text-gray-700 font-medium text-sm disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-100 transition"
                  >
                    &gt;
                  </button>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  </div>
</template>
