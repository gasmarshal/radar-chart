<script setup>
import { onMounted, ref } from "vue";

// Membuat referensi untuk elemen canvas
const radarChart = ref(null);
let isDragging = false;
let draggedIndex = null;
let data1 = [8, 8, 8, 8]; // Data set pertama
const labels = [
  "Energy Support",
  "Iron Supplement",
  "Energy Boost",
  "Recovery Support",
];
const maxValue = 10;
const numCircles = 10; // Jumlah lapisan grid lingkaran

const drawRadarChart = () => {
  const canvas = radarChart.value;
  const ctx = canvas.getContext("2d");

  // Ukuran canvas dan pusat chart
  const width = canvas.width;
  const height = canvas.height;
  const centerX = width / 2;
  const centerY = height / 2;
  const radius = Math.min(width, height) / 2 - 40;
  const angleStep = (2 * Math.PI) / labels.length;

  ctx.clearRect(0, 0, width, height); // Bersihkan canvas sebelum menggambar ulang

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

  // Gambar polygon data
  ctx.beginPath();
  for (let i = 0; i < data1.length; i++) {
    const valueRadius = (data1[i] / maxValue) * radius;
    const x = centerX + valueRadius * Math.cos(i * angleStep);
    const y = centerY + valueRadius * Math.sin(i * angleStep);
    ctx.lineTo(x, y);
  }
  ctx.closePath();

  ctx.strokeStyle = "red";
  ctx.globalAlpha = 0.5;
  ctx.globalAlpha = 1;
  ctx.stroke();

  // Gambar titik data
  ctx.fillStyle = "red";
  for (let i = 0; i < data1.length; i++) {
    const valueRadius = (data1[i] / maxValue) * radius;
    const x = centerX + valueRadius * Math.cos(i * angleStep);
    const y = centerY + valueRadius * Math.sin(i * angleStep);
    ctx.beginPath();
    ctx.arc(x, y, 6, 0, 2 * Math.PI);
    ctx.closePath();
    ctx.fill();
  }

  // Gambar label
  ctx.fillStyle = "white";
  ctx.font = "14px Arial";
  for (let i = 0; i < labels.length; i++) {
    const x = centerX + (radius + 20) * Math.cos(i * angleStep);
    const y = centerY + (radius + 20) * Math.sin(i * angleStep);

    ctx.textAlign = Math.cos(i * angleStep) > 0 ? "left" : "right";
    ctx.textBaseline = Math.sin(i * angleStep) > 0 ? "top" : "bottom";
    ctx.fillText(labels[i], x, y);
  }
};

// Fungsi untuk menangani drag
const handleMouseDown = (e) => {
  const canvas = radarChart.value;
  const rect = canvas.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  const mouseY = e.clientY - rect.top;
  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const radius = Math.min(canvas.width, canvas.height) / 2 - 40;
  const angleStep = (2 * Math.PI) / labels.length;

  // Cek apakah mouse menekan dekat titik data
  for (let i = 0; i < data1.length; i++) {
    const valueRadius = (data1[i] / maxValue) * radius;
    const x = centerX + valueRadius * Math.cos(i * angleStep);
    const y = centerY + valueRadius * Math.sin(i * angleStep);

    const distance = Math.sqrt((mouseX - x) ** 2 + (mouseY - y) ** 2);
    if (distance < 10) {
      isDragging = true;
      draggedIndex = i;
      break;
    }
  }
};

const handleMouseMove = (e) => {
  if (!isDragging) return;

  const canvas = radarChart.value;
  const rect = canvas.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  const mouseY = e.clientY - rect.top;

  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;

  const angle = Math.atan2(mouseY - centerY, mouseX - centerX);
  const distance = Math.min(
    Math.sqrt((mouseX - centerX) ** 2 + (mouseY - centerY) ** 2),
    centerX - 40
  );

  const newValue = Math.round((distance / (centerX - 40)) * maxValue);
  data1[draggedIndex] = newValue;
  drawRadarChart();
};

const handleMouseUp = () => {
  isDragging = false;
  draggedIndex = null;
};

// Menggambar radar chart saat komponen dimuat
onMounted(() => {
  drawRadarChart();
  const canvas = radarChart.value;
  canvas.addEventListener("mousedown", handleMouseDown);
  canvas.addEventListener("mousemove", handleMouseMove);
  canvas.addEventListener("mouseup", handleMouseUp);
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
  cursor: pointer;
}
</style>
