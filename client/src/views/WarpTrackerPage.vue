
<template>
  <div class="flex flex-col gap-2 w-full">
    <div class="flex justify-between items-center">
      <div class="flex flex-col">
        <div>
          <h1 class="text-4xl pb-2 text-title font-bold">
            {{ $t('wh.innerTitle') }}
          </h1>
          <p v-html="$t('wh.intro')" />
        </div>
        <!-- <Note messages="The following page has yet to been optimised for smaller screen devices/windows.,The following page still has features in-development."/> -->
        <Note :messages="$t('wh.tips')" />
      </div>
      <div class="flex flex-col gap-2">
        <button
          class="button p-3 px-20 font-bold text-xl inline-image justify-center items-center"
          @click="redirect('/warp/import')"
        >
          <img src="/images/other/import.png">{{ $t('wh.menu.import') }}
        </button>
        <button
          class="button p-3 px-20 font-bold text-xl inline-image justify-center items-center"
          @click="redirect('/warp/global')"
        >
          <img src="/images/other/earth.png">{{ $t('wh.menu.globalStatus') }}
        </button>
      </div>
    </div>
    <Split />
    <div class="flex flex-row gap-10">
      <div class="w-40 min-w-[10rem] flex flex-col gap-2 child:drop-shadow-2xl child:transition-all child:duration-75">
        <button
          :class="[{ 'pl-0': banner == 'character' }, { 'pl-8': banner != 'character' }]"
          @click="switchBanner('character')"
        >
          <img :src="`/images/tickets/${currentBanner}.png`">
        </button>
        <button
          :class="[{ 'pl-0': banner == 'light_cone' }, { 'pl-8': banner != 'light_cone' }]"
          @click="switchBanner('light_cone')"
        >
          <img :src="`/images/tickets/${currentBanner}_lightCone.png`">
        </button>
        <button
          :class="[{ 'pl-0': banner == 'regular' }, { 'pl-8': banner != 'regular' }]"
          @click="switchBanner('regular')"
        >
          <img src="/images/other/regularWarp.png">
        </button>
      </div>
      <div
        v-if="!warps"
        class="bgcontainer w-full h-[19rem] flex flex-col justify-center items-center"
      >
        <p class="text-description">
          {{ $t('wh.emptyData') }}
        </p>
        <p class="text-title text-2xl">
          {{ $t('wh.emptyDataHint') }}
        </p>
      </div>
      <div v-if="warps">
        <h1 class="text-xl pb-2 text-title capitalize">
          {{ $t(`wh.warpBannerTitle.${banner.replace('_', '')}`) }}
        </h1>
        <div class="flex flex-col gap-5">
          <div class="flex flex-row gap-5">
            <div class="flex flex-col gap-2 w-72">
              <div class="bgcontainer p-2 flex flex-row justify-between items-center">
                <div>
                  <p class="text-title">
                    {{ $t('wh.lifetimePulls') }}
                  </p>
                  <span class="text-description inline-image"><img src="/images/other/stellar_jade.webp">{{ (160 * pulls.lifetime).toLocaleString() }}</span>
                </div>
                <p class="text-4xl">
                  {{ pulls.lifetime }}
                </p>
              </div>
              <div class="bgcontainer p-2 flex flex-row justify-between items-center">
                <div>
                  <p class="text-title inline-image">
                    5<img src="/images/other/star.png"> {{ $t('wh.pity') }}
                  </p>
                  <span class="text-description inline-image">{{ $t('wh.guaranteed90') }}</span>
                </div>
                <p class="text-4xl text-legendary">
                  {{ pulls.legendary }}
                </p>
              </div>
              <div class="bgcontainer p-2 flex flex-row justify-between items-center">
                <div>
                  <p class="text-title inline-image">
                    4<img src="/images/other/star.png"> {{ $t('wh.pity') }}
                  </p>
                  <span class="text-description inline-image">{{ $t('wh.guaranteed10') }}</span>
                </div>
                <p class="text-4xl text-rare">
                  {{ pulls.rare }}
                </p>
              </div>
            </div>
            <div class="bgcontainer w-[37.25rem] h-[16.75rem] p-2">
              <WarpHistory
                v-if="monthlyPulls"
                :monthly-pulls="monthlyPulls"
              />
              <div
                v-if="!monthlyPulls"
                class="flex justify-center items-center w-full h-full"
              >
                <p class="text-description">
                  No monthly pull data to display.
                </p>
              </div>
            </div>
          </div>
          <div>
            <div class="bgcontainer flex flex-col w-[37.25rem] gap-3 p-5">
              <div class="flex flex-row justify-center gap-3 child:h-12 child:w-12 child:justify-center child:items-center child:pl-2 child:transition-colors">
                <button
                  :class="{ 'bg-[#ffb13f] bg-opacity-20': filters.legendary }"
                  class="buttonSquare inline-image"
                  @click="filterWarps('legendary')"
                >
                  5<img src="/images/other/star.png">
                </button>
                <button
                  :class="{ 'bg-[#ffb13f] bg-opacity-20': filters.rare }"
                  class="buttonSquare inline-image hue-rotate-[250deg]"
                  @click="filterWarps('rare')"
                >
                  4<img src="/images/other/star.png">
                </button>
                <button
                  :class="{ 'bg-[#ffb13f] bg-opacity-20': filters.common }"
                  class="buttonSquare inline-image hue-rotate-180"
                  @click="filterWarps('common')"
                >
                  3<img src="/images/other/star.png">
                </button>
              </div>
              <table class="w-full text-left">
                <tr class="text-title child:font-normal">
                  <th class="pl-5">
                    {{ $t('wh.record.time') }}
                  </th>
                  <th class="text-center">
                    {{ $t('wh.record.pity') }}
                  </th>
                  <th>{{ $t('wh.record.name') }}</th>
                </tr>
                <tr
                  v-for="(warp, index) in warps"
                  :key="index"
                  :class="`warp-${warp[4]}`"
                  class="border-t border-gray-900 child:font-normal child:py-2"
                >
                  <th class="w-[12.5rem] pl-5 text-sm font-mono">
                    {{ timeToDay(warp[3]) }}
                  </th>
                  <th
                    :style="`color: hsl(${warp[6]},100%,70%)`"
                    class="text-center pr-2"
                  >
                    {{ warp[5] }}
                  </th>
                  <th class="inline-image gap-3 capitalize">
                    <img
                      :class="[{ 'scale-[1.5]': warp[2] == 'light_cone' }, { 'scale-[2]': warp[2] == 'character' }]"
                      :src="`/images/${warp[2]}/${warp[1]}.webp`"
                    >{{ warp[1].replaceAll("_", " ") }}
                  </th>
                  <th>
                    <Tooltip
                      v-if="warp[7]"
                      :message="warp[7] == 1 && 'Lost 50:50' || warp[7] == 2 && 'Guaranteed' || 'Won 50:50'"
                      :icon="warp[7] == 1 && 'starRed' || warp[7] == 2 && 'starEmpty' || 'star'"
                    />
                  </th>
                </tr>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
import * as vueRouter from 'vue-router'
import Note from '../components/TheNote.vue'
import Split from '../components/TheSplit.vue'
import Tooltip from '../components/TheTooltip.vue'
import WarpHistory from '../components/WarpTracker/WarpHistory.vue'
import {readonly} from 'vue'

export default {
	components:{
		Note,
		Split,
		Tooltip,
		WarpHistory
	},	
	setup(){
		const router = vueRouter.useRouter()
		const days = readonly({ 0: 'Sun', 1: 'Mon', 2: 'Tue', 3: 'Wed', 4: 'Thu', 5: 'Fri', 6: 'Sat' })
		const redirect = (route)=> {
		router.push({
			path: route,
		})
		}
		return {
			router,
			days,
			redirect
		}
	},
  data() {
    return {
      currentBanner: 'contract_zero',
      warps: null,
      banner: 'character',
      pulls: {
        lifetime: 0,
        legendary: 0,
        rare: 0,
      },
      filters: {
        legendary: true,
        rare: true,
        common: true,
      },
      monthlyPulls: null,
    }
  },
  mounted() {
    this.getWarps(10, this.banner)
  },
  methods: {
    switchBanner(banner) {
      this.banner = banner
      this.getWarps(10, this.banner)
    },
    filterWarps(filter) {
      this.filters[filter] ^= true
      this.getWarps(10, this.banner)
    },
    timeToDay(time) {
      return `${this.days[new Date(time).getDay()]} ${time}`
    },
    getWarps(amount, banner) {
      const warps = JSON.parse(localStorage.getItem(`warps_${  banner}`))

      if (warps) {
        const lifetimeWarps = warps.lifetime

        this.pulls.lifetime = warps.lifetime
        this.pulls.legendary = warps.pity[0]
        this.pulls.rare = warps.pity[1]

        const filtered = warps.data.filter(warp => this.filters[warp[4]]).slice(0, amount)

        this.monthlyPulls = Object.fromEntries(Object.entries(warps.monthlyPulls).slice(0, 5))
        this.warps = filtered
      }

      return null
    },
  },
}
</script>
