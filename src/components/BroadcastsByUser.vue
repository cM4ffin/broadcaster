<script setup lang="ts">
import { computed, ref } from 'vue';
import { LichessPaginatedBroadcasts } from '../types';
import { useSettingsStore } from '../stores/settings';
import { lichessFetch, openPath } from '../utils';
import { router } from '../router';
import { onBeforeRouteUpdate } from 'vue-router';

import BroadcastSummary from './BroadcastSummary.vue';
import Pagination from './Pagination.vue';
import { useUserStore } from '../stores/user';

const settings = useSettingsStore();
const user = useUserStore();

const username = ref<string>(router.currentRoute.value.params.username as string);
const pageNum = ref<number>(parseInt(router.currentRoute.value.params.pageNum as string));

const isLoading = ref<boolean>(true);
const broadcasts = ref<LichessPaginatedBroadcasts>();

const pageHasBroadcasts = computed<boolean>(() =>
  broadcasts.value?.currentPageResults ? broadcasts.value.currentPageResults.length > 0 : false,
);

function refresh() {
  broadcasts.value = undefined;
  isLoading.value = true;

  lichessFetch(`/api/broadcast/by/${username.value}`, {
    page: pageNum.value.toString(),
  })
    .then(response => response.json() as Promise<LichessPaginatedBroadcasts>)
    .then(data => {
      broadcasts.value = data;
    })
    .finally(() => (isLoading.value = false));
}

if (!pageHasBroadcasts.value) {
  refresh();
}

onBeforeRouteUpdate((to, _from) => {
  username.value = to.params.username as string;
  pageNum.value = parseInt(to.params.pageNum as string);
  refresh();
});

const viewOnLichessUrl = computed<string>(() => {
  return username.value === 'broadcaster'
    ? `${settings.lichessUrl}/broadcast`
    : `${settings.lichessUrl}/broadcast/by/${username.value}`;
});
</script>

<template>
  <div class="md:flex md:items-center md:justify-between mb-4">
    <div class="min-w-0 flex-1"></div>
    <div class="mt-4 flex md:ml-4 md:mt-0 space-x-1">
      <button
        type="button"
        @click="refresh"
        class="inline-flex items-center rounded-md bg-white/10 px-3 py-2 text-sm font-semibold text-white shadow-xs hover:bg-white/20"
      >
        Refresh
      </button>
      <button
        type="button"
        @click="openPath(viewOnLichessUrl)"
        class="inline-flex items-center rounded-md bg-white/10 px-3 py-2 text-sm font-semibold text-white shadow-xs hover:bg-white/20"
      >
        View on Lichess
      </button>
      <button
        type="button"
        @click="openPath(`${settings.lichessUrl}/broadcast/new`)"
        class="inline-flex items-center rounded-md bg-indigo-500 px-3 py-2 text-sm font-semibold text-white shadow-xs hover:bg-indigo-400 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-500"
      >
        &plus; New Broadcast
      </button>
    </div>
  </div>

  <Pagination :broadcasts="broadcasts" />

  <div v-if="pageHasBroadcasts" class="overflow-y-auto">
    <div role="list" class="divide-y divide-white/5">
      <BroadcastSummary v-for="broadcast in broadcasts?.currentPageResults" :broadcast="broadcast" />
    </div>
  </div>

  <div v-if="!pageHasBroadcasts && !isLoading" class="text-center mt-12">
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      stroke-width="1.5"
      stroke="currentColor"
      class="mx-auto h-12 w-12 text-gray-400"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M13.5 16.875h3.375m0 0h3.375m-3.375 0V13.5m0 3.375v3.375M6 10.5h2.25a2.25 2.25 0 002.25-2.25V6a2.25 2.25 0 00-2.25-2.25H6A2.25 2.25 0 003.75 6v2.25A2.25 2.25 0 006 10.5zm0 9.75h2.25A2.25 2.25 0 0010.5 18v-2.25a2.25 2.25 0 00-2.25-2.25H6a2.25 2.25 0 00-2.25 2.25V18A2.25 2.25 0 006 20.25zm9.75-9.75H18a2.25 2.25 0 002.25-2.25V6A2.25 2.25 0 0018 3.75h-2.25A2.25 2.25 0 0013.5 6v2.25a2.25 2.25 0 002.25 2.25z"
      />
    </svg>

    <h3 class="mt-2 text-sm font-semibold text-gray-200">No broadcasts</h3>
    <p class="mt-1 text-sm text-gray-300">
      No broadcasts found for <span>{{ username }}</span>
    </p>

    <template v-if="user.is(username)">
      <p class="mt-1 text-sm text-gray-300">Get started by creating a new broadcast.</p>
      <div class="mt-6">
        <button
          type="button"
          @click="openPath(`${settings.lichessUrl}/broadcast/new`)"
          class="inline-flex items-center rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-xs hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
        >
          <svg class="-ml-0.5 mr-1.5 h-5 w-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
            <path
              d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z"
            />
          </svg>
          Create a new broadcast
        </button>
      </div>
    </template>
  </div>
</template>
