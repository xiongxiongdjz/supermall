<template>
  <div>
    <detai-nav-bar />
  </div>
</template>

<script>
import DetaiNavBar from "./childCompos/DetailNavBar";

export default {
  data() {
    return {
      iid: null
    };
  },
  components: {
    DetaiNavBar
  },
  created() {
    this.iid = this.$route.params.iid;
  }
};
</script>

<style lang="scss" scoped></style>
