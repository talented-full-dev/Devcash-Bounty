<template>
  <div
    :class="{
        'bg-c-pending': status=='pending',
        'bg-c-success': status=='approved',
        'bg-c-danger': status=='rejected',
        'bg-c-text': status=='feedback'}"
    class="w-full h-px opacity-50"
  ></div>
</template>
<script>
export default {
  props: {
    status: null
  }
};
</script>