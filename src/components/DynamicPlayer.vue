<template>
  <div class="dynamic-player">
    <swiper-container
      ref="swiperEl"
      loop
      noSwiping
      init="false"
      navigation="true"
      pagination="true"
      @init="onInit"
      @slidechange="onSlideChange"
    >
      <div
        slot="container-start"
        class="
          relative
          z-10
          flex
          items-center
          justify-between
          p-5
          text-white text-sm
          font-semibold
        "
      >
        <div>To: Recipient</div>
        <div v-show="showOverlay" class="overlay">
          <div class="tooltip">Please unmute before playing.</div>
        </div>
        <button
          @click="toggleMute"
          class="flex items-center justify-center z-10"
        >
          <img
            :src="isMuted ? volumeOffImage : volumeOnImage"
            class="w-6 h-6"
            alt="Mute/Unmute Button"
          />
        </button>
      </div>
      <swiper-slide
        v-for="(slide, index) in slides"
        :key="index"
        class="-mt-16"
      >
        <div :id="'video-' + index">
          <button
            @click="togglePlay"
            :class="{ 'hide-unless-hovered': hasStartedPlaying }"
            class="
              absolute
              top-1/2
              right-0
              left-0
              flex
              w-16
              h-16
              items-center
              justify-center
              m-auto
              bg-white
              rounded-full
              -translate-y-1/2
            "
          >
            <img
              :src="isPlaying ? pauseImage : playImage"
              class="w-8 h-8"
              :class="{ 'ml-1.5': !isPlaying }"
              alt="Play/Pause Button"
            />
          </button>
        </div>
      </swiper-slide>
    </swiper-container>
  </div>
</template>

<script setup>
import { ref, markRaw, onMounted, onUnmounted, onBeforeUpdate } from 'vue';
import { register } from 'swiper/element/bundle';
import Player from '@vimeo/player';
register();

// Mobile unmute helper
const showOverlay = ref(false);

// Ref for images
const playImage = ref('play.svg');
const pauseImage = ref('pause.svg');
const volumeOnImage = ref('volume-on.svg');
const volumeOffImage = ref('volume-off.svg');

// Refs for swiper element
const swiperEl = ref(null);
const videoRefs = ref([]);
const isPlaying = ref(false);
const isMuted = ref(true);
const hasStartedPlaying = ref(false);
const slides = [
  {
    vimeoId: '823050002',
  },
  {
    vimeoId: '4521583',
  },
  {
    vimeoId: '215880646',
  },
];

onMounted(() => {
  // Show tooltip helper to unmute on mobile
  showOverlay.value = window.innerWidth < 768 && isMuted.value;
  window.addEventListener('resize', () => {
    showOverlay.value = window.innerWidth < 768 && isMuted.value;
  });

  // Swiper parameters
  const swiperParams = {
    autoplay: false,
    loop: true,
    injectStyles: [
      `
        :root {--swiper-theme-color: #fff;}
        swiper-container {height:100%;background-color:#202124;}
        swiper-slide {display:flex;align-items:center;justify-content:center;color:#fff;}
      `,
    ],
  };

  // Assign all parameters to Swiper element and initialize it
  Object.assign(swiperEl.value, swiperParams);
  swiperEl.value.initialize();

  // Initialize Vimeo players
  slides.forEach((slide, index) => {
    const player = new Player(`video-${index}`, {
      id: slide.vimeoId,
      width: 600,
      height: 400,
      autoplay: false,
      muted: true,
    });
    videoRefs.value.push(markRaw(player));

    player.on('ended', () => {
      if (isPlaying.value) {
        isPlaying.value = false;
        const nextIndex = (index + 1) % slides.length;
        const nextVideo = videoRefs.value[nextIndex];
        if (nextVideo) {
          nextVideo.setMuted(isMuted.value);
          nextVideo.play();
          isPlaying.value = true;
          swiperEl.value.swiper.slideNext();
        } else {
          swiperEl.value.swiper.slideTo(0);
        }
      }
    });
  });
});

onUnmounted(() => {
  // Remove event listener when component is unmounted to avoid memory leaks
  window.removeEventListener('resize', () => {
    showOverlay.value = window.innerWidth < 768 && isMuted.value;
  });
});

// Methods
const onInit = (e) => {
  console.log('swiper initialized');
};

const playVideo = (video) => {
  if (video) {
    video.setVolume(isMuted.value ? 0 : 1);
    video.play().catch((error) => {
      if (
        error.name === 'NotAllowedError' ||
        error.name === 'NotSupportedError'
      ) {
        //video.setVolume(0);
        video.play();
      }
    });
  }
};

const togglePlay = () => {
  if (swiperEl.value && swiperEl.value.swiper) {
    const currentVideo = videoRefs.value[swiperEl.value.swiper.realIndex];
    if (currentVideo) {
      if (isPlaying.value) {
        currentVideo.pause();
        isPlaying.value = false;
      } else {
        playVideo(currentVideo);
        isPlaying.value = true;
        hasStartedPlaying.value = true;
      }
    }
  }
};

const toggleMute = () => {
  if (swiperEl.value && swiperEl.value.swiper) {
    const currentVideo = videoRefs.value[swiperEl.value.swiper.realIndex];
    if (currentVideo) {
      if (isMuted.value) {
        currentVideo.setVolume(1); // TODO: for some reason, isMuted wasn't working with the vimeo player api
        isMuted.value = false;
      } else {
        currentVideo.setVolume(0); // TODO: for some reason, isMuted wasn't working with the vimeo player api
        isMuted.value = true;
      }
      showOverlay.value = window.innerWidth < 768 && isMuted.value;
    }
  }
};

const onSlideChange = (e) => {
  console.log('slide changed');
  const currentVideo = videoRefs.value[e.detail[0].realIndex];
  playVideo(currentVideo);
  hasStartedPlaying.value = true;
  const currentIndex = e.detail[0].realIndex;
  currentVideo.on('ended', () => {
    const nextIndex = currentIndex + 1;
    const nextVideo = videoRefs.value[nextIndex];
    if (nextVideo) {
      currentVideo.setVolume(0); // TODO: for some reason, isMuted wasn't working with the vimeo player api
      playVideo(nextVideo);
      swiperEl.value.swiper.slideNext();
    } else {
      swiperEl.value.swiper.slideTo(0);
    }
  });
};
</script>

<style scoped>
body {
  margin: 0;
}
@media (max-width: 767px) {
  swiper-slide > div {
    width: 100%;
    overflow: hidden;
  }
}
iframe {
  display: none !important;
}
.dynamic-player {
  height: 100vh;
}
.overlay {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9;
}
.tooltip {
  background-color: white;
  color: black;
  padding: 15px;
  border-radius: 5px;
  position: absolute;
  top: 60px;
  right: 10px;
}
.tooltip::after {
  content: '';
  position: absolute;
  bottom: 100%;
  right: 16px;
  margin-left: -8px;
  border-width: 8px;
  border-style: solid;
  border-color: transparent transparent white transparent;
}
.hide-unless-hovered {
  opacity: 0;
}
swiper-slide > div:hover .hide-unless-hovered {
  opacity: 1;
}
</style>

<style>
@media (max-width: 767px) {
  swiper-slide iframe {
    width: 100%;
  }
}
</style>
