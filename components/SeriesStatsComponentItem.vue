<template>
  <div>
    <template v-if="sortingOnShop">
      <div class="flex flex-col">
        <div class="flex flex-row justify-between gap-1">
          <div class="flex flex-col">
            <div class="flex gap-2 text-[0.7rem] items-center">
              <div class="flex gap-1">
                <span class="text-details">Stock:</span>
                <div class="flex gap-0.25 items-center">
                  <span class="text-white">{{ Math.max((item.series.total_quantity - item.series.total_sold), 0) }}</span>
                </div>
              </div>
            </div>
            <div class="flex gap-2 text-[0.7rem] items-center">
              <div class="flex gap-1">
                <span class="text-details">Percentage:</span>
                <div class="flex gap-0.25 items-center">
                  <span class="text-amber-500">{{ Math.round((item.series.total_sold / item.series.total_quantity) * 100 ) }}%</span>
                </div>
              </div>
            </div>
          </div>
          <div class="flex flex-col items-end">
            <template v-if="item.series.total_sold < item.series.total_quantity">
              <div class="flex gap-2 text-[0.7rem] items-center">
                <div class="flex gap-1">
                  <span class="text-white/40">Next Mint:</span>
                  <div class="flex gap-0.25 items-center">
                    <span class="text-amber-500">#{{ item.series.total_quantity - (item.series.total_quantity - item.series.total_sold) + 1 }}</span>
                  </div>
                </div>
              </div>
            </template>
            <template v-if="lowestListing">
              <div class="flex gap-2 text-[0.7rem] items-center">
                <div class="flex gap-1">
                  <span class="text-white/40">Mint Profit:</span>
                  <div class="flex items-center">
                    <div :class="{ 'text-green-500': mintProfitInPercentage >= 0, 'text-red-500': mintProfitInPercentage < 0 }">{{ mintProfitInPercentage }}%</div>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </div>
        <div class="py-1 flex items-center">
          <div class="w-full bg-amber-900 rounded-md overflow-hidden">
            <div class="h-2 bg-amber-600 text-[0.65rem] text-white/80 text-center rounded-md overflow-hidden" :style="{ 'width': `${Math.min(100, Math.round((item.series.total_sold / item.series.total_quantity) * 100 ))}%` }"></div>
          </div>
        </div>
      </div>
    </template>
    <template v-else>
      <div class="flex flex-row justify-between gap-1">
        <div class="flex flex-col">
          <div class="flex gap-2 items-center">
            <div class="flex gap-1 text-[0.7rem]">
              <span class="text-details">Floor:</span>
              <template v-if="lowestListing">
                <div class="flex flex-col">
                  <ListingPrice :listing="lowestListing" />
                </div>
              </template>
              <template v-else>
                <div class="flex text-[0.7rem]">
                  <span class="text-white/40">No listings found.</span>
                </div>
              </template>
            </div>
          </div>
          <div class="flex text-[0.7rem]">
            <div class="flex gap-1">
              <div class="flex gap-0.5 items-center">
                <ClockIcon class="text-details w-3 h-3" />
                <span class="text-details">Sale:</span>
              </div>
              <template v-if="item.stats.last_sale">
                <LastSale :sale="item.stats.last_sale" />
              </template>
              <template v-else>
                <span class="text-white/40">No sales yet.</span>
              </template>
            </div>
          </div>
          <div class="flex text-[0.7rem]">
            <div class="flex gap-1">
              <template v-if="sorting === 'lowestFloorMintRatio' || sorting === 'highestFloorMintRatio'">
                <div class="flex items-center gap-0.5 overflow-hidden">
                  <span class="text-details">Mint Profit:</span>
                  <div class="flex gap-0.25 items-center">
                    <div :class="{ 'text-green-500': mintProfitInPercentage >= 0, 'text-red-500': mintProfitInPercentage < 0 }">{{ mintProfitInPercentage }}%</div>
                  </div>
                </div>
              </template>
              <template v-else-if="sorting === 'lowestListedPercentage'">
                <span class="text-details">Listed:</span>
                <div class="flex gap-0.25 items-center">
                  <div class="text-white/80">{{ item.stats.listed_percentage.toFixed(2) }}%</div>
                </div>
              </template>
              <template v-else-if="sorting === 'lowestWeeklyAverage' || sorting === 'highestWeeklyAverage'">
                <span class="text-details">7d Avg:</span>
                <div class="flex gap-0.25 items-center">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                  <div class="text-white/80">{{ (item.stats.weekly_average_price ?? 0).toFixed(4).replace(/\.?0+$/, '') }}</div>
                </div>
              </template>
              <template v-else-if="sorting === 'lowestTwoWeeklyAverage' || sorting === 'highestTwoWeeklyAverage'">
              <span class="text-details">14d Avg:</span>
              <div class="flex gap-0.25 items-center">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                <div class="text-white/80">{{ (item.stats.two_weekly_average_price ?? 0).toFixed(2) }}</div>
              </div>
              </template>
              <template v-else-if="sorting === 'lowestMonthlyAverage' || sorting === 'highestMonthlyAverage'">
                <span class="text-details">30d Avg:</span>
                <div class="flex gap-0.25 items-center">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                  <div class="text-white/80">{{ (item.stats.monthly_average_price ?? 0).toFixed(2) }}</div>
                </div>
              </template>
              <template v-else-if="sorting === 'artistAsc' || sorting === 'artistDesc'">
                <span class="text-details">Artist:</span>
                <div class="flex gap-0.25 items-center">
                  <a :href="`https://reddit.com/u/${item.collection.artist.displayName}`" target="_blank" class="text-amber-500">{{ item.collection.artist.displayName }}</a>
                </div>
              </template>
              <template v-else-if="sorting === 'lowestDailyVolume' || sorting === 'highestDailyVolume'">
                <span class="text-details">24h Vol:</span>
                <div class="flex gap-0.25 items-center">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                  <div class="text-white/80">{{ item.stats.daily_volume.toFixed(2) }}</div>
                </div>
              </template>
              <template v-else-if="sorting === 'lowestWeeklyVolume' || sorting === 'highestWeeklyVolume'">
                <span class="text-details">7d Vol:</span>
                <div class="flex gap-0.25 items-center">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                  <div class="text-white/80">{{ item.stats.weekly_volume.toFixed(2) }}</div>
                </div>
              </template>
              <template v-else>
                <div class="flex gap-0.5 items-center">
                  <ClockIcon class="text-details w-3 h-3" />
                  <span class="text-details">5 Sales Avg:</span>
                </div>
                <div class="flex gap-0.25 items-center">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                  <div class="text-white/80">{{ item.stats.five_last_sales_average.toFixed(4).replace(/\.?0+$/, '') }}</div>
                </div>
              </template>
            </div>
          </div>
        </div>
        <div class="absolute right-0 flex flex-col items-end">
          <div class="flex text-[0.7rem]">
            <div class="flex gap-1">
              <span class="text-white/40">24h:</span>
              <template v-if="item.stats.daily_price_change > 0">
                <div class="flex gap-0.5 items-center text-green-500 text-[0.65rem] rounded">
                  <span>+{{ dailyPriceChange }}%</span>
                </div>
              </template>
              <template v-else-if="item.stats.daily_price_change < 0">
                <div class="flex gap-0.5 items-center text-red-500 text-[0.65rem] rounded">
                  <span>{{ item.stats.daily_price_change.toFixed(2) }}%</span>
                </div>
              </template>
              <template v-else>
                <div class="flex gap-0.5 items-center text-neutral-400 text-[0.65rem] rounded">
                  <span>0%</span>
                </div>
              </template>
            </div>
          </div>
          <div class="flex text-[0.7rem]">
            <div class="flex gap-1">
              <span class="text-white/40">Vol:</span>
              <div class="flex gap-0.25 items-center">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512" fill="currentColor" class="w-3 h-3 text-details"><path d="M311.9 260.8L160 353.6 8 260.8 160 0l151.9 260.8zM160 383.4L8 290.6 160 512l152-221.4-152 92.8z"></path></svg>
                <div class="text-details">{{ (item.stats.total_volume / 1000000000000000000).toFixed(2) }}</div>
              </div>
            </div>
          </div>
          <div class="flex text-[0.7rem]">
            <div class="flex gap-1">
              <span class="text-white/40">Cap:</span>
              <div class="flex gap-0.25 items-center">
                <div class="text-details">{{ ethereumInLocalCurrency(item.series.total_sold * item.stats.last_sale?.payment_token.base_price ?? 0, true) }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import {SeriesStats} from "~/types/seriesStats";
import {PropType} from "@vue/runtime-core";
import {
  useWatchList,
  useEthereumUsdPrice,
  useEthereumPriceMap
} from "~/composables/states";
import {computed, ethereumInLocalCurrency} from "#imports";
import {ETH_TO_GWEI_MODIFIER} from "~/types/ethereum";
import {getLowestListing, getListingAsGweiPrice} from "~/composables/helpers";
import {ClockIcon} from "@heroicons/vue/24/outline";

const watchList = useWatchList();
const ethereumPriceInUsd = useEthereumUsdPrice();
const ethereumPriceMap = useEthereumPriceMap();

const props = defineProps({
  item: {
    type: Object as PropType<SeriesStats>,
    required: true
  },
  sorting: String
});

const sortingOnShop = computed(() => {
  return props.sorting === 'highestShopAvailableAbsolute' || props.sorting === 'lowestShopAvailableAbsolute' || props.sorting === 'highestShopAvailablePercentage' || props.sorting === 'lowestShopAvailablePercentage' || props.sorting === 'lowestShopNextMint';
});

const lowestListing = computed(() => {
  return getLowestListing(props.item);
});

const mintProfitInPercentage = computed(() => {
  return Math.round(((getListingAsGweiPrice(lowestListing.value) / ETH_TO_GWEI_MODIFIER) * ethereumPriceInUsd.value) / (props.item.series.mint_price / 100) * 100 -100);
});

const dailyPriceChange = computed(() => {
  let change = props.item.stats.daily_price_change;

  if (change >= 100 || change <= -100) {
    return change.toFixed(0);
  }

  return change.toFixed(2);
});
</script>

<style scoped>

</style>
