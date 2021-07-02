<template>
  <div>
    <div class="modal">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "BaseModal",
  data() {
    return {
      show: false
    };
  }
};
</script>

<style lang="scss" scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1050;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #2c3e50cc;
  display: flex;
  justify-content: center;
  align-items: center;

  &__content {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    max-width: 556px;
    padding: 60px;
    width: 100%;
    background-color: #fff;
    border-radius: 10px;

    .money {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      padding: 10px;
      background: #00b92d1a;
    }

    .request-note {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      padding: 10px;
      background: #00b92d1a;
    }

    .icon-mail {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      padding: 10px;
      background: #00b92d1a;
    }

    h2 {
      margin-top: 24px;
      font-weight: 600;
      font-size: 22px;
      line-height: 30px;
      text-align: center;
      color: hsl(210, 29%, 24%);
    }

    p {
      font-size: 14px;
      line-height: 19px;
      text-align: center;
      color: #2c3e50;
    }

    .close-btn {
      margin-top: 64px;
      width: 138px;

      &--device-enter{
        margin-top: 20px;
      }
    }

    .confirm-btn {
      color: #000;
      font-size: 16px;
      line-height: 22px;

      width: 182px;
    }

    .top-up {
      margin-top: 34px;
      width: 160px;
    }

    .card-logos {
      display: flex;
      justify-content: center;
      margin-top: 10px;

      svg {
        height: 20px;
        width: 60px;
      }
    }

    .input {
      max-width: 360px;
      width: 100%;
      margin-top: 24px;
    }

    .close-icon {
      position: absolute;
      right: 16px;
      top: 16px;
      width: 40px;
      height: 40px;
      cursor: pointer;
    }

    .device-icon{
       width: 100px;
      height: 100px;
    }

    .warning{
        max-width: 360px;
        font-size: 14px;
        line-height: 30px;
        text-align: center;
        color: #2C3E50;
    }
  }
}

.note {
  position: relative;
  margin-top: 8px;

  .note-box {
    font-size: 16px;
    line-height: 24px;
    color: #2c3e50;
    background: #ffcd331a;
    border-radius: 2px;
    padding: 8px 16px 8px 40px;

    .note-svg {
      width: 16px;
      height: 16px;
      position: absolute;
      top: 8px;
      left: 16px;
    }
  }
}

.comment {
  margin-top: 24px;
  max-width: 360px;
}

.send-btn {
  margin-top: 40px;
}

.topmodal {
  max-width: 360px;
  width: 100%;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
</style>
