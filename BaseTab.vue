<template>
  <div class="tab">
    <div class="tab-wrapper">
      <div
        class="tab-items"
        v-for="(item, i) in tabs"
        :key="i"
        :class="{ dark: dark, active: selectedTab.id === item.id}"
        @click="activate(item)"
      >
        {{ item.name }} {{ counting ?   `(${count})` : "" }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    selectedTab:{
      type: Object,
      default: () => {
        return {
          id: 1
        }
      }
    },
    dark: {
      type: Boolean,
      default: false
    },
    count:{
      type: Number,
      default: 0
    },
    tabs: {
      type: Array,
      required: true
    },
    counting:{
      type: Boolean,
      default: false
    }
  },
  methods: {
    activate(item) {
      this.$emit('clicked', item);
    },
  },

  created() {
    this.active = this.tabs.length > 0 ? this.tabs[0] : null;
  },
};
</script>

<style lang="scss" scoped>
.tab {
  border-bottom: 1px solid #ECF1F7;
  &-wrapper {
    display: flex;
  }

  &-items {
    cursor: pointer;
    font-size: 14px;
    min-width: 100px;
    line-height: 19px;
    text-align: center;
    color: #9da3ac;
    padding: 5px 15px;
    font-weight: 600;
    white-space: nowrap;
  }
}

.active {
  color: #2c3e50;
  font-weight: 600;
  position: relative;
  $self: &;

  &::after {
    content: "";
    position: absolute;
    top: calc(100% - 0.5px);
    left: 0;
    width: 100%;
    height: 2px;
    background: #ffcd33;
  }
}

.dark.active{

  &::after{
      background: #2C3E50;
  }
}
</style>
