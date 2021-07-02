<template>
  <div class="text-block" :class="{magnifier: magnifier}" v-click-outside="() => $emit('closeAutocomplete')"
       @keydown.down.up.prevent="onUpOrDownClicked">
    <div class="text-block__field">
      <input
        :type="inputType"
        class="text-block__input"
        ref="input"
        @input="inputHandler($event.target.value) "
        :value="value"
        :class="{
          error: hasErrorFeedback,
            'status-error': (this.status == 'danger' || this.status == 'warning') && !hasErrorFeedback,
            'status-success': this.status == 'success' && !hasErrorFeedback

        }"
        required
        v-if="!withMask"
        @keypress.enter="$emit('enterPressed')"
        @focus="$emit('openAutocomplete')"
     />

      <TheMask
        :mask="mask"
        :type="inputType"
        :masked="false"
        :placeholder="inputPlaceholder"
        :value="value"
        @input.native="inputHandler($event.target.value)"
        @focus.native="showPlaceholder = true"
        @blur.native="showPlaceholder = false"
        ref="input"
        class="text-block__input text-block__mask"
        :class="{ error: hasErrorFeedback }"
        required
        v-else
        />


      <div class="underlines" :class="{active: !value && !showPlaceholder}" v-if="withMask && underlines !== 0"
           :style="{width: underlines * 7.2 + 'px'}">
        <template v-for="(number, index) in underlines">
          <span :key="index" v-if="!(value && value.length > index)">_</span>
        </template>
      </div>


      <svg class="text-block__magnifier">
        <use :xlink:href="require('@/assets/icons/sprite.svg') + '#magnifier'"></use>
      </svg>
      <svg class="text-block__trash" @click="clearInput" v-if="trash && value.length == 0">
        <use :xlink:href="require('@/assets/icons/sprite.svg') + '#trash'"></use>
      </svg>
      <label for="" class="text-block__label">{{ labelText }}<span v-if="required" class="text-block__label__red">*</span></label>
      <svg class="text-block__icon" @click="toggleShowPassword" v-if="passwordShowable && !showPassword">
        <use :xlink:href="require('@/assets/icons/sprite.svg') + '#closedEye'"></use>
      </svg>
      <svg class="text-block__icon" @click="toggleShowPassword" v-if="passwordShowable && showPassword">
        <use :xlink:href="require('@/assets/icons/sprite.svg') + '#openedEye'"></use>
      </svg>
      <svg class="text-block__icon" @click="clearTheField(value)" v-if="clearable && value && value.length > 0">
        <use :xlink:href="require('@/assets/icons/sprite.svg') + '#clearIcon'"></use>
      </svg>
    </div>

    <div class="text-block__info" v-if="showFeedback">
      <div class="create-info" :class="status">
        <div class="create-info__progress">
          <div class="create-info__indicator"></div>
        </div>
        <div class="create-info__text">
          {{ text }}
        </div>
      </div>
    </div>
    <div class="text-block__error" v-if="hasErrorFeedback">
      {{ errorText }}
    </div>
    <div class="text-block__options search-options" v-show="options && options.length > 0 && showAutocomplete">
      <PerfectScrollbar class="search-options__wrapper">
        <div class="search-options__item" v-for="(option, index) in options" :key="index"
             @click="onAutocompleteOptionClick(option)" ref="searchOptions">
              {{ optionFields.length ? textByOption(option) : option }}
        </div>
      </PerfectScrollbar>
    </div>
  </div>
</template>
<script>
import {
  PerfectScrollbar
} from "vue2-perfect-scrollbar";
import {
  TheMask
} from 'vue-the-mask';
import vClickOutside from "v-click-outside";
import {mapMutations} from "vuex";

export default {
  props: {
    compareRemove:{
      type: Boolean,
      default: false
    },
    magnifier: {
      type: Boolean,
      default: false
    },
    signup: {
      type: Boolean,
      default: false,
    },
    hasErrorFeedback: {
      type: Boolean,
      default: false,
    },
    errorText: {
      type: String,
    },
    labelText: {
      type: String,
      required: true,
    },
    type: {
      type: String,
      default: "text",
    },
    passwordShowable: {
      type: Boolean,
      default: false,
    },
    clearable: {
      type: Boolean,
      default: false,
    },
    trash: {
      type: Boolean,
      default: false
    },
    value: {
      type: [String, null],
      required: true,
    },
    options: {
      type: Array,
      default: () => [],
    },
    inputFields: {
      type: Array,
    },
    withMask: {
      type: Boolean,
      default: false
    },
    mask: {
      type: String,
      default: "8 (###) ### ## ##"
    },
    placeholder: {
      type: String,
      default: ""
    },
    showAutocomplete: {
      type: Boolean,
      default: false
    },
    underlines: {
      type: Number,
      default: 0
    },
    optionFields: {
      type: Array,
      default: null
    },
    optionSeparator: {
      type: String,
      default: " - "
    },
    isCompareAutocomplete:{
      type:Boolean,
      default:false
    },
    countInput:{
      type:Number,
      default:0
    },
    required: {
      type: Boolean,
      default: false
    }
  },
  directives: {
    clickOutside: vClickOutside.directive
  },
  watch: {
    options: function (values) {
      this.options = values;
    },
  },
  data() {
    return {
      showPassword: false,
      text: "",
      status: "",
      showFeedback: false,
      showPlaceholder: false,
      optionIndex: 1,
      ownInputValue: ""
    };
  },
  computed: {
    inputType() {
      let inputType = this.type;

      if (this.passwordShowable) {
        inputType = "password";
      }

      return inputType;
    },
    inputPlaceholder() {
      return this.showPlaceholder ? this.placeholder : ""
    }
  },
  methods: {
    ...mapMutations({
      setNames:"companyCompare/setNames"
    }),
    clearTheField(val){
      this.$emit('input', '')
      if(this.compareRemove && localStorage.getItem('Bins') ){
        const names = JSON.parse(localStorage.getItem('names') || '[]')
        const bins = JSON.parse(localStorage.getItem('Bins') || '[]')
        if(names.includes(val)) {
          const index = names.indexOf(val)
          names.splice(index, 1)
          localStorage.setItem('names', JSON.stringify(names))
          bins.splice(index, 1)
          localStorage.setItem('Bins', JSON.stringify(bins))
          console.log(index)
        }
        // console.log(index)
      }

    },
    clearInput:function (){
      this.countInput--;
      this.$emit('clear',this.countInput)
    },
    textByOptionCompare(obj) {
      let str = "";
      this.optionFields.forEach((key, index) => {
        str += obj[key]
        if(key == 'name'){
          str += " - БИН/ИИН "
        }
      })
      return str;
    },
    textByOption(obj) {
      let str = "";
      this.optionFields.forEach((key, index) => {
        if(index !== 0) str += this.optionSeparator;
        str += obj[key];
      })
      return str;
    },
    onAutocompleteOptionClick(option){
      this.$emit('getOption', option)
      this.$emit('name', option.name);
      this.$emit('click', option.name);
      this.$emit('biin', option.biin);
      this.$emit('typeId', option.type)
      this.$emit('closeAutocomplete');
    },
    removeField() {
      if (this.inputFields.length > 2) {
        this.inputFields.pop()
      }
    },
    toggleShowPassword() {
      if (!this.showPassword) this.showPasswordHandler("text", true);
      else this.showPasswordHandler("password", false);
    },
    showPasswordHandler(type, showPassword) {
      this.$refs.input.type = type;
      this.showPassword = showPassword;
    },
    inputHandler(value) {
      this.$emit("input", value);
      if (this.signup) this.passwordValidationHandler(value);
    },
    inputHandlerCompare(value){
      this.$emit('inputCompare')
    },
    passwordValidationHandler(value) {
      if (!this.signup) return;

      this.showFeedback = true;

      if (!value) {
        this.status = "";
        this.showFeedback = false;
        return;
      }

      if (value.length < 5) {
        this.status = "danger";
        this.text = "Низкий уровень защиты";
      } else if (value.length >= 5 && value.length <= 10) {
        this.status = "warning";
        this.text = "Средний уровень защиты";
      } else {
        this.status = "success";
        this.text = "Высокий уровень защиты";
      }
    },
    moveOption(autoCompleteOptions, elementType) {
      const selectedOption = autoCompleteOptions.find((option) => option.hasAttribute('id'));

      if(selectedOption){
        const siblingElement = selectedOption[elementType];
        this.$emit('castOption', siblingElement?.innerText || "");
        if (siblingElement?.classList.contains('search-options__item')) {
          selectedOption.removeAttribute("id");
          siblingElement.setAttribute("id", 'searchVariant');
          return;
        }
        selectedOption.removeAttribute("id");
        return;
      }

      let innerText = "";

      if(elementType === 'nextElementSibling')
        innerText = this.getAutocompleteOptionText(autoCompleteOptions?.[0]);

      if(elementType === 'previousElementSibling')
          innerText = this.getAutocompleteOptionText(autoCompleteOptions?.[autoCompleteOptions.length - 1]);

      this.$emit('castOption', innerText);
    },
    getAutocompleteOptionText(autocompleteOption){
      autocompleteOption.setAttribute("id", 'searchVariant');
      return autocompleteOption.innerText;
    },
    onUpOrDownClicked(e) {
      const autoCompleteOptions = this.$refs.searchOptions;
      const length = autoCompleteOptions ? autoCompleteOptions.length : 0;

      if (length > 0) {
        switch(e.which){
          case 40:
            this.moveOption(autoCompleteOptions, 'nextElementSibling');
            break;
          case 38:
            this.moveOption(autoCompleteOptions, 'previousElementSibling');
            break;
        }
      }
    },
  },
  components: {
    PerfectScrollbar,
    TheMask
  },
};

</script>
<style lang="scss" scoped>
*:focus {
  outline: none;
}

.text-block {
  $self: &;
  position: relative;
  width: 100%;

  &__error{
    text-align: left;
  }

  &.magnifier {

    #{$self}__magnifier {
      display: block;
    }

    #{$self}__input {
      padding: 19px 40px 5px 40px;
    }

    #{$self}__label {
      left: 40px;
    }
  }

  &__field {
    position: relative;
    overflow: hidden;
  }

  &__input {
    background: #ffffff;
    border: 1px solid #c4c4c4;
    box-sizing: border-box;
    border-radius: 2px;
    width: 100%;
    height: 48px;
    font-size: 14px;
    line-height: 22px;
    display: flex;
    align-items: center;
    color: #2c3e50;
    padding: 19px 40px 5px 16px;
    transition: 0.3s all;

    &:focus~#{$self}__label,
    &:not(:focus):valid~#{$self}__label {
      top: 15px;
      font-size: 10px;
      color: #71757a;
    }

    &:focus {
      border: 1px solid #2c3e50;
    }

    &::placeholder {
      font-size: 12px;
    }

    &.error {
      background: #ff2e431f;
    }
  }

  &__mask {
    letter-spacing: 1px;
  }

  &__label {
    position: absolute;
    pointer-events: none;
    left: 16px;
    top: 50%;
    transform: translateY(-50%);
    white-space: nowrap;
    overflow: hidden;
    line-height: 40px;
    transition: 0.3s;
    color: #c4c4c4;

    &__red {
      color: rgba(255, 0, 0, 0.53);
    }
  }

  &__icon {
    position: absolute;
    right: 16px;
    top: 50%;
    transform: translateY(-50%);
    width: 14px;
    height: 14px;
    cursor: pointer;
  }

  &__magnifier {
    position: absolute;
    left: 18px;
    top: 50%;
    transform: translateY(-50%);
    width: 16px;
    height: 16px;
    cursor: pointer;
    display: none;
  }

  &__trash {
    position: absolute;
    right: 16px;
    top: 50%;
    transform: translateY(-50%);
    width: 16px;
    height: 16px;
    cursor: pointer;
  }

  &__error {
    font-size: 10px;
    line-height: 14px;
    color: #ff2e43;
    margin-top: 6px;
  }

  &__options {
    position: absolute;
    top: calc(100% + 4px);
    left: 0;
    width: 100%;
    background: #fff;
    z-index: 1000;
    border-radius: 2px;
    border: 1px solid #c4c4c480;
    padding: 12px 0;
  }
}

.search-options {

  &__item {
    padding: 5px 16px;
    line-height: 16px;
    font-size: 14px;
    color: #9da3ac;
    transition: 0.3s all;
    cursor: pointer;

    &:hover {
      background: #eef0f5;
    }
  }
}

.create-info {
  $self: &;

  margin-top: 9px;
  margin-bottom: 12px;

  &.danger {
    #{$self}__indicator {
      background: #ff2e43;
      width: 33.333%;
    }

    #{$self}__text {
      color: #ff2e43;
    }
  }

  &.warning {
    #{$self}__indicator {
      background: #ffcd33;
      width: 66.666%;
    }

    #{$self}__text {
      color: #fab619;
    }
  }

  &.success {
    #{$self}__indicator {
      background: #00b92d;
      width: 100%;
    }

    #{$self}__text {
      color: #00b92d;
    }
  }

  &__progress {
    height: 1px;
    background: #c4c4c4;
    position: relative;
  }

  &__indicator {
    position: absolute;
    top: 33.333%;
    transform: translateY(-50%);
    height: 2px;
    border-radius: 10px;
  }

  &__text {
    font-style: normal;
    font-weight: normal;
    font-size: 10px;
    line-height: 14px;
    display: flex;
    align-items: center;
    margin-top: 4px;
  }
}

.ps {
  &__thumb-y {
    width: 2px;
  }
}

.underlines {
  display: flex;
  position: absolute;
  bottom: 24px;
  left: 40px;
  height: 1px;
  width: 100%;
  pointer-events: none;
  justify-content: flex-end;

  &.active {
    display: none;
  }

  &>div {
    display: flex;
  }

  span {
    display: inline-block;
    margin-right: 2px;
  }
}

#searchVariant {
  background: #eef0f5;
}

</style>
