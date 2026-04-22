<template>
  <div class="slide-number">
    <SlideCurrentNo /> / <SlidesTotal />
  </div>
</template>

<style scoped>
.slide-number {
  position: absolute;
  right: 0.9rem;
  bottom: 0.6rem;
  font-size: 0.7rem;
  font-variant-numeric: tabular-nums;
  color: rgba(0, 0, 0, 0.35);
  pointer-events: none;
  user-select: none;
  z-index: 10;
}

.dark .slide-number {
  color: rgba(255, 255, 255, 0.4);
}

/* Hide on the cover (first) slide */
[data-slidev-no="1"] .slide-number {
  display: none;
}
</style>
