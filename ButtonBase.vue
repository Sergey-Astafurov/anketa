<template>
<button class="btn__baseee">
        {{label}}
      </button>
</template>

<script setup>
const props = defineProps({
  label: {
    type: String,
    required: true
  }
})
</script>

<style scoped>
.btn__baseee {
  padding: 15px 35px;
  border-radius: 8px;
  color: #fff;
  font-size: 14px;
  font-weight: 400;
  line-height: 16px;
  text-align: center;
  cursor: pointer;
  transition: 0.3s;
  position: relative;
  z-index: 10;
  /* background: #f90042; */
  border: 1px solid #f90042;
}
.btn__baseee::after{
    position: absolute;
    top: 1px;
    right: 1px;
    bottom: 1px;
    left: 1px;
    content: "";
    border-radius: 8px;
    z-index: -1;
    background: #f90042;
}
</style>
