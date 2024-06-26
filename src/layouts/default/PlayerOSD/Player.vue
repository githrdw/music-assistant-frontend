<template>
  <div
    :class="`${useFloatingPlayer ? 'mediacontrols-bg-2' : 'mediacontrols-bg-1'}`"
    :style="`background: ${backgroundColor};`"
  ></div>
  <PlayerTimeline
    v-if="getBreakpointValue('bp3')"
    v-breakpoint="{ breakpoint: 'bp3', condition: 'lt' }"
    :color="
      $vuetify.theme.current.dark
        ? coverImageColorPalette.lightColor || '#fff'
        : coverImageColorPalette.darkColor || '#000'
    "
    :is-progress-bar="true"
  />

  <div
    class="mediacontrols"
    :style="`padding: ${
      getBreakpointValue({ breakpoint: 'phone' }) ? 3 : 10
    }px ${getBreakpointValue({ breakpoint: 'phone' }) ? 10 : 10}px;`"
  >
    <div :class="`mediacontrols-left-${getBreakpointValue('bp3') ? '1' : '2'}`">
      <PlayerTrackDetails
        :show-quality-details-btn="getBreakpointValue('bp8') ? true : false"
        :show-only-artist="getBreakpointValue('bp7') ? false : true"
        :color-palette="coverImageColorPalette"
        :primary-color="$vuetify.theme.current.dark ? '#fff' : '#000'"
      />
    </div>
    <div
      :class="`mediacontrols-bottom-center-${
        getBreakpointValue('bp3') ? '1' : '2'
      }`"
    >
      <div style="width: 100%">
        <!-- player control buttons -->
        <PlayerControls
          :visible-components="{
            repeat: { isVisible: getBreakpointValue('bp3') },
            shuffle: { isVisible: getBreakpointValue('bp3') },
            play: {
              isVisible: true,
              icon: {
                staticWidth: '50px',
                staticHeight: '50px',
              },
            },
            previous: { isVisible: getBreakpointValue('bp3') },
            next: { isVisible: getBreakpointValue('bp3') },
          }"
        />
        <!-- progress bar -->
        <PlayerTimeline
          v-breakpoint="{ breakpoint: 'mobile', condition: 'gt' }"
          :color="
            $vuetify.theme.current.dark
              ? coverImageColorPalette.lightColor || '#fff'
              : coverImageColorPalette.darkColor || '#000'
          "
          :is-progress-bar="false"
        />
      </div>
    </div>
    <div class="mediacontrols-bottom-right">
      <div>
        <!-- player mobile extended control buttons -->
        <PlayerExtendedControls
          :queue="{
            isVisible: getBreakpointValue('bp3'),
            color: $vuetify.theme.current.dark ? '#fff' : '#000',
          }"
          :player="{
            isVisible: true,
            color: $vuetify.theme.current.dark ? '#fff' : '#000',
          }"
          :volume="{
            isVisible: true,
            color: $vuetify.theme.current.dark ? '#fff' : '#000',
          }"
        />
        <!-- player mobile control buttons -->

        <PlayerControls
          style="padding-right: 5px"
          :visible-components="{
            repeat: { isVisible: false },
            shuffle: { isVisible: false },
            play: {
              isVisible: getBreakpointValue({
                breakpoint: 'bp3',
                condition: 'lt',
              }),
              withCircle: false,
              icon: {
                staticWidth: '48px',
                staticHeight: '48px',
                color: $vuetify.theme.current.dark ? '#fff' : '#000',
              },
            },
            previous: { isVisible: false },
            next: { isVisible: false },
          }"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';
//@ts-ignore

import { ImageType } from '@/plugins/api/interfaces';
import { store } from '@/plugins/store';
import { getImageThumbForItem } from '@/components/MediaItemThumb.vue';
import PlayerTimeline from './PlayerTimeline.vue';
import PlayerControls from './PlayerControls.vue';
import PlayerTrackDetails from './PlayerTrackDetails.vue';
import PlayerExtendedControls from './PlayerExtendedControls.vue';
import { getBreakpointValue } from '@/plugins/breakpoint';
import vuetify from '@/plugins/vuetify';
import { ImageColorPalette, getColorPalette } from '@/helpers/utils';
import {
  imgCoverDark,
  imgCoverLight,
} from '@/components/QualityDetailsBtn.vue';
import { useTheme } from 'vuetify/lib/framework.mjs';

interface Props {
  useFloatingPlayer: boolean;
}
defineProps<Props>();

// global refs
const theme = useTheme();

// local refs
const coverImageColorPalette = ref<ImageColorPalette>({
  '0': '',
  '1': '',
  '2': '',
  '3': '',
  '4': '',
  '5': '',
  lightColor: '',
  darkColor: '',
});

// utility feature to extract the dominant colors from the cover image
// we use this color palette to colorize the playerbar/OSD
const img = new Image();
img.src = vuetify.theme.current.value.dark ? imgCoverDark : imgCoverLight;
img.crossOrigin = 'Anonymous';
img.addEventListener('load', function () {
  coverImageColorPalette.value = getColorPalette(img);
});

const backgroundColor = computed(() => {
  if (vuetify.theme.current.value.dark) {
    if (coverImageColorPalette.value && coverImageColorPalette.value.darkColor)
      return coverImageColorPalette.value.darkColor + '26';
    return 'CCCCCC26';
  }
  if (coverImageColorPalette.value && coverImageColorPalette.value.lightColor)
    return coverImageColorPalette.value.lightColor + '26';
  return 'CCCCCC26';
});

// watchers
watch(
  () => store.curQueueItem?.queue_item_id,
  () => {
    // load cover image for the (new) QueueItem
    // make sure that the image selection is exactly the same as on the player OSD thumb
    if (store.curQueueItem?.media_item) {
      img.src =
        getImageThumbForItem(store.curQueueItem.media_item, ImageType.THUMB) ||
        '';
    } else if (store.curQueueItem) {
      img.src = getImageThumbForItem(store.curQueueItem, ImageType.THUMB) || '';
    } else {
      img.src = '';
    }
  },
);
</script>

<style scoped>
.mediadetails-streamdetails .icon {
  opacity: 100;
}

.mediacontrols {
  display: table;
  width: 100%;
}

.mediacontrols-bg-1 {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0px;
  top: 0px;
}

.mediacontrols-bg-2 {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0px;
  top: 0px;
  border-radius: 10px;
}

.mediacontrols-top-right {
  display: table-row;
}

.mediacontrols-left-1 {
  display: table-cell;
  vertical-align: middle;
  width: 25%;
}

.mediacontrols-left-2 {
  display: table-cell;
  vertical-align: middle;
}

.mediacontrols-left-1 > div {
  padding: 0px !important;
}

.mediacontrols-left-2 > div {
  padding: 0px !important;
}

.mediacontrols-bottom-center-1 {
  display: table-cell;
  text-align: center;
  width: 40%;
  vertical-align: middle;
}

.mediacontrols-bottom-center-2 {
  display: none;
  width: 25%;
}

.mediacontrols-bottom-right {
  display: table-cell;
  text-align: right;
  vertical-align: middle;
}

.mediacontrols-bottom-right > div {
  display: inline-flex;
  align-items: center;
}
</style>
