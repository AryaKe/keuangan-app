<template>
  <div class="form-container">
    <h2>💰 Form Keuangan</h2>

    <label>Periode:</label>
    <select v-model="periode" required>
      <option disabled value="">-- Pilih Periode --</option>
      <option value="mingguan">Mingguan</option>
      <option value="bulanan">Bulanan</option>
      <option value="tahunan">Tahunan</option>
    </select>

    <label>Uang yang kamu miliki sekarang:</label>
    <input type="number" v-model.number="uangDimiliki" placeholder="Contoh: 500000" min="0" />

    <label>Batas sisa ATM (opsional):</label>
    <input type="number" v-model.number="batasATM" placeholder="Contoh: 100000" min="0" />

    <div v-for="(item, index) in keperluan" :key="index" class="keperluan-row">
      <input type="text" v-model="item.nama" placeholder="Nama Keperluan" required />
      <input type="number" v-model.number="item.nominal" placeholder="Nominal" min="0" />
      <select v-model="item.satuan">
        <option v-for="opsi in opsiSatuan" :key="opsi" :value="opsi">
          {{ opsi }}
        </option>
      </select>
      <button @click="hapusKeperluan(index)">🗑</button>
    </div>

    <button @click="tambahKeperluan">+ Tambah Keperluan</button>

    <button class="submit-btn" @click="prosesData">Hitung & Download PDF</button>
    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import jsPDF from 'jspdf'

const periode = ref('')
const keperluan = ref([{ nama: '', nominal: null, satuan: 'total' }])
const uangDimiliki = ref(null)
const batasATM = ref(null)
const error = ref('')

const opsiSatuan = computed(() => {
  if (periode.value === 'mingguan') return ['total', 'perhari', 'perminggu']
  if (periode.value === 'bulanan') return ['total', 'perhari', 'perminggu', 'perbulan']
  if (periode.value === 'tahunan') return ['total', 'perhari', 'perminggu', 'perbulan']
  return ['total'] // default
})


// Faktor konversi satuan
const hitungNominal = (nominal, satuan) => {
  if (satuan === 'total') return nominal
  const faktor = {
    mingguan: { perhari: 7, perminggu: 1, perbulan: 1 / 4 },
    bulanan: { perhari: 30, perminggu: 4, perbulan: 1 },
    tahunan: { perhari: 365, perminggu: 52, perbulan: 12 }
  }
  return Math.round(nominal * (faktor[periode.value]?.[satuan] || 1))
}

const getLabelPerkalian = (nominal, satuan) => {
  if (satuan === 'total') return `Rp ${nominal.toLocaleString()}`
  const faktor = {
    mingguan: { perhari: 7, perminggu: 1, perbulan: '1/4' },
    bulanan: { perhari: 30, perminggu: 4, perbulan: 1 },
    tahunan: { perhari: 365, perminggu: 52, perbulan: 12 }
  }
  const f = faktor[periode.value]?.[satuan]
  const hasil = hitungNominal(nominal, satuan)
  return `Rp ${nominal.toLocaleString()} × ${f} = Rp ${hasil.toLocaleString()}`
}

const tambahKeperluan = () => {
  keperluan.value.push({ nama: '', nominal: null, satuan: 'total' })
}

const hapusKeperluan = (index) => {
  keperluan.value.splice(index, 1)
}

const prosesData = () => {
  error.value = ''
  if (!periode.value) return error.value = 'Pilih periode dahulu.'
  if (uangDimiliki.value == null || uangDimiliki.value <= 0)
    return error.value = 'Masukkan jumlah uang yang dimiliki.'
  for (let item of keperluan.value) {
    if (!item.nama) return error.value = 'Isi semua nama keperluan.'
  }

  // Hitung total keperluan terisi
  const totalTerisi = keperluan.value
    .filter(k => k.nominal != null)
    .reduce((a, b) => a + hitungNominal(b.nominal, b.satuan), 0)

  // Hitung dan bagi keperluan yang kosong
  const kosong = keperluan.value.filter(k => k.nominal == null)
  const sisaUntukBagi = uangDimiliki.value - (batasATM.value || 0) - totalTerisi
  const bagi = kosong.length ? Math.floor(sisaUntukBagi / kosong.length) : 0
  kosong.forEach(k => k.nominal = bagi)

  // Hitung ulang total keperluan
  const total = keperluan.value.reduce((a, b) => a + hitungNominal(b.nominal, b.satuan), 0)

  const sisa = uangDimiliki.value - (batasATM.value || 0) - total

  // PDF output
  const doc = new jsPDF()
  doc.text(`Periode: ${periode.value}`, 10, 10)
  doc.text(`Uang Dimiliki: Rp ${uangDimiliki.value.toLocaleString()}`, 10, 20)
  doc.text(`Batas ATM: Rp ${batasATM.value ? batasATM.value.toLocaleString() : 0}`, 10, 30)

  let y = 40
  keperluan.value.forEach((k, i) => {
    const hasil = hitungNominal(k.nominal, k.satuan)
    doc.text(`${i + 1}. ${k.nama} (${k.satuan}) - ${getLabelPerkalian(k.nominal, k.satuan)}`, 10, y)
    y += 10
  })

  doc.text(`Total Keperluan: Rp ${total.toLocaleString()}`, 10, y + 10)
  doc.text(`Sisa Anggaran: Rp ${sisa.toLocaleString()}`, 10, y + 20)

  const namaFile = `catatan-keuangan-${periode.value}-${new Date().toISOString().slice(0, 10)}.pdf`
  doc.save(namaFile)

  // Reset form
  periode.value = ''
  uangDimiliki.value = null
  batasATM.value = null
  keperluan.value = [{ nama: '', nominal: null, satuan: 'total' }]
}
</script>

<style scoped>
.form-container {
  max-width: 600px;
  margin: auto;
  padding: 1rem;
  font-family: sans-serif;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.keperluan-row {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

input,
select {
  padding: 0.5rem;
  width: 100%;
  max-width: 300px;
}

button {
  cursor: pointer;
}

.submit-btn {
  background-color: #4CAF50;
  color: white;
  padding: 0.7rem;
  border: none;
  border-radius: 5px;
}

.error {
  color: red;
}
</style>
