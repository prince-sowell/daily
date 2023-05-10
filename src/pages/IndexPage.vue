<template>
  <q-page class="app">
    <home-screen v-if="appState === 'idle'" @join-call="joinCall" />
    <call-tile
      v-else-if="appState === 'incall'"
      @leave-call="leaveCall"
      :name="roomInfos.name"
      :room-url="roomInfos.roomUrl"
  /></q-page>
</template>

<script>
import { reactive, defineComponent, ref } from "vue";
import CallTile from "src/components/CallTile.vue";
import HomeScreen from "src/components/HomeScreen.vue";

export default defineComponent({
  name: "IndexPage",
  components: { HomeScreen, CallTile },
  setup() {
    const appState = ref("idle");
    const roomInfos = reactive({
      name: "Guest",
      roomUrl: "https://julienprince.daily.co/prince",
    });

    const joinCall = (infos) => {
      if (!infos.url && !infos.name) {
        console.log("%cIndexPage.vue line:29 no infos", "color: #007acc;");
        return;
      }
      roomInfos.name = infos.name;
      roomInfos.roomUrl = infos.url;
      appState.value = "incall";
    };
    const leaveCall = () => {
      appState.value = "idle";
    };

    return { roomInfos, appState, joinCall, leaveCall };
  },
});
</script>
<style>
.app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  overflow-x: hidden;
  background-color: #121a24;
}
a {
  text-decoration: none;
  color: #2c3e50;
  display: flex;
  align-items: center;
}
body {
  margin: 0;
}
</style>
