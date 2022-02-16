<template>
  <div class="flex flex-col justify-between w-screen max-w-screen-sm py-5 px-5 sm:px-0 space-y-4">
    <div class="flex flex-col space-y-5">
      <div class="header">
        <div v-if="!localTime" class="bg-slate-100 rounded-full h-4 w-40"/>
        <div v-else class="text-xs text-slate-400">{{localTime}}</div>

        <h1 class="font-bold">Shalat.co</h1>
      </div>
      <!-- .header -->

      <div class="next-prayer h-40 flex items-center justify-center">
        <div class="flex flex-col items-center space-y-1">
          <div class="text-indigo-700">
            <div v-if="!nextPrayer" class="bg-slate-100 rounded-full h-4 w-40" />
            <template v-else>{{nextPrayer.name}}</template>
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
        <div class="current-location flex items-center space-x-2 bg-indigo-50 border border-indigo-700 rounded-xl p-2">
          <i class="fa-solid fa-location-arrow text-indigo-700"></i>
          <div v-if="!currentLocation.name">
            <div class="bg-indigo-100 rounded-full h-5 w-40 mb-2 mt-1" />
            <div class="bg-indigo-100 rounded-full h-3 w-40" />
          </div>
          <div v-else>
            <div class="text-indigo-700">{{currentLocation.name}}</div>
            <div class="text-xs text-indigo-400">Current Location</div>
          </div>
        </div>
        
        <button @click.prevent="getLocationByDevice" class="underline text-xs text-slate-400 h-9 appearance-none">
          <template v-if="isLoading">
            <i class="fa-solid fa-loader fa-spin fa-2x"></i>
          </template>
          <template v-else>
            Refresh location
          </template>
        </button>

        <div class="list-prayer mt-0">
          <div v-if="!prayers" class="grid gap-y-4">
            <div v-for="i in 5" :key="i" class="flex justify-between">
              <div :class="['bg-slate-100 rounded-full h-7 w-1/5', ]"></div>
              <div class="bg-slate-100 rounded-full h-7 w-1/5"></div>
            </div>
          </div>
          <template v-else>
            <div v-for="(prayer, key) in prayers" :key="key"
              :class="['flex items-center justify-between h-11',

                // hide border top for Fajr
                {'border-t border-slate-200' : key !== 'Fajr'},

                // highlight for current prayer
                {'bg-indigo-500 text-white -mx-2 px-2 rounded-lg border-t-0 font-bold' : key === currentPrayer },

                // text muted for passed prayer
                {'text-slate-400' : nowTime > prayer.split(':').join() }
              ]">
              <span>{{key}}</span>
              <span class="font-bold">{{prayer}}</span>
            </div>
          </template>
        </div>
      </div>
      <!-- .schedule -->
    </div>
    
    <div @click.prevent="isShowQr = !isShowQr" class="relative border border-slate-200 rounded p-3 cursor-pointer">
      <p class="text-xs text-slate-400 text-center">Donate ETH <i class="fa-brands fa-ethereum"></i> <span class="font-bold">{{ethWallet}}</span></p>
      <div v-show="isShowQr" class="absolute z-10 -top-3 left-1/2 -translate-y-full -translate-x-1/2 bg-white px-4 pt-5 pb-3 border border-slate-200 rounded flex flex-col items-center space-y-2">
        <QrcodeVue :value="`ethereum:${ethWallet}`" size="250"/>
        <!-- <QrcodeVue :value="ethWallet" size="250"/> -->
        <div class="text-xs text-slate-400"><i class="fa-solid fa-mobile-screen-button"></i> Scan</div>
        <div class="absolute bg-white bottom-0 left-1/2 -translate-x-1/2 translate-y-1/2 h-3 w-3">
          <span class="absolute top-0 left-0 inline-block h-3 w-3 rotate-45 border border-slate-200 border-l-0 border-t-0"></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs"
import QrcodeVue from 'qrcode.vue'

export default {
  name: 'App',
  components: {
    QrcodeVue,
  },
  data() {
    return {
      localTime: null,
      prayers: null,
      currentPrayer: null,
      nextPrayer: null,
      currentLocation: {
        name: null,
        time: null
      },
      isLoading: false,
      ethWallet: '0xB3BeD53b13164a99D71b3Abf9652ED4Eba764b84', // Metamask Andy
      // ethWallet: '0x370af8728cb6C10ccF099335f4738d56269a079A', // Luno Andy
      isShowQr: false
    }
  },
  computed: {
    nowTime: () => {
      return dayjs().format('HHmm')
      // return '2000'
    }
  },
  async created() {
    // get local time
    setInterval(() => {
      this.localTime = dayjs().format('MMMM D, YYYY - HH:mm:ss')
    }, 1000)
  },
  async mounted() {
    // if geolocation allowed
    if(localStorage.getItem('location'))
      this.getLocationByDevice()
    else
      this.getLocationByAPI()
  },
  methods: {
    // get location using api
    getLocationByAPI() {
      fetch(`https://geolocation-db.com/json/`)
        .then(response => response.json())
        .then(data => {
          // console.log(data)

          // change display city name
          this.changeCityName(data.country_name, data.state, data.city)

          // get prayers
          const today = dayjs().format('YYYY-MM-DD')
          this.getPrayers(today, data.latitude, data.longitude, 'geolocation-db')
        })
    },

    // get user-geolocation
    getLocationByDevice() {
      // set loading
      this.isLoading = true

      // get user location
      navigator.geolocation.getCurrentPosition(
        position => {
          // console.log(position.coords.latitude);
          // console.log(position.coords.longitude);
          const today = dayjs().format('YYYY-MM-DD')
          this.currentLocation.latitude = position.coords.latitude
          this.currentLocation.longitude = position.coords.longitude
        
          // get address by coords
          const apiKey = import.meta.env.VITE_LOCATIONIQ_API_KEY
          fetch(`https://us1.locationiq.com/v1/reverse.php?key=${apiKey}&lat=${position.coords.latitude}&lon=${position.coords.longitude}&format=json`)
            .then(response => response.json())
            .then(data => {
              // console.log(data)

              // change display city name
              const address = data.address
              this.changeCityName(address.country, address.city, address.city_district)

              // change local storage
              const locationStorage = {
                latitude: position.coords.latitude,
                longitude: position.coords.longitude
              }
              this.setLocalStorage("location", locationStorage)

            })

          // get prayers
          this.getPrayers(today, position.coords.latitude,  position.coords.longitude, 'browser-geolocation')

        },
        error => {
          console.log(error.message);
        },
      )
    },

    // get prayers time
    getPrayers(date, latitude, longitude, note) {
      const prayerUrl = 'https://api.pray.zone'

      fetch(`${prayerUrl}/v2/times/day.json?latitude=${latitude}&longitude=${longitude}&elevation=0&date=${date}&key=MagicKey`)
        .then(response => response.json())
        .then(data => {
          console.log(data)    
          
          const sunrise = data.results.datetime[0].times['Sunrise'].split(':').join('')
          const dhuhr = data.results.datetime[0].times['Dhuhr'].split(':').join('')
          const midnight = data.results.datetime[0].times['Midnight'].split(':').join('')
          const isha = data.results.datetime[0].times['Isha'].split(':').join('')
          
          const results = data.results.datetime[0].times

          // delete unused times
          delete results['Imsak']
          delete results['Sunrise']
          delete results['Sunset']
          delete results['Midnight']

          // set next prayer
          let isNextPrayer = null
          this.nextPrayer = null

          
          const now = dayjs().format('HHmm')
          // const now = '2000'

          // manipulate prayer
          Object.entries(results).map((item) => {
            
            const prayerTime = item[1].split(':').join('')

            // set current prayer
            if( now >= prayerTime ){
              this.currentPrayer = item[0]
              if( now > sunrise && now < dhuhr ){
                this.currentPrayer = null
              }
            }

            // set next prayer
            if(prayerTime >= now && !this.nextPrayer){
              isNextPrayer = { name: `Next: ${item[0]}`, time: item[1] }
              this.nextPrayer = isNextPrayer
            }
          })

          // next prayer is tomorrow fajr
          if(now >= isha){
            const tomorrow = dayjs().add(1, 'day').format('YYYY-MM-DD')
            fetch(`${prayerUrl}/v2/times/day.json?latitude=${latitude}&longitude=${longitude}&elevation=0&date=${tomorrow}&key=MagicKey`)
              .then(response => response.json())
              .then(data => {
                const results = data.results.datetime[0].times
                this.nextPrayer = {name: 'Tomorrow: Fajr', time: results['Fajr'] }
              })
          }

          // asign used times value
          this.prayers = results

          // set loading
          this.isLoading = false

          // console the getPrayers triggered by
          console.warn(`getPrayers(${date}, ${latitude}, ${longitude}, ${note})`)
        });
    },

    // change display city name
    changeCityName(country, state, city) {
      let displayCity = country
        
      // if have state
      if(state)
        displayCity = `${state}, ${country}`
      

      // if this have city, use it
      if(city)
        displayCity = `${city}, ${state}`

      // change display city name
      this.currentLocation.name = displayCity
    },

    setLocalStorage(key, value){
      localStorage.setItem(key, JSON.stringify(value))
    }
  }
}
</script>
