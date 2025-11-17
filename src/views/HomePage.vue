<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Proyek Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <ion-refresher slot="fixed" @ionRefresh="handleRefresh">
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>

      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Proyek Cuaca Jakarta</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-card v-if="weatherData.length > 0 && !loading">
        <ion-card-header>
          <ion-card-title>🌡️ Suhu Saat Ini</ion-card-title>
          <ion-card-subtitle>Jakarta (Lat: -6.2, Lon: 106.8)</ion-card-subtitle>
        </ion-card-header>
        <ion-card-content>
          <h1 class="current-temp">
            {{ weatherData[0].temperature_2m.toFixed(1) }}°C
          </h1>
          <p class="current-time">
            Diperbarui pada: {{ formatTime(weatherData[0].time) }}
          </p>
        </ion-card-content>
      </ion-card>

      <h2>Prakiraan Suhu Per Jam</h2>

      <div v-if="loading">
        <div class="skeleton-container">
          <ion-skeleton-text v-if="weatherData.length === 0" animated style="height: 150px; border-radius: 12px; margin-bottom: 20px;"></ion-skeleton-text>
          
          <div class="hourly-forecast">
            <ion-skeleton-text v-for="n in 7" :key="n" animated class="temp-chip-skeleton"></ion-skeleton-text>
          </div>
        </div>
      </div>

      <div v-else-if="error" class="ion-text-center">
        <p class="error-message">
          Gagal mengambil data cuaca: {{ error.message }}
        </p>
        <ion-button expand="block" @click="fetchWeatherData">Coba Lagi</ion-button>
      </div>

      <ion-grid v-else>
        <ion-row class="ion-justify-content-start hourly-forecast">
          <ion-col
            size="auto"
            v-for="(data, index) in weatherData"
            :key="index"
          >
            <div class="temp-chip" :class="getTempClass(data.temperature_2m)">
              <div class="time-label">{{ formatTimeDetailed(data.time) }}</div>
              <div class="temp-value">
                {{ data.temperature_2m.toFixed(0) }}°
              </div>
            </div>
          </ion-col>
        </ion-row>
      </ion-grid>
      
      <p v-if="weatherData.length > 0 && !loading" class="scroll-hint">
        &larr; Geser ke kiri untuk melihat prakiraan jam selanjutnya &rarr;
      </p>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonGrid,
  IonRow,
  IonCol,
  IonRefresher, 
  IonRefresherContent, 
  IonSkeletonText, 
  IonButton, 
} from "@ionic/vue";
import { ref, onMounted } from "vue";

// Mendefinisikan tipe data untuk hasil akhir yang akan ditampilkan
interface HourlyWeather {
  time: string;
  temperature_2m: number;
}

// Mendefinisikan state reaktif
const weatherData = ref<HourlyWeather[]>([]);
const loading = ref(true);
const error = ref<Error | null>(null);

// URL API Cuaca
const API_URL =
  "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m&timezone=Asia%2FJakarta";

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

    if (data.hourly && data.hourly.time && data.hourly.temperature_2m) {
      const times: string[] = data.hourly.time;
      const temps: number[] = data.hourly.temperature_2m;

      const combinedData: HourlyWeather[] = times.map((t, index) => ({
        time: t,
        temperature_2m: temps[index],
      }));

      // Ambil hanya 24 jam ke depan
      weatherData.value = combinedData.slice(0, 24); 
    } else {
      throw new Error("Struktur data API tidak sesuai yang diharapkan.");
    }
  } catch (err) {
    if (err instanceof Error) {
      error.value = err;
    } else {
      error.value = new Error("Terjadi kesalahan yang tidak diketahui.");
    }
  } finally {
    loading.value = false;
  }
};


const handleRefresh = async (event: CustomEvent) => {
  // Hanya atur loading ke false setelah data berhasil dimuat
  await fetchWeatherData();
  event.detail.complete(); 
};


const formatTime = (isoString: string): string => {
  const date = new Date(isoString);
  return date.toLocaleString("id-ID", {
    day: "2-digit",
    month: "short",
    hour: "2-digit",
    minute: "2-digit",
  });
};


const formatTimeDetailed = (isoString: string): string => {
  const date = new Date(isoString);
 
  return date.toLocaleString("id-ID", { hour: "2-digit", minute: "2-digit" });
};


const getTempClass = (temp: number): string => {
  if (temp >= 32) {
    return 'temp-hot'; // Merah (Panas)
  } else if (temp >= 28) {
    return 'temp-warm'; // Kuning/Oranye (Hangat)
  } else if (temp >= 25) {
    return 'temp-mild'; // Hijau (Sedang)
  } else {
    return 'temp-cool'; // Biru (Sejuk)
  }
};

// Panggil fungsi fetch data saat komponen dimuat (onMounted)
onMounted(() => {
  fetchWeatherData();
});
</script>

<style scoped>

ion-content {
  --background: #f4f5f8; 
}

h2 {
    margin-top: 30px;
    margin-bottom: 15px;
    font-weight: 600;
}


ion-card {
  --background: #ffffff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border-radius: 12px;
  margin-bottom: 20px;
}

.current-temp {
  font-size: 4em; 
  font-weight: 700;
  margin: 0;
  color: var(--ion-color-primary, #3880ff); 
}

.current-time {
  font-size: 0.8em;
  color: #666;
  margin-top: 5px;
}


.hourly-forecast {
  flex-wrap: nowrap; 
  overflow-x: scroll; 
  padding-bottom: 15px; 
  display: flex;
}


.temp-chip {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 70px; 
  height: 90px;
  border-radius: 10px;
  margin-right: 10px;
  padding: 8px 0;
  flex-shrink: 0; 
  color: white; 
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15);
}

.time-label {
  font-size: 0.75em;
  opacity: 0.8;
  margin-bottom: 4px;
}

.temp-value {
  font-size: 1.6em;
  font-weight: bold;
}

.temp-hot {
  background: var(--ion-color-danger, #eb445a); 
}
.temp-warm {
  background: var(--ion-color-warning, #ffc409); 
  color: #333; 
}
.temp-mild {
  background: var(--ion-color-success, #2dd36f); 
}
.temp-cool {
  background: var(--ion-color-tertiary, #5260ff); 
}

.scroll-hint {
    text-align: center;
    font-size: 0.8em;
    color: #999;
    margin-top: 10px;
}

.skeleton-container {
    padding-top: 10px;
}

.temp-chip-skeleton {
    width: 70px;
    height: 90px;
    border-radius: 10px;
    margin-right: 10px;
    flex-shrink: 0; 
    display: inline-block;
}

.error-message {
    color: var(--ion-color-danger, red);
    margin-bottom: 15px;
}
</style>