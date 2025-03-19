<template>
  <div id="switch">
    <input @click="handleClick" :checked="value" type="checkbox">
    <div class="knob">
      <span>m</span>
      <span>ft</span>
    </div>
  </div>
</template>
<script>
export default {
  name: "SwitchUnit",
  props: {
    value: Boolean,
    callback: Function
  },
  methods: {
    handleClick() {
      this.$emit("switched");
    }
  }
};
</script>
<style scoped>
#switch {
  --switch-width: 60;
  --switch-height: 30;
  position: absolute;
  top: 10px;
  right: 10px;
  width: calc(1px * var(--switch-width));
  height: calc(1px * var(--switch-height));
  border-radius: 5px;
  background: #555;
}

input {
  position: relative;
  opacity: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
  z-index: 3;
}

.knob {
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding: 0 2px;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}

.knob > span {
  position: relative;
  color: #ddd;
  z-index: 2;
}

.knob::before {
  content: "";
  position: absolute;
  top: 0;
  left: calc(100% - (1px * (var(--switch-height))));
  width: calc(1px * (var(--switch-height) - 4));
  height: calc(1px * (var(--switch-height) - 4));
  margin: 2px;
  background: rgb(21, 98, 214);
  border-radius: 5px;
  z-index: 1;
  transition: all ease-in .1s;
}

input:checked + .knob::before {
  right: initial;
  left: 0;
}
</style>


