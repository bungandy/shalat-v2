<template>
  <div class="flex flex-col w-screen p-5">
    <div class="header">
      <div v-if="!localTime" class="bg-slate-100 rounded-full h-4 w-40"/>
      <div v-else class="text-xs text-slate-400">{{localTime}}</div>

      <h1 class="font-bold">Shalat.co</h1>
    </div>
    <!-- .header -->

    <div class="next-prayer h-60 flex items-center justify-center">
      <div class="flex flex-col items-center space-y-1">
        <div class="text-emerald-600">Next: {{nextPrayer.name}}</div>
        <h2 class="text-7xl">{{nextPrayer.time}}</h2>
        <div>00:00:00</div>
      </div>
    </div>
    <!-- .next-prayer -->

    <div class="schedule flex flex-col space-y-5">
      <div class="current-location flex items-center space-x-2 border border-emerald-600 rounded-xl p-2">
        <i class="fa-solid fa-location-arrow text-emerald-600"></i>
        <div>
          <div class="text-emerald-600">Ciledug</div>
          <div class="text-xs text-slate-400">Current Location</div>
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
              {'border-t border-slate-200' : key > 0 },
              {'bg-emerald-500 text-white -mx-2 px-2 rounded-lg' : key === 3}
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
      nextPrayer: {
        name: 'Asr',
        time: '15:27'
      },
    }
  },
  computed: {
    isNow: () => {
      
    }
  },
  async created() {
    // get local time
    setInterval(() => {
      this.localTime = dayjs().format('MMMM D, YYYY - hh:mm:ss')
    }, 1000)
    
    // get user geolocation
    navigator.geolocation.getCurrentPosition(
      position => {
        // console.log(position.coords.latitude);
        // console.log(position.coords.longitude);

        // fetch data
        const prayerUrl = 'https://api.pray.zone'
        const today = dayjs().format('YYYY-MM-DD')

        fetch(`${prayerUrl}/v2/times/day.json?latitude=${position.coords.latitude}&longitude=${position.coords.longitude}&elevation=0&date=${today}&key=MagicKey`)
          .then(response => response.json())
          .then(data => {
            const results = data.results.datetime[0].times

            // delete unused times
            delete results['Imsak']
            delete results['Sunrise']
            delete results['Sunset']
            delete results['Midnight']

            // assign value from formated data
            this.prayers = results
          });

        // get address by coords
        const apiKey = 'AIzaSyBsG88vZJKSGWrftZxJu_JoLsTzwmwMtVE'
        fetch(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${position.coords.latitude},${position.coords.longitude}&key=${apiKey}`)
          .then(response => response.json())
          .then(data => {
            console.log(data)
          })
          
      },
      error => {
        console.log(error.message);
      },
    )
  },
}
</script>
