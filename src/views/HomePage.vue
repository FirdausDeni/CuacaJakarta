<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Proyek Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Proyek Cuaca Jakarta</ion-title>
        </ion-toolbar>
      </ion-header>

      <h2>Prakiraan Suhu Per Jam</h2>

      <div v-if="loading">
        <p>Memuat data cuaca...</p>
      </div>

      <div v-else-if="error">
        <p style="color: red;">Gagal mengambil data cuaca: {{ error.message }}</p>
      </div>

      <ion-list v-else>
        <ion-item>
          <ion-label>
            <strong>Waktu</strong>
          </ion-label>
          <ion-note slot="end">
            <strong>Suhu (°C)</strong>
          </ion-note>
        </ion-item>
        
        <ion-item v-for="(data, index) in weatherData" :key="index">
          <ion-label>
            {{ formatTime(data.time) }}
          </ion-label>
          <ion-note slot="end">
            {{ data.temperature_2m.toFixed(1) }} °C
          </ion-note>
        </ion-item>
      </ion-list>
      
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonList, IonItem, IonLabel, IonNote } from '@ionic/vue';
import { ref, onMounted } from 'vue';

// Definisikan tipe data untuk hasil akhir yang akan ditampilkan
interface HourlyWeather {
  time: string;
  temperature_2m: number;
}

// Definisikan state reaktif
const weatherData = ref<HourlyWeather[]>([]);
const loading = ref(true);
const error = ref<Error | null>(null);

// URL API Cuaca
const API_URL = "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m";

// Fungsi untuk mengambil data dari API
const fetchWeatherData = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await fetch(API_URL);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();

    // Pastikan data 'hourly' ada dan memiliki 'time' dan 'temperature_2m'
    if (data.hourly && data.hourly.time && data.hourly.temperature_2m) {
      const times: string[] = data.hourly.time;
      const temps: number[] = data.hourly.temperature_2m;
      
      // Menggabungkan array time dan temperature_2m
      const combinedData: HourlyWeather[] = times.map((t, index) => ({
        time: t,
        temperature_2m: temps[index],
      }));
      
      weatherData.value = combinedData;
    } else {
      throw new Error("Struktur data API tidak sesuai yang diharapkan.");
    }
  } catch (err) {
    // Penanganan error
    if (err instanceof Error) {
        error.value = err;
    } else {
        error.value = new Error("Terjadi kesalahan yang tidak diketahui.");
    }
  } finally {
    loading.value = false;
  }
};

// Fungsi untuk memformat waktu dari string ISO menjadi format yang lebih mudah dibaca
const formatTime = (isoString: string): string => {
  const date = new Date(isoString);
  // Contoh format: 03 Nov, 12:00
  return date.toLocaleString('id-ID', { day: '2-digit', month: 'short', hour: '2-digit', minute: '2-digit' });
}

// Panggil fungsi fetch data saat komponen dimuat (onMounted)
onMounted(() => {
  fetchWeatherData();
});
</script>
<style scoped>
ion-list {
  padding-top: 0;
}


ion-label {
  /* Atur ukuran font untuk kolom Waktu */
  font-size: 16px;
}
ion-note {
  /* Atur ukuran font untuk kolom Suhu */
  font-size: 16px;
  font-weight: bold;
  color: var(--ion-color-dark, #333);
}
</style>
