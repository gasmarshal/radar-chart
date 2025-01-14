<script setup>
import { onMounted, ref } from "vue";

// Membuat referensi untuk elemen canvas
const radarChart = ref(null);

const drawRadarChart = () => {
  const canvas = radarChart.value;
  const ctx = canvas.getContext("2d");

  // Data chart
  const labels = [
    "Energy Support",
    "Iron Supplement",
    "Energy Boost",
    "Recovery Support",
  ];
  const data1 = [8, 8, 8, 8]; // Data set pertama
  const data2 = [8, 0, 0, 7]; // Data set kedua
  const maxValue = 10;
  const numCircles = 10; // Jumlah lapisan grid lingkaran

  // Ukuran canvas dan pusat chart
  const width = canvas.width;
  const height = canvas.height;
  const centerX = width / 2;
  const centerY = height / 2;
  const radius = Math.min(width, height) / 2 - 40;

  // Hitung sudut antar sumbu
  const angleStep = (2 * Math.PI) / labels.length;

  // Gambar grid lingkaran konsentris
  ctx.strokeStyle = "#ccc";
  ctx.lineWidth = 1;
  for (let i = 1; i <= numCircles; i++) {
    const currentRadius = (i / numCircles) * radius;
    ctx.beginPath();
    ctx.arc(centerX, centerY, currentRadius, 0, 2 * Math.PI);
    ctx.closePath();
    ctx.stroke();
  }

  // Gambar garis radial
  ctx.beginPath();
  for (let i = 0; i < labels.length; i++) {
    const x = centerX + radius * Math.cos(i * angleStep);
    const y = centerY + radius * Math.sin(i * angleStep);
    ctx.moveTo(centerX, centerY);
    ctx.lineTo(x, y);
  }
  ctx.stroke();

  // Fungsi untuk menggambar polygon data
  const drawDataPolygon = (data, color) => {
    ctx.beginPath();
    for (let i = 0; i < data.length; i++) {
      const valueRadius = (data[i] / maxValue) * radius;
      const x = centerX + valueRadius * Math.cos(i * angleStep);
      const y = centerY + valueRadius * Math.sin(i * angleStep);
      ctx.lineTo(x, y);
    }
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.strokeStyle = color;
    ctx.globalAlpha = 0.5;
    ctx.fill();
    ctx.globalAlpha = 1;
    ctx.stroke();
  };

  // Gambar polygon untuk data pertama
  drawDataPolygon(data1, "rgba(255, 0, 0, 0.5)");

  // Gambar polygon untuk data kedua
  drawDataPolygon(data2, "rgba(0, 128, 255, 0.5)");

  // Gambar label
  ctx.fillStyle = "white";
  ctx.font = "14px Arial";
  for (let i = 0; i < labels.length; i++) {
    const x = centerX + (radius + 20) * Math.cos(i * angleStep);
    const y = centerY + (radius + 20) * Math.sin(i * angleStep);

    // Menyesuaikan posisi teks agar tidak terpotong
    ctx.textAlign = Math.cos(i * angleStep) > 0 ? "left" : "right";
    ctx.textBaseline = Math.sin(i * angleStep) > 0 ? "top" : "bottom";

    ctx.fillText(labels[i], x, y);
  }
};

// Menggambar radar chart saat komponen dimuat
onMounted(() => {
  drawRadarChart();
});
</script>

<template>
  <canvas ref="radarChart" width="800" height="500"></canvas>
</template>

<style scoped>
canvas {
  display: block;
  margin: auto;
  border: 1px solid #b4b4b4;
}
</style>
