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
      <div slot="container-start" class="p-5 text-white text-sm">
        To: Recipient
      </div>
      <swiper-slide v-for="(slide, index) in slides" :key="index">
        <div :id="'video-' + index"></div>
      </swiper-slide>
    </swiper-container>
    <button @click="playVideo" class="btn btn-primary btn-lg">Play</button>
  </div>
</template>

<script setup>
import { ref, markRaw, onMounted, onBeforeUpdate } from 'vue';
import { register } from 'swiper/element/bundle';
import Player from '@vimeo/player';
register();

// Ref for swiper element
const swiperEl = ref(null);
const videoRefs = ref([]);
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
  });
});

// Methods
const onInit = (e) => {
  console.log('swiper initialized');
};

const onSlideChange = (e) => {
  console.log('slide changed');
  const currentVideo = videoRefs.value[e.detail[0].realIndex];
  //currentVideo.setMuted(false);
  currentVideo.play();
  const currentIndex = e.detail[0].realIndex;
  currentVideo.on('ended', () => {
    const nextIndex = currentIndex + 1;
    const nextVideo = videoRefs.value[nextIndex];
    if (nextVideo) {
      currentVideo.pause();
      //nextVideo.setMuted(false);
      nextVideo.play();
      swiperEl.value.swiper.slideNext();
    } else {
      swiperEl.value.swiper.slideTo(0);
    }
  });
};

const playVideo = () => {
  if (videoRefs.value.length > 0) {
    const firstVideo = videoRefs.value[0];
    //firstVideo.setMuted(false);
    firstVideo.play();
    firstVideo.on('ended', () => {
      swiperEl.value.swiper.slideNext();
    });
  }
};
</script>

<style scoped>
body {
  margin: 0;
}
.dynamic-player {
  height: 100vh;
}
</style>
