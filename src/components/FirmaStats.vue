<template>
  <div >
    <h1>Organisasjonsstatistikk</h1>
    <br><br>

    <div v-if="stats">
        

      <div>
        <h2 >Status (antall)</h2>
        <div >
          <ul >
            <li v-for="(count, status) in stats.statusCount" :key="status">
              {{ status }}: {{ count }}
            </li>
          </ul>
          <div >
            <Pie :data="chartData.status" :options="{ ...chartOptions, maintainAspectRatio: false }" />
          </div>
        </div>
      </div>

      <div>
        <h2 >Organisasjonsformer (%)</h2>
        <div >
          <ul >
            <li v-for="(percent, form) in stats.orgFormPercent" :key="form">
              {{ form }}: {{ percent.toFixed(1) }} %
            </li>
          </ul>
          <div>
            <Doughnut :data="chartData.orgForm" :options="{ ...chartOptions, maintainAspectRatio: false }" />
          </div>
        </div>
      </div>

      <div >
        <h2>Ansattefordeling</h2>
        <div>
          <ul >
            <li>0 ansatte: {{ stats.ansatteKategorier["0"] }}</li>
            <li>1-9 ansatte: {{ stats.ansatteKategorier["1-9"] }}</li>
            <li>10-49 ansatte: {{ stats.ansatteKategorier["10-49"] }}</li>
            <li>50+ ansatte: {{ stats.ansatteKategorier["50+"] }}</li>
          </ul>
          <div>
            <Bar :data="chartData.ansatte" :options="{ ...chartOptions, maintainAspectRatio: false }" />
          </div>
        </div>
      </div>

    </div>

    <div v-else >Laster data...</div>
  </div>
</template>




<script setup>
import { ref, onMounted } from 'vue'
import Papa from 'papaparse' // Importer papaparse for CSV parsing

// Chart.js imports
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  ArcElement,
  BarElement,
  CategoryScale,
  LinearScale
} from 'chart.js'

import { Pie, Doughnut, Bar } from 'vue-chartjs'

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  ArcElement,
  BarElement,
  CategoryScale,
  LinearScale
)

const stats = ref(null)
const chartData = ref({
  status: null,
  orgForm: null,
  ansatte: null
})
const chartOptions = {
  responsive: true,
  plugins: {
    legend: {
      position: 'right'
    },
    tooltip: {
      enabled: true
    }
  }
};

//onMounted : Kjører etter at komponenten er satt inn i DOM-en
onMounted(async () => {
  const response = await fetch('/firmaer_output.csv')
  const text = await response.text()

  Papa.parse(text, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      const data = results.data.filter(row => row.OrgNo) 

      const statusCount = {}
      const orgFormCount = {}
      const ansatteKategorier = {
        '0': 0,
        '1-9': 0,
        '10-49': 0,
        '50+': 0
      }

      let totalOrgForms = 0

      data.forEach(row => {
        const status = row.Status?.trim()
        const form = row.OrganisasjonsformKode?.trim()
        const ansatte = parseInt(row.AntallAnsatte)

        //status
        if (status) {
          statusCount[status] = (statusCount[status] || 0) + 1
        }

        //organisasjonsform
        if (form) {
          orgFormCount[form] = (orgFormCount[form] || 0) + 1
          totalOrgForms++
        }

        //Ansattekategorier
        if (!isNaN(ansatte)) {
          if (ansatte === 0) ansatteKategorier['0']++
          else if (ansatte <= 9) ansatteKategorier['1-9']++
          else if (ansatte <= 49) ansatteKategorier['10-49']++
          else ansatteKategorier['50+']++
        }
      })

      // Konverter antall til prosent
      const orgFormPercent = {}
      for (const form in orgFormCount) {
        orgFormPercent[form] = (orgFormCount[form] / totalOrgForms) * 100
      }

      stats.value = {
        statusCount,
        orgFormPercent,
        ansatteKategorier
      }

       // Klargjør data til Chart.js
      chartData.value.status = {
        labels: Object.keys(statusCount),
        datasets: [
          {
            label: 'Antall firma per status',
            data: Object.values(statusCount),
            //5 Statuskoder, 5 forskjellige farger
            backgroundColor: [
                    '#60a5fa', // blue-400
                    '#a78bfa', // purple-400
                    '#f472b6', // pink-400
                    '#34d399', // green-400
                    '#fbbf24' // yellow-400
                    ]
          }
        ]
      }

      chartData.value.orgForm = {
        labels: Object.keys(orgFormPercent),
        datasets: [
          {
            label: 'Organisasjonsform (%)',
            data: Object.values(orgFormPercent),
            //12 orgformer, 12 Forskjellige farger
            backgroundColor: [
                    '#60a5fa', // blue-400
                    '#a78bfa', // purple-400
                    '#f472b6', // pink-400
                    '#34d399', // green-400
                    '#fbbf24', // yellow-400
                    '#fb7185', // rose-400
                    '#38bdf8', // sky-400
                    '#facc15', // amber-400
                    '#4ade80', // emerald-400
                    '#f97316', // orange-400
                    '#818cf8', // indigo-400
                    '#c084fc'  // violet-400
                ]
          }
        ]
      }

      chartData.value.ansatte = {
        labels: Object.keys(ansatteKategorier),
        datasets: [
          {
            label: 'Antall firma per ansattkategori',
            data: Object.values(ansatteKategorier),
            //4 ansattkategorier, 4 forskjellige farger
            backgroundColor: [
                    '#fb7185', // rose-400
                    '#38bdf8', // sky-400
                    '#facc15', // amber-400
                    '#4ade80' // emerald-400
                    ]
          }
        ]
      }

    }
  })
})
</script>

<style scoped>
    
</style>