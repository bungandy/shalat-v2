<template>
  <div class="flex flex-col w-screen max-w-screen-sm py-5 px-5 sm:px-0">
    <div class="header">
      <div v-if="!localTime" class="bg-slate-100 rounded-full h-4 w-40"/>
      <div v-else class="text-xs text-slate-400">{{localTime}}</div>

      <h1 class="font-bold">Shalat.co</h1>
    </div>
    <!-- .header -->

    <div class="next-prayer h-60 flex items-center justify-center">
      <div class="flex flex-col items-center space-y-1">
        <div class="text-emerald-600">
          <div v-if="!nextPrayer" class="bg-slate-100 rounded-full h-4 w-40" />
          <template v-else>Next: {{nextPrayer.name}}</template>
        </div>
        <h2 class="text-7xl">
          <div v-if="!nextPrayer" class="bg-slate-100 rounded-full h-14 w-40 my-3" />
          <template v-else>{{nextPrayer.time}}</template>
        </h2>
        <!-- <div class="text-slate-400">
          <div v-if="!nextPrayer" class="bg-slate-100 rounded-full h-4 w-40" />
          <template v-else>00:00</template>
        </div> -->
      </div>
    </div>
    <!-- .next-prayer -->

    <div class="schedule flex flex-col space-y-5">
      <div class="current-location flex items-center space-x-2 border border-emerald-600 rounded-xl p-2">
        <i class="fa-solid fa-location-arrow text-emerald-600"></i>
        <div v-if="!currentLocation">
          <div class="bg-slate-100 rounded-full h-5 w-40 mb-2 mt-1" />
          <div class="bg-slate-100 rounded-full h-3 w-40" />
        </div>
        <div v-else>
          <div class="text-emerald-600">{{currentLocation}}</div>
          <div class="text-xs text-slate-400">Current Location <button @click.prevent="getLocationByBrowser">Get my location</button></div>
        </div>
      </div>
      <div class="list-prayer">
        <div v-if="!prayers" class="grid gap-y-4">
          <div v-for="i in 5" :key="i" class="flex justify-between">
            <div :class="['bg-slate-100 rounded-full h-5 w-1/5', ]"></div>
            <div class="bg-slate-100 rounded-full h-5 w-1/5"></div>
          </div>
        </div>
        <template v-else>
          <div v-for="(prayer, key) in prayers" :key="key"
            :class="['flex items-center justify-between h-10',
              {'border-t border-slate-200' : key !== 'Fajr'},
              {'bg-emerald-500 text-white -mx-2 px-2 rounded-lg border-t-0 font-bold' : key === currentPrayer },
              {'text-slate-400' : prayer.split(':').join() < nowTime }
            ]">
            <span>{{key}}</span>
            <span>{{prayer}}</span>
          </div>
        </template>
      </div>
    </div>
    <!-- .schedule -->

  </div>
</template>

<script>
import dayjs from "dayjs"

export default {
  name: 'App',
  data() {
    return {
      localTime: null,
      prayers: null,
      currentPrayer: null,
      nextPrayer: null,
      currentLocation: null,
    }
  },
  computed: {
    nowTime: () => {
      return dayjs().format('HHmm')
    }
  },
  async created() {
    // get local time
    setInterval(() => {
      this.localTime = dayjs().format('MMMM D, YYYY - HH:mm:ss')
    }, 1000)
  },
  async mounted() {
    // [default] get user location by API
    fetch(`https://geolocation-db.com/json/`)
      .then(response => response.json())
      .then(data => {
        // console.log(data)
        this.currentLocation = `${data.state}, ${data.country_name}`

        // if this have city, use it
        if(data.city){
          this.currentLocation = `${data.city}, ${data.country_name}`
        }

        // get prayers
        this.getPrayers(data.latitude, data.longitude)
      })
  },
  methods: {
    // get user-geolocation
    getLocationByBrowser() {
      navigator.geolocation.getCurrentPosition(
        position => {
          // console.log(position.coords.latitude);
          // console.log(position.coords.longitude);
          getPrayers(position.coords.latitude, position.coords.longitude)
        },
        error => {
          console.log(error.message);
        },
      )
    },

    // get prayers time
    getPrayers(latitude,longitude) {
      const prayerUrl = 'https://api.pray.zone'
      const today = dayjs().format('YYYY-MM-DD')

      fetch(`${prayerUrl}/v2/times/day.json?latitude=${latitude}&longitude=${longitude}&elevation=0&date=${today}&key=MagicKey`)
        .then(response => response.json())
        .then(data => {
          // console.log(data)          
          const results = data.results.datetime[0].times
          const fajrEnd = results['Sunrise']
          
          // delete unused times
          delete results['Imsak']
          delete results['Sunrise']
          delete results['Sunset']
          delete results['Midnight']

          let isNextPrayer = null

          Object.entries(results).map((item) => {
            const now = dayjs().format('HHmm')
            const prayerTime = item[1].split(':').join('')

            // set current prayer
            if(now >= prayerTime && now < fajrEnd){
              this.currentPrayer = item[0]
            }

            // set next prayer
            if(prayerTime >= now && !isNextPrayer){
              isNextPrayer = { name: item[0], time: item[1] }
              // setTimeout(() => {
                this.nextPrayer = isNextPrayer
              // }, 2000)
            }
          })

          // asign used times value
          this.prayers = results
        });
    }
  }
}
</script>
