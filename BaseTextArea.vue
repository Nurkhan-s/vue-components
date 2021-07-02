<template>
  <div class="textarea">
    <textarea
      name="texarea"
      class="text-area"
      :rows="rows"
      required
      @input="onChange"
      :value="value"
      :class="{error: hasErrorFeedback}"
    >
    </textarea>
    <label for="" class="text-area__label">{{ labelText }}</label>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: "",
    },
    rows: {
      type: Number,
      default: 9,
    },
    labelText: {
      type: String,
      required: true,
    },
    hasErrorFeedback: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onChange(e) {
      this.$emit("input", e.target.value);
      this.$emit("changed", e.target.value);
    },
  },
};
</script>

<style lang="scss" scoped>
.textarea {
  position: relative;
}

.text-area {
  $self: &;
  resize: none;
  outline: none;
  width: 100%;
  background: #ffffff;
  border: 1px solid #c4c4c4;
  box-sizing: border-box;
  border-radius: 2px;
  font-size: 14px;
  line-height: 22px;
  padding: 19px 40px 5px 16px;
  &::placeholder {
    font-size: 12px;
  }

   &.error {
      background: #ff2e431f;
    }
    
  &:focus ~ #{$self}__label,
  &:not(:focus):valid ~ #{$self}__label {
    top: 15px;
    font-size: 10px;
    color: #71757a;
  }
  &__label {
    position: absolute;
    pointer-events: none;
    left: 16px;
    top: 22px;
    transform: translateY(-50%);
    white-space: nowrap;
    overflow: hidden;
    line-height: 40px;
    transition: 0.3s;
    color: #c4c4c4;
  }
  &::placeholder {
    font-size: 12px;
  }
}

</style>
