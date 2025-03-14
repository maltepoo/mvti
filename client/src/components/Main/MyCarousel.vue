<template>
  <div>
    <transition name="carousel-fade" mode="out-in">
        <div class="load-screen" v-if="loading" key="1">
        <p>Loading MCU Timeline</p>
        </div>
        <div class="load-screen" v-else-if="!loading &amp;&amp; error" key="2">
        <p>Timeline could not assemble</p>
        </div>
        <main class="main-screen" v-else key="3" :class="{ 'show-modal' : showModal }">
        <carousel v-if="movies.length" :class="scrollDirection" :options="carouselOptions" :movies="movies" :current="current" @scroll-direction="handleScrollDirection" @set-current="handleCurrentSlide" @trigger-modal="handleModal"></carousel>
        <transition name="modal">
            <modal v-if="showModal" :id="movies[current].id" @trigger-modal="handleModal"></modal>
        </transition>
        </main>
    </transition>

    <div class="carousel">
    <div class="image__viewport">
      <transition name="fade" mode="out-in">
        <carousel-image class="carousel__image" v-if="index === current"></carousel-image>
      </transition>
    </div>
    <embla ref="embla" :options="options">
      <carousel-item v-for="movie in movies" :title="movie.title" :key="movie.id" v-on="$listeners"></carousel-item>
    </embla>
    <transition name="fade">
      <button class="btn btn--prev" v-if="canScrollPrev()" @click="scrollPrev">
        <svg class="icon--control" viewBox="0 0 24 24">
          <path d="M20,11V13H8L13.5,18.5L12.08,19.92L4.16,12L12.08,4.08L13.5,5.5L8,11H20Z"></path>
        </svg>
      </button>
    </transition>
    <transition name="fade">
      <button class="btn btn--next" v-if="canScrollNext()" @click="scrollNext">
        <svg class="icon--control" viewBox="0 0 24 24">
          <path d="M4,11V13H16L10.5,18.5L11.92,19.92L19.84,12L11.92,4.08L10.5,5.5L16,11H4Z"></path>
        </svg>
      </button>
    </transition>
    <div class="timeline">
      <!-- <div class="dot" v-for="(movie, index) in movies" :class="{ current : index === current }" @click="scrollTo(index)">
        <div class="dot__image-wrapper">
          <carousel-image :backdrop="movie.poster_path" size="154" :key="movie.id"></carousel-image>
        </div>
      </div> -->
    </div>
  </div>
  </div>
</template>

<script>
const userKey = "38a848f0eb613aa2c80ddf98f614b96a"

export default {
  name: 'MyCarosel',
  data: function () {
    return {
        carouselOptions: { speed: 25 },
        current: 0,
        movies: [],
        scrollDirection: null,
        showModal: false,
        loading: true,
        error: false
    }
  
  },
  methods: {
    handleScrollDirection(value) {
      this.scrollDirection = value;
    },
    
    handleCurrentSlide(value) {
      this.current = value;
    },
    
    handleModal() {
      this.showModal = !this.showModal;
    },
    
    fetchMovieList() { 
      this.loading = true;
      fetch(`https://api.themoviedb.org/3/list/142235?api_key=${userKey}&language=en-US`)
        .then(res => res.json())
        .then(data => this.movies = data.items)
        .then(this.loading = false)
        .catch(() => this.error);
    },
    setCurrent() {
      this.$emit('set-current', this.$refs.embla.api().selectedScrollSnap());
    },
    
    setScrollDirection() {
      if (this.$refs.embla.api().selectedScrollSnap() > this.$refs.embla.api().previousScrollSnap()) {                
       return this.$emit('scroll-direction', 'scroll-next');
      }
      
      this.$emit('scroll-direction', 'scroll-prev');
    },
    
    scrollTo(target) {
      this.$refs.embla.api().scrollTo(target);
    },
    
    scrollPrev() {
      this.$refs.embla.api().scrollPrev();
    },
    
    scrollNext() {
      this.$refs.embla.api().scrollNext();
    },
    
    canScrollPrev() {
      return this.$refs.embla ? this.$refs.embla.api().canScrollPrev() : false;
    },
    
    canScrollNext() {
      return this.$refs.embla ? this.$refs.embla.api().canScrollNext() : true;
    },
    
    handleUpdate() {
      this.setCurrent();
      this.setScrollDirection();
    },
  },
  
  mounted() {
    this.fetchMovieList();
    this.$refs.embla.api().on('select', this.handleUpdate);
  }

}
</script>

<style>
:root {
  --font-primary: 'Oswald', sans-serif;
  --font-secondary: 'Playfair Display', serif;
  --ease-out: cubic-bezier(0.33, 1, 0.68, 1);
  --ease-in: cubic-bezier(0.5, 0, 0.75, 0);
  --duration: 500ms;
  --backdrop-offset: 12px;
  --btn-size: calc(0.75rem + 5vmin);
  --btn-cta-font-size: min(max(14px, 3vw), 30px);
  --slide-title-font-size: min(max(16px, 4vw), 60px);
}

* {
  box-sizing: border-box;
}

html, body {
  height: 100%;
  height: -webkit-fill-available;
}

body {
  background-color: black;
  font-family: var(--font-primary);
  overflow: hidden;
}

.load-screen {
  position: absolute;
  top: 50%;
  left: 50%;
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
          align-items: center;
  -webkit-box-pack: center;
          justify-content: center;
  color: rgba(255, 255, 255, 0.4);
  text-transform: uppercase;
  text-align: center;
  letter-spacing: 0.025rem;
  font-size: min(max(16px, 4vw), 60px);
  -webkit-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
}
.load-screen p {
  opacity: 0;
  -webkit-animation: fadein var(--duration) var(--duration) var(--ease-out) forwards;
          animation: fadein var(--duration) var(--duration) var(--ease-out) forwards;
}
@-webkit-keyframes fadein {
  to {
    opacity: 1;
  }
}
@keyframes fadein {
  to {
    opacity: 1;
  }
}
.carousel__viewport {
  --mask-size: 6vw;
  position: relative;
  width: 100%;
  height: 100vh;
  z-index: 2;
  overflow: hidden;
  -webkit-mask-image: -webkit-gradient(linear, left top, right top, from(transparent), color-stop(transparent), color-stop(black), color-stop(black), color-stop(transparent), to(transparent));
  -webkit-mask-image: linear-gradient(to right, transparent, transparent var(--mask-size), black calc(var(--mask-size) * 2), black calc(100% - calc(var(--mask-size) * 2)), transparent calc(100% - var(--mask-size)), transparent);
          mask-image: -webkit-gradient(linear, left top, right top, from(transparent), color-stop(transparent), color-stop(black), color-stop(black), color-stop(transparent), to(transparent));
          mask-image: linear-gradient(to right, transparent, transparent var(--mask-size), black calc(var(--mask-size) * 2), black calc(100% - calc(var(--mask-size) * 2)), transparent calc(100% - var(--mask-size)), transparent);
}

.carousel__viewport.is-draggable {
  cursor: move;
  cursor: -webkit-grab;
  cursor: grab;
}

.carousel__viewport.is-dragging {
  cursor: -webkit-grabbing;
  cursor: grabbing;
}

.carousel__container {
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
          align-items: center;
  height: 100%;
  height: -webkit-fill-available;
}

.carousel__slide {
  position: relative;
  display: -webkit-box;
  display: flex;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
          flex-direction: column;
  -webkit-box-align: center;
          align-items: center;
  -webkit-box-pack: center;
          justify-content: center;
  -webkit-box-flex: 0;
          flex: 0 0 auto;
  padding: 0 8vw;
  text-align: center;
  list-style: none;
  color: white;
  -webkit-transform: translateY(var(--btn-cta-font-size)) scale(1.0001);
          transform: translateY(var(--btn-cta-font-size)) scale(1.0001);
  -webkit-transition: -webkit-transform calc(var(--duration) / 2) var(--ease-out);
  transition: -webkit-transform calc(var(--duration) / 2) var(--ease-out);
  transition: transform calc(var(--duration) / 2) var(--ease-out);
  transition: transform calc(var(--duration) / 2) var(--ease-out), -webkit-transform calc(var(--duration) / 2) var(--ease-out);
}
.carousel__slide.is-selected {
  -webkit-transform: translateY(0) scale(1.0001);
          transform: translateY(0) scale(1.0001);
  -webkit-transition-duration: calc(var(--duration) / 2);
          transition-duration: calc(var(--duration) / 2);
  -webkit-transition-delay: calc(var(--duration) * 1.25);
          transition-delay: calc(var(--duration) * 1.25);
}

.slide__title {
  font-size: var(--slide-title-font-size);
  line-height: 1.2;
  opacity: 0.6;
  text-transform: uppercase;
  -webkit-transform: scale(0.98);
          transform: scale(0.98);
  -webkit-transition: opacity calc(var(--duration) * 1.5) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity calc(var(--duration) * 1.5) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity calc(var(--duration) * 1.5) var(--ease-out), transform var(--duration) var(--ease-out);
  transition: opacity calc(var(--duration) * 1.5) var(--ease-out), transform var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
}
.is-selected .slide__title {
  opacity: 1;
  -webkit-transform: scale(1.18);
          transform: scale(1.18);
}

.image__viewport {
  --mask-size: 40px;
  position: absolute;
  top: 0;
  left: 0;
  overflow: hidden;
  width: 100%;
  height: 100%;
  z-index: 1;
  -webkit-transform: scale(1.02);
          transform: scale(1.02);
  -webkit-mask-image: -webkit-gradient(linear, left top, left bottom, from(transparent), color-stop(transparent), color-stop(black), color-stop(black), color-stop(transparent), to(transparent));
  -webkit-mask-image: linear-gradient(to bottom, transparent, transparent var(--mask-size), black calc(var(--mask-size) * 4), black calc(100% - calc(var(--mask-size) * 4)), transparent calc(100% - var(--mask-size)), transparent);
          mask-image: -webkit-gradient(linear, left top, left bottom, from(transparent), color-stop(transparent), color-stop(black), color-stop(black), color-stop(transparent), to(transparent));
          mask-image: linear-gradient(to bottom, transparent, transparent var(--mask-size), black calc(var(--mask-size) * 4), black calc(100% - calc(var(--mask-size) * 4)), transparent calc(100% - var(--mask-size)), transparent);
}

.carousel__image {
  position: absolute;
  top: calc(var(--backdrop-offset) / -2);
  left: calc(var(--backdrop-offset) / -2);
  width: calc(100% + var(--backdrop-offset));
  height: calc(100% + var(--backdrop-offset));
  z-index: 1;
  opacity: 0.5;
  -o-object-fit: cover;
     object-fit: cover;
}

.btn {
  border: 0;
  padding: 0;
  outline: none;
  cursor: pointer;
  background-color: transparent;
  color: white;
  fill: currentcolor;
  z-index: 3;
  touch-action: manipulation;
}

.btn--cta {
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
          align-items: center;
  margin-top: 1rem;
  font-family: var(--font-secondary);
  font-size: var(--btn-cta-font-size);
  font-style: italic;
  letter-spacing: 0.05rem;
  opacity: 0;
  -webkit-transform: translateY(12px);
          transform: translateY(12px);
  pointer-events: none;
  visibility: hidden;
  -webkit-transition: opacity calc(var(--duration) / 4) var(--ease-out), visibility 0s var(--duration), -webkit-transform 0s var(--duration) var(--ease-out);
  transition: opacity calc(var(--duration) / 4) var(--ease-out), visibility 0s var(--duration), -webkit-transform 0s var(--duration) var(--ease-out);
  transition: transform 0s var(--duration) var(--ease-out), opacity calc(var(--duration) / 4) var(--ease-out), visibility 0s var(--duration);
  transition: transform 0s var(--duration) var(--ease-out), opacity calc(var(--duration) / 4) var(--ease-out), visibility 0s var(--duration), -webkit-transform 0s var(--duration) var(--ease-out);
}
.btn--cta > * + * {
  margin-left: 0.25rem;
}
.btn--cta svg {
  margin-top: 0.125em;
  width: 1.25em;
  height: 1.25em;
  fill: currentcolor;
  -webkit-transition: fill calc(var(--duration) / 4) var(--ease-out);
  transition: fill calc(var(--duration) / 4) var(--ease-out);
}
.is-selected .btn--cta {
  opacity: 1;
  visibility: visible;
  pointer-events: initial;
  -webkit-transform: translateY(0);
          transform: translateY(0);
  -webkit-transition-duration: calc(var(--duration) / 1.5);
          transition-duration: calc(var(--duration) / 1.5);
  -webkit-transition-delay: calc(var(--duration) * 1.5);
          transition-delay: calc(var(--duration) * 1.5);
}

.btn--prev,
.btn--next {
  position: absolute;
  top: calc(50% - (var(--btn-size) / 2));
}

.btn--prev {
  left: 12px;
}

.btn--next {
  right: 12px;
}

.btn .icon--control {
  width: var(--btn-size);
  height: var(--btn-size);
}

.btn--close {
  position: absolute;
  right: -8px;
  bottom: calc(100% + 1rem);
  width: var(--btn-size);
  height: var(--btn-size);
  opacity: 0;
  -webkit-transition: opacity calc(var(--duration) / 4) var(--ease-out);
  transition: opacity calc(var(--duration) / 4) var(--ease-out);
}
.show-modal .btn--close {
  opacity: 1;
  -webkit-transition-duration: calc(var(--duration) / 2);
          transition-duration: calc(var(--duration) / 2);
}

.timeline {
  --margin: 10vw;
  display: -webkit-box;
  display: flex;
  -webkit-box-pack: justify;
          justify-content: space-between;
  position: absolute;
  bottom: 4rem;
  margin: 0 var(--margin);
  width: calc(100% - (var(--margin) * 2));
  z-index: 3;
}

.dot {
  --line-size: 6px;
  --line-height: 40px;
  -webkit-box-flex: 1;
          flex: 1 0 auto;
  display: -webkit-box;
  display: flex;
  -webkit-box-pack: justify;
          justify-content: space-between;
  position: relative;
  border-bottom: var(--line-size) solid white;
  height: var(--line-height);
  cursor: pointer;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
}
.dot::before {
  /* content: '';
  position: absolute;
  right: 1px;
  bottom: 4px;
  width: 1px;
  height: calc(var(--line-height) / 6);
  background-color: white; */
}
.dot::after {
  content: '';
  position: absolute;
  top: calc(100% + var(--line-size));
  width: 100%;
  height: var(--line-height);
}
.dot:hover ~ .dot {
  opacity: 0.25;
}
:not(:hover) .dot.current:not(:hover) ~ .dot {
  opacity: 0.25;
}

.dot__image-wrapper {
  --wrapper-size: 100px;
  position: absolute;
  bottom: 100%;
  right: calc((var(--wrapper-size) * -1) + (var(--line-size) / 2));
  width: var(--wrapper-size);
  height: auto;
  pointer-events: none;
  opacity: 0;
  -webkit-transform: translateX(-50%);
          transform: translateX(-50%);
  -webkit-transition: opacity calc(var(--duration) * 2) var(--ease-out);
  transition: opacity calc(var(--duration) * 2) var(--ease-out);
}
.dot__image-wrapper::after {
  content: '';
  position: absolute;
  top: 100%;
  right: 0;
  left: 0;
  margin: 0 auto;
  height: var(--line-height);
  width: var(--line-size);
  pointer-events: none;
  background-color: white;
  -webkit-transform-origin: 50% 100%;
          transform-origin: 50% 100%;
  opacity: 0;
}
.dot__image-wrapper img {
  width: 100%;
  opacity: 0;
  -webkit-transition: opacity var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out);
}
.dot:hover .dot__image-wrapper {
  opacity: 1;
  pointer-events: initial;
  -webkit-transition-duration: calc(var(--duration) / 4);
          transition-duration: calc(var(--duration) / 4);
  z-index: 1;
}
.dot:hover .dot__image-wrapper img {
  opacity: 1;
  -webkit-transition-duration: 0s;
          transition-duration: 0s;
}
.dot:hover .dot__image-wrapper:after {
  opacity: 1;
}

.modal {
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
          align-items: center;
  -webkit-box-pack: center;
          justify-content: center;
  position: fixed;
  top: 0;
  left: 0;
  padding: 4rem 4vw;
  width: 100%;
  height: 100%;
  z-index: 2;
}

.modal__overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.modal__wrapper {
  position: relative;
  width: 100vw;
  max-width: 1000px;
  margin: auto;
  background: black;
}
.modal__wrapper::after {
  content: "";
  display: block;
  padding-bottom: calc(100% / (16 / 9));
}
.modal__wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.fade-enter-active {
  -webkit-transition: opacity var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), transform var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
}

.fade-leave-active {
  -webkit-transition: opacity calc(var(--duration) / 2) var(--ease-in), -webkit-transform calc(var(--duration) / 2) var(--ease-in);
  transition: opacity calc(var(--duration) / 2) var(--ease-in), -webkit-transform calc(var(--duration) / 2) var(--ease-in);
  transition: opacity calc(var(--duration) / 2) var(--ease-in), transform calc(var(--duration) / 2) var(--ease-in);
  transition: opacity calc(var(--duration) / 2) var(--ease-in), transform calc(var(--duration) / 2) var(--ease-in), -webkit-transform calc(var(--duration) / 2) var(--ease-in);
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}

.scroll-prev .fade-enter {
  -webkit-transform: translateX(calc(var(--backdrop-offset) * -1));
          transform: translateX(calc(var(--backdrop-offset) * -1));
}
.scroll-next .fade-enter {
  -webkit-transform: translateX(var(--backdrop-offset));
          transform: translateX(var(--backdrop-offset));
}

.scroll-prev .fade-leave-to {
  -webkit-transform: translateX(var(--backdrop-offset));
          transform: translateX(var(--backdrop-offset));
}
.scroll-next .fade-leave-to {
  -webkit-transform: translateX(calc(var(--backdrop-offset) * -1));
          transform: translateX(calc(var(--backdrop-offset) * -1));
}

.carousel-fade-enter-active {
  -webkit-transition: opacity calc(var(--duration) * 4) var(--duration) var(--ease-out);
  transition: opacity calc(var(--duration) * 4) var(--duration) var(--ease-out);
}

.carousel-fade-leave-active {
  -webkit-transition: opacity var(--duration) calc(var(--duration) * 3) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) * 3) var(--ease-out);
}

.carousel-fade-enter,
.carousel-fade-leave-to {
  opacity: 0;
}

.modal-enter-active {
  -webkit-transition: opacity var(--duration) calc(var(--duration) / 3) var(--ease-out), -webkit-transform var(--duration) calc(var(--duration) / 3) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 3) var(--ease-out), -webkit-transform var(--duration) calc(var(--duration) / 3) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 3) var(--ease-out), transform var(--duration) calc(var(--duration) / 3) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 3) var(--ease-out), transform var(--duration) calc(var(--duration) / 3) var(--ease-out), -webkit-transform var(--duration) calc(var(--duration) / 3) var(--ease-out);
}
.modal-enter-active .btn--close {
  opacity: 0;
}

.modal-leave-active {
  -webkit-transition: opacity calc(var(--duration) / 2) var(--ease-out), -webkit-transform calc(var(--duration) / 2) var(--ease-out);
  transition: opacity calc(var(--duration) / 2) var(--ease-out), -webkit-transform calc(var(--duration) / 2) var(--ease-out);
  transition: opacity calc(var(--duration) / 2) var(--ease-out), transform calc(var(--duration) / 2) var(--ease-out);
  transition: opacity calc(var(--duration) / 2) var(--ease-out), transform calc(var(--duration) / 2) var(--ease-out), -webkit-transform calc(var(--duration) / 2) var(--ease-out);
}

.modal-enter, .modal-leave-to {
  opacity: 0;
  -webkit-transform: scale(1.04);
          transform: scale(1.04);
}

.carousel__viewport, .image__viewport, .timeline, .btn--prev, .btn--next {
  -webkit-transition: opacity var(--duration) calc(var(--duration) / 2) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 2) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 2) var(--ease-out), transform var(--duration) var(--ease-out);
  transition: opacity var(--duration) calc(var(--duration) / 2) var(--ease-out), transform var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out);
}
.show-modal .carousel__viewport,
.show-modal .image__viewport,
.show-modal .timeline,
.show-modal .btn--prev,
.show-modal .btn--next {
  -webkit-transition-delay: 0s;
          transition-delay: 0s;
  -webkit-transition-duration: var(--duration);
          transition-duration: var(--duration);
}

.show-modal .carousel__viewport {
  opacity: 0;
  -webkit-transform: scale(0.9);
          transform: scale(0.9);
}

.show-modal .image__viewport {
  opacity: 0.75;
  -webkit-transform: scale(1.0001);
          transform: scale(1.0001);
}

.show-modal .timeline,
.show-modal .btn--prev,
.show-modal .btn--next {
  opacity: 0;
  -webkit-transform: scale(0.96);
          transform: scale(0.96);
  pointer-events: none;
}

.image__viewport {
  -webkit-transition: opacity var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out), -webkit-filter var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out), -webkit-filter var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), transform var(--duration) var(--ease-out), filter var(--duration) var(--ease-out);
  transition: opacity var(--duration) var(--ease-out), transform var(--duration) var(--ease-out), filter var(--duration) var(--ease-out), -webkit-transform var(--duration) var(--ease-out), -webkit-filter var(--duration) var(--ease-out);
}
.show-modal .image__viewport {
  -webkit-filter: grayscale(100%);
          filter: grayscale(100%);
}
</style>