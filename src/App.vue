<template>
  <div id="app">
    <div class="content-view">
      <router-view />
    </div>
    <tabbar v-model="active">
      <tabbar-item icon="home-o" to="/">标签</tabbar-item>
      <tabbar-item icon="setting-o" to="/about">标签</tabbar-item>
    </tabbar>
  </div>
</template>

<script>
import { Tabbar, TabbarItem } from "vant";
export default {
  name: "navbar",
  components: {
    Tabbar,
    TabbarItem,
  },
  data() {
    return {
      active: 0,
    };
  },
};
</script>

<style lang="scss">
html,
body {
  width: 100%;
  height: 100%;
}
#app {
  width: 100%;
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
  // 内容布局
  .content-view {
    height: calc(100% - 50px);
  }
}
</style>
