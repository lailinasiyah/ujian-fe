<template>
  <div class="container my-4">
    <h3 class="mb-3 text-center">Form Input Nilai Ujian</h3>
    <form @submit.prevent="submitForm">
      <!-- Nama -->
      <div class="mb-3">
        <label class="form-label">Nama Peserta</label>
        <input type="text" class="form-control" v-model="form.nama" />
      </div>

      <!-- Jenis Ujian -->
      <div class="mb-3">
        <label class="form-label">Jenis Ujian</label>
        <select class="form-select" v-model="form.jenis" @change="handleChange">
          <option disabled value="">Pilih Jenis Ujian</option>
          <option>JLPT</option>
          <option>JTEST</option>
          <option>NAT</option>
        </select>
      </div>

      <!-- Nilai -->
      <div class="row">
        <div class="col-md-4 mb-3">
          <label class="form-label">Mojigoi</label>
          <input
            type="number"
            class="form-control"
            v-model.number="form.mojigoi"
            :min="range.mojigoi.min"
            :max="range.mojigoi.max"
          />
          <small v-if="!isInRange('mojigoi')" class="text-danger">
            Nilai harus antara {{ range.mojigoi.min }} - {{ range.mojigoi.max }}
          </small>
        </div>
        <div class="col-md-4 mb-3">
          <label class="form-label">Dokkai</label>
          <input
            type="number"
            class="form-control"
            v-model.number="form.dokkai"
            :min="range.dokkai.min"
            :max="range.dokkai.max"
          />
          <small v-if="!isInRange('dokkai')" class="text-danger">
            Nilai harus antara {{ range.dokkai.min }} - {{ range.dokkai.max }}
          </small>
        </div>
        <div class="col-md-4 mb-3">
          <label class="form-label">Choukai</label>
          <input
            type="number"
            class="form-control"
            v-model.number="form.choukai"
            :min="range.choukai.min"
            :max="range.choukai.max"
          />
          <small v-if="!isInRange('choukai')" class="text-danger">
            Nilai harus antara {{ range.choukai.min }} - {{ range.choukai.max }}
          </small>
        </div>
      </div>

      <!-- Reactive Hasil -->
      <div class="mb-3">
        <label class="form-label">Total Nilai</label>
        <input type="number" class="form-control" :value="totalNilai" readonly />
      </div>

      <button type="submit" class="btn btn-success" :disabled="!isValid">
        Hitung & Simpan
      </button>
    </form>

    <!-- Hasil dari server -->
    <div v-if="result" class="alert alert-info mt-3">
      <p><b>Server Nama:</b> {{ result?.nama }}</p>
      <p><b>Server Jenis:</b> {{ result?.jenis }}</p>
      <p><b>Server Total Nilai:</b> {{ result?.nilai?.total }}</p>
      <p><b>Server Hasil:</b> {{ result?.nilai?.hasil }}</p>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted } from "vue";
import axios from "axios";

// reactive form
const form = reactive({
  nama: "",
  jenis: "",
  mojigoi: null,
  dokkai: null,
  choukai: null,
});

const result = ref(null);

// range nilai dinamis
const range = computed(() => {
  if (form.jenis === "JTEST") {
    return {
      mojigoi: { min: 0, max: 200 },
      dokkai: { min: 0, max: 200 },
      choukai: { min: 0, max: 200 },
    };
  }
  // default JLPT/NAT
  return {
    mojigoi: { min: 0, max: 60 },
    dokkai: { min: 0, max: 60 },
    choukai: { min: 0, max: 60 },
  };
});

// total nilai reactive
const totalNilai = computed(() => {
  if (form.mojigoi === null || form.dokkai === null || form.choukai === null) {
    return null;
  }
  return (form.mojigoi || 0) + (form.dokkai || 0) + (form.choukai || 0);
});



// cek per input sesuai range
const isInRange = (field) => {
  return (
    form[field] >= range.value[field].min &&
    form[field] <= range.value[field].max
  );
};

// validasi form keseluruhan
const isValid = computed(() => {
  return (
    form.nama.trim() !== "" &&
    form.jenis !== "" &&
    isInRange("mojigoi") &&
    isInRange("dokkai") &&
    isInRange("choukai")
  );
});

// ambil dari .env
const dev = false;
const API_URL = dev ? import.meta.env.VITE_API_URL_DEV : import.meta.env.VITE_API_URL;

onMounted(() => {
  console.log("API URL:", API_URL);
});

// handle jenis ujian
const handleChange = () => {
  form.mojigoi = null;
  form.dokkai = null;
  form.choukai = null;
  result.value = null;
};

// submit ke server
const submitForm = async () => {
  try {
    const res = await axios.post(`${API_URL}/ujian/hitung`, form);
    result.value = res.data;
  } catch (err) {
    console.error("Error:", err);
  }
};
</script>
