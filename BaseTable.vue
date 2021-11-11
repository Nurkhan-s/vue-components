<template>
  <div class="base-table">
    <div class="cols-wrapper">
      <div
        class="cols-head"
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
      >
        <component
          :is="row.url ? 'a' : 'div'"
          class="row"
          v-for="(col, i) in cols"
          :key="i"         
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
    .cols {
      min-width: 35px;
      width: 100%;
      font-size: 14px;
      line-height: 19px;
      color: #71757a;
      white-space: nowrap;     
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
