<template>
  <div class="base-table">
    <div class="cols-wrapper">
      <div
        class="cols-head"
        :class="[
          { isSearchConnection: isSearchConnection },
          { isTaxes: isTaxes },
          { left: left },
          { right: right },
          { isCard: isCard },
        ]"
      >
        <div
          class="cols"
          :class="{ isTaxes: isTaxes, isCard: isCard }"
          v-for="(col, i) in cols"
          :key="i"
        >
          {{ col.name }}
        </div>
      </div>
      <div
        class="rows"
        v-for="(row, i) in rows"
        :key="i"
        @click="$emit('clicked', row)"
        :class="[
          { hover: hover },
          { stripped: stripped },
          { isSearchConnection: isSearchConnection },
          { left: left },
          { right: right },
          { isTaxes: isTaxes },
          { isCard: isCard },
        ]"
      >
        <component
          :is="row.url ? 'a' : 'div'"
          class="row"
          v-for="(col, i) in cols"
          :key="i"
          :class="{ isTaxes: isTaxes }"
        >
          <slot :name="'key-' + col.key" :row="row">
            {{ row[col.key] || defaultEmpty }}
          </slot>
        </component>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BaseTable",
  props: {
    cols: {
      type: Array,
      default: () => [],
    },
    rows: {
      type: Array,
      default: () => [],
    },
    left: {
      type: Boolean,
      default: false,
    },
    right: {
      type: Boolean,
      default: false,
    },
    defaultEmpty: {
      type: [String, Number],
      default: "",
    },
    isTaxes: Boolean,
    isCard: Boolean,
    stripped: Boolean,
    hover: Boolean,
    isSearchConnection: Boolean,
  },
};
</script>

<style lang="scss" scoped>
.base-table {
  .cols-head {
    display: grid;
    grid-template-rows: 1fr;
    grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
    column-gap: 20px;
    background: #2c3e501a;
    border-radius: 2px 2px 0 0;
    text-align: left;
    &.isTaxes {
      grid-template-columns: 322px 52px 115px 189px 50px 76px 52px 52px 98px;
      text-align: center;
    }
    &.isCard {
      grid-template-columns: 240px 122px 208px 206px 121px 115px;
      grid-column-gap: 20px;
      height: 60px;
    }
    &.isSearchConnection {
      .cols {
        padding: 10px 16px;
        font-weight: 400;
        font-size: 14px;
        line-height: 19px;
        color: #2c3e50;
      }
    }
    .cols {
      min-width: 35px;
      width: 100%;
      font-size: 14px;
      line-height: 19px;
      color: #71757a;
      white-space: nowrap;
      &.isTaxes {
        font-size: 12px;
        font-style: normal;
        font-weight: 600;
        font-size: 12px;
        line-height: 16px;
        padding: 16px 15px 16px 15px;
        white-space: normal;
        color: #2c3e50;
        &:nth-child(3) {
          background: rgba(0, 185, 45, 0.08);
        }
      }
      &.isCard {
        padding: 20px 15px;
        font-weight: 400;
        font-size: 14px;
        line-height: 19px;

        color: #2c3e50;
      }
    }
  }

  .rows {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
    column-gap: 20px;
    font-size: 14px;
    line-height: 16px;
    border-bottom: 1px solid #eef0f5;
    cursor: pointer;
    &.isTaxes {
      grid-template-columns: 322px 52px 115px 189px 50px 76px 52px 52px 129px;
      text-align: left;
      line-height: 20px;
      color: #2c3e50;
      border-bottom: none;
    }
    &.isCard {
      grid-template-columns: 240px 122px 208px 206px 121px 115px;
      grid-column-gap: 20px;
      .row {
        display: flex;
        padding-top: 10px !important;
        padding-bottom: 10px !important;
        align-items: center;
        text-align: left;
        font-weight: 400;
        font-size: 12px;
        line-height: 20px;

        color: #333333;
      }
    }
    &.isSearchConnection {
      .row {
        font-weight: 400;
        font-size: 12px;
        line-height: 16px;
        color: #333333;
      }
    }
    &:first-child {
      background: rgba(0, 185, 45, 0.08);
    }

    &:hover {
      background: #2c3e500d;

      .btn {
        background-color: #007aff;
        color: white;
      }
    }
    .row {
      display: flex;
      padding: 10px 16px !important;
      text-align: left;
      &.isTaxes {
        &:nth-child(3) {
          background: rgba(0, 185, 45, 0.08);
        }
      }
    }
  }
}
.stripped:nth-child(odd) {
  background: #f7f8fa;
}

.hover {
  &:hover {
    background-color: #eef0f5;
  }

  &:hover button {
    background-color: #007aff;
    color: #fff;
  }
}

.isSearchConnection {
  .row {
    justify-content: space-between !important;
    text-align: left !important;
    white-space: pre-line;
    &:nth-child(3n) {
      display: flex;
      flex-direction: row-reverse;
    }
  }
  .cols {
    text-align: left;
  }
}
.isTaxes {
  .row {
    text-align: left !important;
    padding: 0 !important;
  }
}

.right {
  .row {
    justify-content: right !important;
  }
  .cols {
    text-align: right !important;
  }
}
.left {
  .row {
    justify-content: left !important;
  }
  .cols {
    text-align: left !important;
  }
}
</style>
