<template>
  <svg>
    <use
      :xlink:href="require('@/assets/icons/sprite.svg') + `#${imageName}`"
    />
  </svg>
</template>
<script>
export default {
  computed: {
    imageName () {
      return this.$slots.default[0].text.replace(/\s/g, '')
    }
  }
}
</script>
<style lang="scss" scoped>
  .ic {
    &-size {

      @for $size from 1 through 100 {
        &-#{$size} {
          width: #{$size}px;
          height: #{$size}px;
          min-width: #{$size}px;
        }
      }

      @for $height from 1 through 100 {
        &-height {
          height: #{$height};
        }
      }
      @for $width from 1 through 100 {
        &-width {
          width: #{$width};
          min-width: #{$width};
        }
      }
    }
  }
</style>
