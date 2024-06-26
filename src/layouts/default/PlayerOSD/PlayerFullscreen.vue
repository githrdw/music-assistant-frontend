<template>
  <v-dialog
    v-model="store.showFullscreenPlayer"
    fullscreen
    :scrim="false"
    style="overflow: hidden"
    transition="dialog-bottom-transition"
  >
    <v-card :color="darkenBrightColors(coverImageColorCode, 77, 40)">
      <v-toolbar class="v-toolbar-default" color="transparent">
        <template #prepend>
          <Button icon @click="store.showFullscreenPlayer = false">
            <v-icon icon="mdi-chevron-down" />
          </Button>
        </template>
      </v-toolbar>

      <Container class="fullscreen-container">
        <div class="fullscreen-media-space"></div>
        <div class="fullscreen-row-centered">
          <MediaItemThumb
            v-if="
              store.curQueueItem &&
              getBreakpointValue({ breakpoint: 'mobile' }) &&
              getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' }) &&
              false
            "
            :item="store.curQueueItem?.media_item || store.curQueueItem"
            :size="
              getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })
                ? 512
                : 1024
            "
            style="
              height: min(calc(100vw - 40px), calc(100vh - 340px));
              width: min(calc(100vw - 40px), calc(100vh - 340px));
            "
            :thumbnail="false"
          />
          <MediaItemThumb
            v-else-if="store.curQueueItem"
            :item="store.curQueueItem.media_item || store.curQueueItem"
            :size="
              getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })
                ? 512
                : 1024
            "
            style="
              height: min(calc(100vw - 40px), calc(100vh - 340px));
              width: min(calc(100vw - 40px), calc(100vh - 340px));
            "
            :thumbnail="false"
          />
          <v-img
            v-else
            :src="defaultImage"
            style="
              height: min(calc(100vw - 40px), calc(100vh - 340px));
              width: min(calc(100vw - 40px), calc(100vh - 340px));
            "
          />
        </div>
        <div class="fullscreen-row">
          <div
            v-if="store.curQueueItem && store.curQueueItem.media_item"
            class="fullscreen-track-info"
          >
            <!-- title -->
            <!-- radio station name -->
            <h2
              v-if="
                store.curQueueItem?.media_item &&
                store.curQueueItem.media_item?.media_type == MediaType.RADIO
              "
              style="cursor: pointer; width: fit-content; display: inline"
              class="title line-clamp-1"
              @click="
                store.curQueueItem?.media_item
                  ? radioNameClick(store.curQueueItem.media_item as Radio)
                  : ''
              "
            >
              {{ store.curQueueItem.media_item.name }}
            </h2>
            <!-- track -->
            <h2
              v-else
              style="cursor: pointer; width: fit-content; display: inline"
              class="title line-clamp-1"
              @click="
                store.curQueueItem?.media_item
                  ? trackClick(store.curQueueItem.media_item as Track)
                  : ''
              "
            >
              <!-- name + version (if present) -->
              {{
                `${store.curQueueItem.media_item.name} ${
                  'version' in store.curQueueItem.media_item &&
                  store.curQueueItem.media_item.version
                    ? '(' + store.curQueueItem.media_item.version + ')'
                    : ''
                }`
              }}
            </h2>

            <!-- subtitle -->

            <!-- radio station stream title -->
            <div
              v-if="
                store.curQueueItem.media_item?.media_type == MediaType.RADIO &&
                store.curQueueItem?.streamdetails?.stream_title
              "
            >
              <!-- radio live metadata -->
              <h4
                v-if="
                  store.curQueueItem.streamdetails.stream_title.includes(' - ')
                "
                class="fullscreen-track-info-subtitle"
                style="cursor: pointer"
                @click="
                  radioTitleClick(store.curQueueItem.streamdetails.stream_title)
                "
              >
                {{ store.curQueueItem.streamdetails.stream_title }}
              </h4>
              <h4 v-else class="fullscreen-track-info-subtitle">
                {{ store.curQueueItem.streamdetails.stream_title }}
              </h4>
            </div>
            <!-- track: artists(s) + album -->
            <div
              v-else-if="
                store.curQueueItem.media_item?.media_type == MediaType.TRACK &&
                'artists' in store.curQueueItem.media_item &&
                store.curQueueItem.media_item.artists
              "
              style="cursor: pointer"
              class="line-clamp-1"
            >
              <h4>
                <!-- album -->
                <span
                  v-if="
                    'album' in store.curQueueItem.media_item &&
                    store.curQueueItem.media_item.album
                  "
                  class="fullscreen-track-info-subtitle"
                  @click="albumClick(store.curQueueItem.media_item.album)"
                >
                  {{ store.curQueueItem.media_item.album.name }}
                </span>
                <!-- artist(s) -->
                <span
                  class="fullscreen-track-info"
                  @click="
                    artistClick(
                      (store.curQueueItem.media_item as Track).artists[0],
                    )
                  "
                >
                  <br /><br />{{
                    (store.curQueueItem.media_item as Track).artists[0].name
                  }}
                </span>
              </h4>
            </div>
            <!-- other description -->
            <h4
              v-else-if="
                store.curQueueItem &&
                store.curQueueItem.media_item?.metadata.description
              "
              class="fullscreen-track-info-subtitle"
            >
              {{ store.curQueueItem.media_item.metadata.description }}
            </h4>
            <!-- queue empty message -->
            <h4
              v-else-if="
                store.activePlayerQueue && store.activePlayerQueue.items == 0
              "
              class="fullscreen-track-info-subtitle"
            >
              {{ $t('queue_empty') }}
            </h4>
            <!-- 3rd party source active -->
            <h4
              v-else-if="
                store.selectedPlayer?.active_source !=
                store.selectedPlayer?.player_id
              "
              class="fullscreen-track-info-subtitle"
            >
              {{
                $t('external_source_active', [
                  store.selectedPlayer?.active_source,
                ])
              }}
            </h4>
          </div>
        </div>
        <div class="fullscreen-row" style="margin: 0 10px">
          <PlayerTimeline
            :is-progress-bar="false"
            :color="
              $vuetify.theme.current.dark
                ? props.colorPalette.lightColor
                : props.colorPalette.darkColor
            "
          />
        </div>
        <div v-if="false">
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette[0]}`,
            }"
          ></div>
          <a style="color: #000"> {{ props.colorPalette[0] }}</a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette[1]}`,
            }"
          ></div>
          <a style="color: #000"> {{ props.colorPalette[1] }}</a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette[2]}`,
            }"
          ></div>
          <a style="color: #000"> {{ props.colorPalette[2] }}</a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette[3]}`,
            }"
          ></div>
          <a style="color: #000"> {{ props.colorPalette[3] }}</a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette[4]}`,
            }"
          ></div>
          <a style="color: #000"> {{ props.colorPalette[4] }}</a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette.darkColor}`,
            }"
          ></div>
          <a style="color: #000"> DarkColor </a>
          <div
            style="height: 50px; width: 50px"
            :style="{
              background: `${props.colorPalette.lightColor}`,
            }"
          ></div>
          <a style="color: #000"> LightColor </a>
        </div>
        <div class="fullscreen-row">
          <PlayerExtendedControls
            v-if="getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })"
            :queue="{ isVisible: false }"
            :player="{ isVisible: false }"
            :volume="{
              isVisible: true,
              volumeSize: '100%',
              responsiveVolumeSize: false,
            }"
          />
        </div>
        <div
          v-if="getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })"
          class="fullscreen-media-controls"
        >
          <ShuffleBtn class="media-controls-item" max-height="24px" />

          <PreviousBtn class="media-controls-item" max-height="35px" />

          <PlayBtn class="media-controls-item" max-height="80px" />

          <NextBtn class="media-controls-item" max-height="35px" />

          <RepeatBtn class="media-controls-item" max-height="24px" />
        </div>
        <div
          v-else
          class="fullscreen-media-controls"
          :style="{
            paddingTop: '1vh',
            flex: getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })
              ? '1 1 auto'
              : '0 1 auto',
          }"
        >
          <div
            class="media-controls-item"
            style="display: grid; align-content: center"
          >
            <QualityDetailsBtn />
          </div>
          <div class="media-controls-item fullscreen-row-centered">
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
            </div>
          </div>
          <div class="fullscreen-mediacontrols-right media-controls-item">
            <div>
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
                  },
                  previous: { isVisible: false },
                  next: { isVisible: false },
                }"
              />
              <!-- player mobile extended control buttons -->
              <PlayerExtendedControls
                :queue="{ isVisible: getBreakpointValue('bp3') }"
                :player="{ isVisible: true }"
                :volume="{ isVisible: getBreakpointValue('bp0') }"
              />
            </div>
          </div>
        </div>
        <div
          class="fullscreen-media-controls-bottom"
          if="store.activePlayerQueue"
        >
          <div
            v-if="getBreakpointValue({ breakpoint: 'bp3', condition: 'lt' })"
          >
            <Button
              @click="
                store.showFullscreenPlayer = false;
                store.showPlayersMenu = true;
              "
            >
              <v-badge
                v-if="curGroupPlayers && curGroupPlayers.length > 0"
                :content="store.selectedPlayer?.group_childs.length"
                :color="
                  $vuetify.theme.current.dark
                    ? props.colorPalette.lightColor
                    : props.colorPalette.darkColor
                "
              >
                <v-icon :size="30">mdi-speaker</v-icon>
              </v-badge>
              <v-icon v-else :size="30">mdi-speaker</v-icon>
              <div class="line-clamp-1">
                {{ store.activePlayerQueue?.display_name }}
              </div>
            </Button>
          </div>
          <div style="text-align: end">
            <PlayerExtendedControls
              :queue="{
                isVisible: getBreakpointValue({
                  breakpoint: 'bp3',
                  condition: 'lt',
                }),
              }"
              :player="{ isVisible: false }"
              :volume="{ isVisible: false }"
            />
          </div>
        </div>
      </Container>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

import PlayerExtendedControls from './PlayerExtendedControls.vue';
import MediaItemThumb from '@/components/MediaItemThumb.vue';
import api from '@/plugins/api';
import {
  Album,
  Artist,
  ItemMapping,
  MediaType,
  Radio,
  Track,
} from '@/plugins/api/interfaces';
import { store } from '@/plugins/store';
import PlayerTimeline from './PlayerTimeline.vue';
import Breakpoint, { getBreakpointValue } from '@/plugins/breakpoint';
import Button from '@/components/mods/Button.vue';
import vuetify from '@/plugins/vuetify';
import Container from '@/components/mods/Container.vue';
import PlayBtn from '@/layouts/default/PlayerOSD/PlayerControlBtn/PlayBtn.vue';
import NextBtn from '@/layouts/default/PlayerOSD/PlayerControlBtn/NextBtn.vue';
import PreviousBtn from '@/layouts/default/PlayerOSD/PlayerControlBtn/PreviousBtn.vue';
import ShuffleBtn from '@/layouts/default/PlayerOSD/PlayerControlBtn/ShuffleBtn.vue';
import RepeatBtn from '@/layouts/default/PlayerOSD/PlayerControlBtn/RepeatBtn.vue';
import {
  imgCoverDark,
  imgCoverLight,
} from '@/components/QualityDetailsBtn.vue';
import PlayerControls from './PlayerControls.vue';
import QualityDetailsBtn from '@/components/QualityDetailsBtn.vue';
import router from '@/plugins/router';
import { ImageColorPalette, darkenBrightColors } from '@/helpers/utils';

interface Props {
  colorPalette: ImageColorPalette;
}
const props = defineProps<Props>();

const coverImageColorCode = ref<string>('');
// Local refs
const fullTrackDetails = ref<Track>();

// Computed properties
const curGroupPlayers = computed(() => {
  if (store.selectedPlayer) {
    return store.selectedPlayer.group_childs;
  }
  return undefined;
});

const defaultImage = computed(() => {
  return vuetify.theme.current.value.dark ? imgCoverDark : imgCoverLight;
});

// methods
const trackClick = function (item: Track | ItemMapping) {
  router.push({
    name: 'track',
    params: { itemId: item.item_id, provider: item.provider },
  });
  store.showFullscreenPlayer = false;
};

const artistClick = function (item: Artist | ItemMapping) {
  // album entry clicked
  router.push({
    name: 'artist',
    params: {
      itemId: item.item_id,
      provider: item.provider,
    },
  });
  store.showFullscreenPlayer = false;
};

const albumClick = function (item: Album | ItemMapping) {
  // album entry clicked
  router.push({
    name: 'album',
    params: {
      itemId: item.item_id,
      provider: item.provider,
    },
  });
  store.showFullscreenPlayer = false;
};

const radioNameClick = function (item: Radio | ItemMapping) {
  // radio station name clicked
  router.push({
    name: 'radio',
    params: {
      itemId: item.item_id,
      provider: item.provider,
    },
  });
  store.showFullscreenPlayer = false;
};

const radioTitleClick = function (streamTitle: string) {
  // radio station title clicked
  store.globalSearchTerm = streamTitle;
  router.push({ name: 'search' });
  store.showFullscreenPlayer = false;
};

// watchers
watch(
  () => store.curQueueItem,
  async (result) => {
    if (
      result &&
      result.media_item &&
      result.media_item.media_type === MediaType.TRACK
    ) {
      fullTrackDetails.value = (await api.getItemByUri(
        result.media_item.uri,
      )) as Track;
    }
  },
);

watch(
  () => props.colorPalette,
  (result) => {
    if (!result.darkColor || !result.lightColor) {
      coverImageColorCode.value = vuetify.theme.current.value.dark
        ? '#000'
        : '#fff';
    } else {
      coverImageColorCode.value = vuetify.theme.current.value.dark
        ? result.darkColor
        : result.lightColor;
    }
  },
);
</script>

<style scoped>
.fullscreen-container {
  display: flex;
  flex-flow: column;
  height: 100%;
  padding-bottom: 5px;
}

.fullscreen-track-info {
  margin-top: 10px;
  margin-bottom: 10px;
  padding-top: 2vh;
  padding-bottom: 2vh;
  text-align: center;
}

.fullscreen-track-info-subtitle {
  opacity: 0.5;
}

.media-controls-item {
  margin: 0 10px;
  width: 100%;
  height: 100%;
}

@media (max-width: 768px) {
  .media-controls-item {
    margin: 0 5px;
    width: 100%;
    height: 100%;
  }
}

.fullscreen-row {
  display: grid;
  flex: 0 1 auto;
  align-content: center;
}

.fullscreen-row-centered {
  display: grid;
  flex: 0 1 auto;
  justify-content: center;
}

.fullscreen-media-controls {
  display: flex;
  flex: 1 1 auto;
  align-items: center;
}

.fullscreen-media-controls > button {
  flex: 1 1 auto;
}

.fullscreen-media-controls-bottom {
  display: flex;
  flex: 0 1 auto;
  justify-content: center;
  align-items: center;
}

.fullscreen-media-controls-bottom > div {
  flex-basis: 0;
  flex-grow: 1;
  max-width: 100%;
}

.fullscreen-media-controls > div {
  width: calc(100% / 3);
}

.fullscreen-mediacontrols-right {
  display: table-cell;
  text-align: right;
  vertical-align: middle;
}

.fullscreen-mediacontrols-right > div {
  display: inline-flex;
  align-items: center;
}
</style>
