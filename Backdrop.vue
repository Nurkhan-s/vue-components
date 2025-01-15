<template>
  <div class="backdrop-component" v-show="isOpened">
    <div class="backdrop__overlay"
         ref="backdrop-overlay"
         v-if="overlay"
         :style="{'z-index': overlayZIndex}"
    />
    <div
      ref="backdrop-panel"
      class="backdrop"
      :class="{ '_backdrop_moving': isMoving }"
      :style="{ 'height': maxHeight }"
    >
      <div class="backdrop__header-notch" ref="backdrop-header-notch"></div>
      <div @click="$emit('closed')" class="backdrop__header-close-icon" v-if="!hideCross">
        <img src="@/assets/icons/backdrop-close-icon.svg" alt="">
      </div>
      <div class="backdrop__inner-content" ref="backdrop-content">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'nuxt-property-decorator';
import { animateByFrame } from '@/utils/helpers';

@Component
export default class NonCringyBackdrop extends Vue {
  @Prop({ default: 200 }) animationDuration!: number;
  @Prop({ default: 100 }) ignoreSwipeSize!: number;
  @Prop({ default: true }) overlay!: boolean;
  @Prop({ default: '50%' }) maxHeight!: string;
  @Prop({ default: true }) swipeAble!: boolean;
  @Prop({ default: false }) isVisible!: boolean;
  @Prop({ default: false }) hideCross!: boolean;
  @Prop({ default: '2000' }) overlayZIndex!: string;
  @Prop() minHeight?: string;

  isOpened: boolean = this.isVisible;
  touchStartPosition: number = 0;
  contentTouchStartPositionY: number = 0;
  currentTouchPosition: number = 0;
  isExpanded: boolean = true;
  isMoving: boolean = false;

  panel!: HTMLElement;
  overlayElement!: HTMLElement;
  panelContent!: HTMLElement;
  panelHeader!: HTMLElement;

  mounted() {
    if (!this.swipeAble) return;
    this.declareBackdropDOMElements();
    this.addBackdropEventListeners();
  }

  destroyed() {
    if (!this.swipeAble) return;
    this.removeBackdropEventListeners();
  }

  declareBackdropDOMElements() {
    this.panel = this.$refs['backdrop-panel'] as HTMLElement;
    this.overlayElement = this.$refs['backdrop-overlay'] as HTMLElement
    this.panelContent = this.$refs['backdrop-content'] as HTMLElement;
    this.panelHeader = this.$refs['backdrop-header-notch'] as HTMLElement;
  }

  addBackdropEventListeners() {
    if (this.overlay) {
      this.overlayElement.addEventListener('touchmove', this.handleOverlayTouchMove);
    }

    this.panelHeader.addEventListener('touchmove', this.handleHeaderTouchMove, { passive: false });
    this.panelHeader.addEventListener('touchend', this.handleTouchEnd);

    this.panelContent.addEventListener('touchstart', this.handleContentTouchStart, { passive: false });
    this.panelContent.addEventListener('touchmove', this.handleContentTouchMove, { passive: false });
    this.panelContent.addEventListener('touchend', this.handleTouchEnd);
  }

  removeBackdropEventListeners() {
    if (this.overlayElement) {
      this.overlayElement.removeEventListener('touchmove', this.handleOverlayTouchMove);
    }

    this.panelHeader.removeEventListener('touchmove', this.handleHeaderTouchMove);
    this.panelHeader.removeEventListener('touchend', this.handleTouchEnd);

    this.panelContent.removeEventListener('touchstart', this.handleContentTouchStart);
    this.panelContent.removeEventListener('touchmove', this.handleContentTouchMove);
    this.panelContent.removeEventListener('touchend', this.handleTouchEnd);
  }

  resetBackdrop() {
    if (this.panel.style.transform) {
      this.panel.style.transform = 'translate3d(0, 0, 1px)';
      (this.panel.style.transform as any) = null;
    }
    this.currentTouchPosition = 0;
    this.isMoving = false;
  }

  drawShow(progress: number) {
    if (this.panel) {
      this.panel.style.bottom = ((progress * 100) - 100).toFixed(2) + 'vh';
    }
    if (this.overlayElement) {
      this.overlayElement.style.opacity = (progress).toFixed(2);
    }
  }

  drawHide(progress: number) {
    if (this.panel) {
      this.panel.style.bottom = (-(progress * 100)) + 'vh';
    }
    if (this.overlayElement) {
      let opacity = (+this.overlayElement.style.opacity - (progress / 2));
      opacity = opacity < 0 ? 0 : opacity;
      this.overlayElement.style.opacity = opacity.toFixed(2);
    }
  }

  toggleBackdropOpen(show = false): void {

    const draw = show
      ? this.drawShow
      : this.drawHide;

    animateByFrame({
      duration: this.animationDuration,
      timing(timeFraction) {
        return timeFraction;
      },
      draw,
    });
  }

  handleOverlayTouchMove(e: TouchEvent) {
    this.preventDefault(e);
  }

  handleHeaderTouchMove(e: TouchEvent) {
    this.touchMove(e);
    this.preventDefault(e);
  }

  handleContentTouchStart(e: TouchEvent): void {
    this.contentTouchStartPositionY = e.changedTouches[0].clientY;

  }

  handleContentTouchMove(e: TouchEvent): void {
    if (this.contentTouchStartPositionY === 0) {
      this.contentTouchStartPositionY = e.changedTouches[0].clientY;
    }

    const dirY = e.changedTouches[0].clientY - this.contentTouchStartPositionY;

    // Touch direction DOWN
    if (dirY > 0) {
      if (this.panelContent.scrollTop <= 0) {
        this.touchMove(e);
        this.preventDefault(e);
      }
      return;
    }

    // Touch direction UP
    const contentWasScrolledToBottom = (
      this.panelContent.scrollTop === (this.panelContent.scrollHeight - this.panelContent.offsetHeight)
    );
    if (contentWasScrolledToBottom) {
      this.preventDefault(e);
    }
  }

  touchMove(e: TouchEvent): void {
    this.isMoving = true;
    const y = e.changedTouches[0].clientY;
    if (this.touchStartPosition === 0) {
      this.touchStartPosition = y;
    }

    if (this.isExpanded) {
      this.currentTouchPosition = y - this.touchStartPosition;
    } else {
      this.currentTouchPosition = y - this.touchStartPosition + this.minifiedSizeInPixels!;
    }

    if (this.currentTouchPosition > 0) {
      this.panel.style.transform = `translate3d(0, ${this.currentTouchPosition}px, 1px)`;

      const opacityPerHeight = 1 / this.panel.offsetHeight;
      if (this.overlay) {
        this.overlayElement.style.opacity = `${1 - (opacityPerHeight * this.currentTouchPosition)}`;
      }
    }
  }

  handleTouchEnd(e: TouchEvent): void {
    this.isMoving = false;
    if (!this.isExpanded) {
      if (Math.abs(this.touchStartPosition - e.changedTouches[0].clientY) < this.ignoreSwipeSize) {
        this.panel.style.transform = `translate3d(0, ${this.minifiedSizeInPixels}px, 1px)`;
      } else if (this.touchStartPosition > this.currentTouchPosition) {
        this.isExpanded = true;
        this.resetBackdrop();
      }
    }
    this.touchStartPosition = 0;
    this.contentTouchStartPositionY = 0;
    if (this.currentTouchPosition < this.ignoreSwipeSize) {
      if (this.panel.style.transform) {
        this.panel.style.transform = 'translate3d(0, 0, 1px)';
        (this.panel.style.transform as any) = null;
      }
      if (this.overlay) {
        this.overlayElement.style.opacity = '1';
      }
    } else {
      if (!this.isCloseable) {
        this.isExpanded = false;
        this.panel.style.transform = `translate3d(0, ${this.minifiedSizeInPixels}px, 1px)`;
        return;
      }
      this.toggleBackdropOpen(false)
      setTimeout(() => {
        this.isOpened = false;
        this.$emit('closed');
        this.resetBackdrop();
      }, this.animationDuration);
    }
  }

  protected preventDefault(event: TouchEvent): void {
    if (event.cancelable) {
      event.stopPropagation();
      event.preventDefault();
    }
  }

  get isCloseable() {
    return !this.minHeight;
  }

  get minifiedSizeInPixels() {
    if (!this.minHeight) return;
    let minHeightNumber = parseInt(this.minHeight) / 100;
    return this.panel.offsetHeight - (this.panel.offsetHeight * minHeightNumber);
  }

  @Watch('isVisible')
  visibilityChanged(val: boolean) {
    if (val != this.isOpened) {
      this.toggleBackdropOpen(val);
    }
    if (!val) {
      setTimeout(() => {
        this.isOpened = this.isVisible;
      }, this.animationDuration);
    } else {
      this.isOpened = this.isVisible;
    }
    if (val) {
      document.body.style.height = '100vh';
      document.body.style.overflowY = 'hidden';
    } else {
      document.body.style.height = 'auto';
      document.body.style.overflowY = 'auto';
    }
  }

  beforeDestroy() {
    document.body.style.height = 'auto';
    document.body.style.overflowY = 'auto';
  }
}
</script>


<style lang="scss" scoped>

%backdrop-notch {
  &::after {
    position: absolute;
    top: toVw(8);
    left: 50%;
    transform: translateX(-50%);

    display: block;
    content: "";

    width: toVw(32);
    height: toVw(4);

    background-color: #ddd;
    border-radius: toVw(3);
  }
}

%overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(26, 26, 26, 0.5);;
  z-index: 2000;
  @extend %background;
}

@supports (backdrop-filter: none) or (-webkit-backdrop-filter: none) {
  %background {
    -webkit-backdrop-filter: blur(6px);
    backdrop-filter: blur(6px);
  }
}

%a-scroll-momentum {
  -webkit-overflow-scrolling: touch;
}


%safe-area-inset-bottom {
  @include safe-area-padding-bottom;
}

// class for body
._backdrop_open {
  overflow: hidden;
}

.backdrop {
  position: fixed;
  z-index: 99998;
  left: 0;
  bottom: 0;
  width: 100%;
  min-height: toVw(136);
  max-height: 100vh;
  display: flex;
  flex-direction: column;
  background: $white;
  border-radius: toVw(16) toVw(16) 0 0;
  will-change: transform, bottom;
  transition: transform 0.2s;
  .topbar {
    position: fixed;
    z-index: 100;
  }
  .topbar + * {
    padding-top: toVw($topbar-height);
  }
}

._backdrop_moving {
  transition: none;
}
// .backdrop_ext {}

.backdrop__overlay {
  @extend %overlay;
  will-change: opacity;
}

.backdrop__header-notch {
  z-index: 2020;
  padding: toVw(46) 0 0;
  margin: toVw(-22) 0 0;

  @extend %backdrop-notch;
}

.backdrop__header-close-icon {
  position: absolute;
  z-index: 2000;
  height: toVw(24);
  width: toVw(24);
  top: toVw(16);
  right: toVw(16);
}

.backdrop__inner-content {
  @extend %a-scroll-momentum;
  @extend %safe-area-inset-bottom;
  overflow-x: hidden;
  overflow-y: auto;
  height: 100%;
  margin-top: toVw(-12)
}

</style>
