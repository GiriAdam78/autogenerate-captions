<script setup>
import "zone.js";
import { ref, watch } from "vue";

const inputText = ref("");
const ipReputationText = ref("");
const caption = ref("");
const sessionParagraph = ref("");
const alertResult = ref("");
const ipReputationResult = ref("");

// Pilihan waktu anomaly
const anomaly = ref([
  "08:00 -10:00",
  "10:00 - 12:00",
  "13:00 - 15:00",
  "16:00 - 18:00",
  "18:00 - 20:00",
  "20:00 - 22:00",
  "22:00 - 24:00",
  "00:00 - 02:00",
]);

const selectedTimeRange = ref("");
const manualFirstTime = ref("");
const manualLastTime = ref("");
const selectedTimeForDisplay = ref("");

// Regex baru untuk mendeteksi domain
const domainRegex =
  /(?:https?:\/\/)?(?:www\.)?([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})(?:\/[^\s]*)?/;

// Update tampilan jam ketika memilih waktu
watch(selectedTimeRange, (newValue) => {
  selectedTimeForDisplay.value = newValue;
});

function generateAll() {
  const lines = inputText.value
    .split("\n")
    .map((line) => line.trim())
    .filter(Boolean);

  const websiteData = lines.find((line) => domainRegex.test(line));

  let timeUsed = "";
  if (manualFirstTime.value && manualLastTime.value) {
    timeUsed = `${manualFirstTime.value} - ${manualLastTime.value}`;
  }

  // Generate Alert Anomaly
  if (websiteData && (timeUsed || selectedTimeForDisplay.value)) {
    const match = websiteData.match(domainRegex);
    let domain = match && match[1] ? match[1] : "domain-tidak-terdeteksi.com";

    // Tambahkan .co.id jika belum ada
    if (!domain.endsWith(".co.id")) {
      domain += ".co.id";
    }

    const displayTime = selectedTimeForDisplay.value
      ? selectedTimeForDisplay.value + " WIB"
      : timeUsed
      ? timeUsed + " WIB"
      : "";

    let alertText = `⚠️ANOMALY TRAFFIC ALERT NOTIFICATION⚠️\n\nDomain/Website : *${domain}*\n\nJam : *${displayTime}*`;
    alertText += `\n\nUpdate alert notification anomaly traffic, pada jam *${
      timeUsed || selectedTimeForDisplay.value
    } WIB* telah terjadi peningkatan blocking traffic requests overtime yang terjadi di domain *${domain}*.`;
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

  // Generate IP Reputation
  if (Object.keys(extraInfo).length > 0) {
    const sourceIP = extraInfo["Source IP"] || "-";
    const country = extraInfo["Country"] || "-";
    const city = extraInfo["City"] || "-";

    const eventNames = attackEvents.map((e) => `*${e.name}*`).join(", ");

    const linesReputation = ipReputationText.value
      .split("\n")
      .map((line) => line.trim())
      .filter(Boolean);

    let asn = "-";
    let riskScore = "-";

    linesReputation.forEach((line) => {
      if (line.toLowerCase().startsWith("asn/isp")) {
        const value = line.split(":")[1]?.trim();
        if (value) asn = value;
      }
      if (line.toLowerCase().startsWith("risk score")) {
        const value = line.split(":")[1]?.trim();
        if (value) riskScore = value;
      }
    });

    ipReputationResult.value = `\n\nDan berikut adalah Informasi dari Source IP *${sourceIP}* yang telah melakukan serangan ${eventNames} yang dilihat dari IP Reputation Intelligence:\n\nCountry: ${country}\nCity: ${city}\nASN/ISP: ${asn}\nRisk Score: ${riskScore}`;
  } else {
    ipReputationResult.value = "";
  }
}

// Fungsi untuk Copy Text
function handleCopy(text) {
  if (!text) return;
  navigator.clipboard
    .writeText(text)
    .then(() => alert("Teks berhasil disalin!"))
    .catch(() => alert("Gagal menyalin teks."));
}

// Fungsi untuk Reset semua input dan output
function resetAll() {
  inputText.value = "";
  ipReputationText.value = "";
  caption.value = "";
  sessionParagraph.value = "";
  alertResult.value = "";
  ipReputationResult.value = "";
  selectedTimeRange.value = "";
  manualFirstTime.value = "";
  manualLastTime.value = "";
}
</script>

<template>
  <div class="min-h-screen p-6 flex flex-col lg:flex-row gap-6">
    <!-- Sidebar: Waktu Input -->
    <div class="lg:w-1/2 w-full rounded-xl shadow-sm dark:border-white p-6">
      <h2
        class="text-lg font-bold mb-4 dark:text-white mx-2 my-2 flex items-center"
      >
        <UIcon name="i-lucide-clock" class="size-5 mx-1" />Pengaturan Waktu
      </h2>

      <!-- Input Manual -->
      <div class="-z-10 space-y-5 mb-6 py-2">
        <p class="text-sm font-medium mx-2 my-2">Masukkan Waktu Anomaly:</p>

        <input
          type="text"
          v-model="manualFirstTime"
          placeholder="Frist Time (ex: 08:00)"
          class="border border-gray-300 py-1.5 p-2 mx-2 rounded outline-none md:w-[47%] w-full focus:ring-2 focus:ring-offset-1 focus:ring-blue-400 focus:border-blue-500/10 shadow-sm caret-blue-500"
        />
        <input
          type="text"
          v-model="manualLastTime"
          placeholder="Last Time (ex: 10:00)"
          class="border border-gray-300 py-1.5 p-2 mx-2 rounded outline-none md:w-[47%] w-full focus:ring-2 focus:ring-offset-1 focus:ring-blue-400 focus:border-blue-500/10 shadow-sm caret-blue-500"
        />
      </div>

      <!-- Select Time Range -->
      <div class="space-y-2 mx-2 my-2">
        <p class="text-sm font-medium">Pilih Rentang Waktu:</p>
        <select
          v-model="selectedTimeRange"
          class="border border-gray-300 py-1.5 p-2 rounded appearance-none outline-none w-full focus:ring-2 focus:ring-offset-1 focus:ring-blue-400 focus:border-blue-500/10 cursor-pointer shadow-sm caret-blue-500"
        >
          <option disabled value="" class="mt-3">
            --- Pilih Jam Anomaly ---
          </option>
          <option value="08:00 - 10:00">08:00 - 10:00</option>
          <option value="10:00 - 12:00">10:00 - 12:00</option>
          <option value="13:00 - 15:00">13:00 - 15:00</option>
          <option value="16:00 - 18:00">16:00 - 18:00</option>
          <option value="18:00 - 20:00">18:00 - 20:00</option>
          <option value="20:00 - 22:00">20:00 - 22:00</option>
          <option value="22:00 - 24:00">22:00 - 24:00</option>
          <option value="24:00 - 00:00">24:00 - 00:00</option>
          <option value="00:00 - 02:00">00:00 - 02:00</option>
        </select>
        <!-- <USelect
          placeholder="Pilih Waktu Kejadian Anomaly "
          v-model="selectedTimeRange"
          :items="anomaly"
          class="-z-10 md:z-20 w-full cursor-pointer"
        /> -->
      </div>
    </div>

    <!-- Main Form -->
    <div
      class="lg:w-2/3 w-full rounded-xl shadow-md p-6 dark:border dark:border-white"
    >
      <h1
        class="text-2xl font-bold mb-4 dark:text-white text-slate-900 font-sans"
      >
        Paste Caption :
      </h1>

      <!-- <UTextarea
        v-model="inputText"
        class="w-full mb-4 -z-10 md:z-20"
        :rows="8"
        placeholder="Paste data event atau alert dari Excel di sini"
      /> -->

      <textarea
        v-model="inputText"
        rows="8"
        placeholder="Paste Data Event Atau Alert Dari File Excel Ke Sini"
        class="p-1 mb-2 border resize border-gray-300 rounded-md field-sizing-fixed w-full outline-none focus:ring-2 focus:ring-offset-1 focus:ring-blue-400 focus:border-blue-500/10 cursor-text caret-blue-500"
      ></textarea>
      <div>
        <h2 class="text-lg font-semibold mb-2">Input IP Reputation Data</h2>
        <textarea
          v-model="ipReputationText"
          rows="8"
          placeholder="Masukkan Data IP Reputation Di Sini ...."
          class="p-1 mb-2 border resize border-gray-300 rounded-md field-sizing-fixed w-full outline-none focus:ring-2 focus:ring-offset-1 focus:ring-blue-400 focus:border-blue-500/10 cursor-text caret-blue-500"
        ></textarea>
      </div>

      <div class="flex flex-wrap gap-3 mb-4">
        <UButton
          @click="generateAll"
          class="dark:text-white text-white cursor-pointer"
          color="secondary"
        >
          <UIcon name="i-lucide-file" class="size-3"></UIcon> Generate
        </UButton>

        <UButton
          @click="resetAll"
          class="dark:text-white text-white cursor-pointer"
          color="error"
        >
          <UIcon name="i-lucide-trash-2" class="size-3"></UIcon> Reset Form
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
      <!-- WAF Policies Information -->
      <div
        v-if="sessionParagraph"
        class="dark:bg-transparent dark:text-white bg-gray-50 border p-4 rounded whitespace-pre-line mt-3"
      >
        <div class="flex justify-between items-start mb-2">
          <p class="font-semibold text-lg">Informasi WAF Policies:</p>
          <UButton
            color="secondary"
            @click="handleCopy(sessionParagraph)"
            class="dark:text-white text-white cursor-pointer"
          >
            <UIcon name="i-lucide-files" class="size-3"></UIcon> Copy Text
          </UButton>
        </div>
        <p>{{ sessionParagraph }}</p>
      </div>

      <!-- Generated Caption Section -->
      <div v-if="caption" class="dark:bg-transparent border p-4 rounded mt-3">
        <div class="flex justify-between items-start mb-2">
          <p class="font-semibold mb-2 font-sans">Hasil Generate Caption:</p>

          <UButton
            color="secondary"
            class="dark:text-white cursor-pointer"
            v-if="caption || alertResult"
            @click="handleCopy(caption || alertResult)"
          >
            <UIcon name="i-lucide-files" class="size-3"></UIcon> Copy Text
          </UButton>
        </div>
        <p>{{ caption }}</p>
      </div>

      <!-- Ip Reputation Results -->
      <div
        v-if="ipReputationResult"
        class="dark:bg-transparent border p-4 rounded mt-3"
      >
        <div class="flex justify-between items-start mb-2">
          <p class="font-semibold mb-2 font-sans">Hasil IP Reputation:</p>

          <UButton
            color="secondary"
            class="dark:text-white cursor-pointer"
            v-if="ipReputationResult"
            @click="handleCopy(ipReputationResult)"
          >
            <UIcon name="i-lucide-files" class="size-3"></UIcon> Copy Text
          </UButton>
        </div>
        <p>{{ ipReputationResult }}</p>
      </div>
    </div>
  </div>
</template>
