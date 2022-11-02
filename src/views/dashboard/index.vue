<template>
  <div>
    <MyCard></MyCard>
    <MySale></MySale>
    <MyObserve></MyObserve>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import MyCard from "./Card";
import MySale from "./Sale";
import MyObserve from "./Observe";
export default {
  name: "Dashboard",
  computed: {
    ...mapGetters(["name"]),
  },
  components: {
    MyCard,
    MySale,
    MyObserve,
  },
  mounted(){
    this.$store.dispatch('getData')
  }
};
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
