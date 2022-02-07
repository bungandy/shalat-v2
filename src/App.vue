<template>
  <div class="flex flex-col w-screen">
    <div class="header flex flex-row">
      <div>
        <p>7 Feb, 2022</p>
        <h1>Shalat.co</h1>
      </div>
      <div>
        <p>Koordinat</p>
      </div>
    </div>
    <!-- .header -->

    <div class="next-prayer h-96">
      <p>Next: Asr</p>
      <h2>15:27</h2>
      <p>00:00:00</p>
    </div>
    <!-- .next-prayer -->

    <div class="schedule">
      <div class="list-prayer">
        <div class="flex justify-between">
          <span>Fajr</span>
          <span>04:39</span>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>
