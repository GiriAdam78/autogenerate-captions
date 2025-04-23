<template>
  <div class="min-h-screen p-6 flex flex-col lg:flex-row gap-6">
    <!-- Sidebar: Waktu Input -->
    <div class="lg:w-1/2 w-full rounded-xl shadow-sm  dark:border-white p-6 ">
      <h2 class="text-lg font-bold mb-4 dark:text-white mx-2 my-2">
        Pengaturan Waktu
      </h2>

      <!-- Input Manual -->
      <div class="space-y-5 mb-6 py-2">
        <p class="text-sm font-medium mx-2 my-2">Masukkan Waktu Anomaly:</p>

        <input
          type="text"
          v-model="manualFirstTime"
          placeholder="First Time (contoh: 08:00)"
          class="rounded-lg block py-2 px-3 lg:w-[99%] w-[95%] border border-gray-200 outline-none focus:border-green-600"
        />

        <input
          type="text"
          v-model="manualLastTime"
          placeholder="Last Time (contoh: 10:00)"
          class="rounded-lg block py-2 px-3 lg:w-[99%] w-[95%] border border-gray-200 outline-none focus:border-green-600"
        />
      </div>

      <!-- Select Time Range -->
      <div class="space-y-2 mx-2 my-2">
        <p class="text-sm font-medium">Pilih Rentang Waktu:</p>
        <select v-model="selectedTimeRange" class="rounded-lg block py-2 px-3 lg:w-full w-[96%] border border-gray-200 outline-none focus:border-green-600 cursor-text">
          <option disabled value="">--- Pilih Jam Anomaly ---</option>
          <option value="08:00 - 10:00">08:00 - 10:00</option>
          <option value="10:00 - 12:00">10:00 - 12:00</option>
          <option value="13:00 - 15:00">13:00 - 15:00</option>
          <option value="16:00 - 18:00">16:00 - 18:00</option>
        </select>
      </div>
    </div>

    <!-- Main Form -->
    <div
      class="lg:w-2/3 w-full rounded-xl shadow-md p-6 dark:border dark:border-white"
    >
      <h1
        class="text-2xl font-bold mb-4 dark:text-white text-slate-900 font-sans"
      >
        Auto Generate Caption
      </h1>

      <UTextarea
        v-model="inputText"
        class="w-full mb-4"
        :rows="8"
        placeholder="Paste data event atau alert dari Excel di sini"
      />

      <div class="flex flex-wrap gap-3 mb-4">
        <UButton @click="generateAll" class="dark:text-white text-white">
          <UIcon name="i-lucide-file" class="size-3"></UIcon> Generate
        </UButton>

        <UButton
          @click="resetAll"
          class="dark:text-white text-white"
          color="error"
        >
          <UIcon name="i-lucide-trash-2" class="size-3"></UIcon> Reset Form
        </UButton>

        <UButton
          color="secondary"
          class="dark:text-white"
          v-if="caption || alertResult"
          @click="handleCopy(caption || alertResult)"
        >
          <UIcon name="i-lucide-files" class="size-3"></UIcon> Copy Text
        </UButton>
      </div>

      <!-- Alert Anomaly Section -->
      <div
        v-if="alertResult"
        class="dark:bg-transparent dark:text-white bg-gray-50 border p-4 rounded whitespace-pre-line mt-6"
      >
        <div class="flex justify-between items-start mb-2">
          <p class="font-semibold text-lg">Notifikasi Alert Anomali:</p>
          <button
            @click="handleCopy(alertResult)"
            class="bg-yellow-500 hover:bg-yellow-600 text-white px-3 py-1 rounded text-sm"
          >
            Copy Alert
          </button>
        </div>
        <p>{{ alertResult }}</p>
      </div>

      <!-- Generated Caption Section -->
      <div v-if="caption" class="dark:bg-transparent border p-4 rounded mt-3">
        <p class="font-semibold mb-2 font-sans">Hasil Generate Caption:</p>
        <p>{{ caption }}</p>
      </div>

      <!-- WAF Policies Information -->
      <div
        v-if="sessionParagraph"
        class="dark:bg-transparent dark:text-white bg-gray-50 border p-4 rounded whitespace-pre-line mt-3"
      >
        <div class="flex justify-between items-start mb-2">
          <p class="font-semibold text-lg">Informasi WAF Policies:</p>
          <button
            @click="handleCopy(sessionParagraph)"
            class="bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded text-sm"
          >
            Copy Paragraf WAF
          </button>
        </div>
        <p>{{ sessionParagraph }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import 'zone.js';
import { ref, watch } from "vue";

const inputText = ref("");
const caption = ref("");
const sessionParagraph = ref("");
const alertResult = ref("");

const selectedTimeRange = ref("");
const manualFirstTime = ref("");
const manualLastTime = ref("");
const selectedTimeForDisplay = ref(""); // Variabel baru untuk tampilan jam

// Regex untuk mendeteksi domain (termasuk www, subdomain, dll.)
const domainRegex =
  /(?:https?:\/\/)?(?:www\.co.id)?([a-zA-Z0-9-]+\.[a-zA-Z]{2,6})(?:\/[^\s]*)?/;

watch(selectedTimeRange, (newValue) => {
  selectedTimeForDisplay.value = newValue;
});

function generateAll() {
  const lines = inputText.value
    .split("\n")
    .map((line) => line.trim())
    .filter(Boolean);

  const alertData = {};
  const websiteData = lines.find((line) => domainRegex.test(line));

  let timeUsed = "";
  if (manualFirstTime.value && manualLastTime.value) {
    timeUsed = `${manualFirstTime.value} - ${manualLastTime.value}`;
  }

  // Generate Alert Anomaly
  if (websiteData && (timeUsed || selectedTimeForDisplay.value)) {
    const match = websiteData.match(domainRegex);
    let domain = match && match[1] ? match[1] : "domain-tidak-terdeteksi.com";

    const displayTime = selectedTimeForDisplay.value
      ? selectedTimeForDisplay.value + " WIB"
      : timeUsed
      ? timeUsed + " WIB"
      : "";

    let alertText = `⚠️ANOMALY TRAFFIC ALERT NOTIFICATION⚠️\n\nDomain/Website : *${domain}*\n\nJam : *${displayTime}*`;
    alertText += `\n\nUpdate alert notification anomaly traffic, pada jam *${timeUsed} WIB* telah terjadi peningkatan blocking traffic requests overtime yang terjadi di domain *${domain}*.`;
    alertResult.value = alertText;
  } else {
    alertResult.value = "";
  }

  const attackEvents = [];
  const extraInfo = {};
  const detailKeys = [
    "Client Type",
    "Client App",
    "Method",
    "Time",
    "Country",
    "Source IP",
  ];

  for (let i = 0; i < lines.length; i++) {
    const line = lines[i];
    const attackMatch = line.match(/^(.*?)(?:\s+(\d+))$/);
    if (attackMatch) {
      const name = attackMatch[1].trim();
      const count = parseInt(attackMatch[2]);
      if (count > 0) attackEvents.push({ name, count });
    }

    detailKeys.forEach((key) => {
      if (line.toLowerCase().startsWith(key.toLowerCase())) {
        const value = lines[i + 1]?.trim();
        if (value) extraInfo[key] = value;
      }
    });
  }

  // Generate Caption
  if (attackEvents.length > 0) {
    const eventNames = attackEvents.map((e) => `*${e.name}*`).join(", ");
    const extraList = Object.entries(extraInfo)
      .map(([k, v]) => `${k}: ${v}`)
      .join("\n");
    caption.value = `Berikut adalah Detail Event ${eventNames} tersebut sebagai berikut :\n\n${extraList}`;
  } else {
    caption.value = "Tidak ada event yang terdeteksi.";
  }

  // Generate WAF Policies Information
  if (attackEvents.length > 0) {
    sessionParagraph.value = `Setelah dilakukan pengecekan pada dashboard WAF Policies dan Security Rules, insiden anomali tersebut memicu WAF dengan jenis Incident ${attackEvents
      .map((e) => `*${e.name}* Sebanyak ${e.count} Sessions`)
      .join(", ")}.`;
  } else {
    sessionParagraph.value = "Tidak ada event yang memicu WAF Policies.";
  }
}

function handleCopy(text) {
  if (!text) return;
  navigator.clipboard
    .writeText(text)
    .then(() => alert("Teks berhasil disalin!"))
    .catch(() => alert("Gagal menyalin teks."));
}

function resetAll() {
  inputText.value = "";
  caption.value = "";
  sessionParagraph.value = "";
  alertResult.value = "";
  selectedTimeRange.value = "";
  manualFirstTime.value = "";
  manualLastTime.value = "";
}
</script>
