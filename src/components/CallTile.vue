<template>
  <main>
    <!-- loading is true when the call is in the "joining-meeting" meeting state -->
    <template v-if="loading">
      <div class="loading-spinner"><loading-tile /></div>
    </template>

    <template v-else>
      <div class="wrapper">
        <template v-if="error">
          <p class="error-text">{{ error }}</p>
          <!-- refreshing will leave the call and reset the app state -->
          <button class="error-button" @click="leaveAndCleanUp">Refresh</button>
        </template>

        <template v-if="showPermissionsError">
          <permissions-error-msg :reset="leaveAndCleanUp" />
        </template>

        <template v-else>
          <div
            :class="screen ? 'tile-container' : 'tile-container full-height'"
          >
            <template v-if="screen">
              <screenshare-tile :participant="screen" />
            </template>

            <div v-if="participants" class="participants-container">
              <template v-for="p in participants" :key="p.session_id">
                <video-tile
                  :participant="p"
                  :handle-video-click="handleVideoClick"
                  :handle-audio-click="handleAudioClick"
                  :handle-screenshare-click="handleScreenshareClick"
                  :disable-screen-share="screen && !screen?.local"
                  @leave-call="leaveAndCleanUp"
                />
              </template>

              <template v-if="count === 1">
                <waiting-card :url="roomUrl" />
              </template>
            </div>
          </div>
        </template>
      </div>
    </template>
  </main>
</template>

<script>
import daily from "@daily-co/daily-js";

import WaitingCard from "./WaitingCard.vue";
import VideoTile from "./VideoTile.vue";
import ScreenshareTile from "./ScreenshareTile.vue";
import LoadingTile from "./LoadingTile.vue";
import PermissionsErrorMsg from "./PermissionsErrorMsg.vue";
import { defineComponent, onMounted, ref, initCall, onUnmounted } from "vue";

export default defineComponent({
  name: "CallTile",
  components: {
    VideoTile,
    WaitingCard,
    ScreenshareTile,
    LoadingTile,
    PermissionsErrorMsg,
  },
  props: ["name", "roomUrl"],
  emits: ["leaveCall"],
  setup(props, { emit }) {
    const option = {
      url: props.roomUrl,
    };
    console.log("🚀 ~ file: CallTile.vue:76 ~ setup ~ option:", option);
    const callObject = ref(null);
    const participants = ref(null);
    const count = ref(0);
    const error = ref(false);
    const loading = ref(false);
    const showPermissionsError = ref(false);
    const screen = ref(null);

    const initCall = () => {
      // Create instance of Daily call object
      const co = daily.createCallObject(option);
      // Assign in data obj for future reference
      callObject.value = co;
      // Join the call with the name set in the Home.vue form
      co.join({ userName: props.name });
      // Add call and participant event handler
      // Visit https://docs.daily.co/reference/daily-js/events for more event info
      co.on("joining-meeting", handleJoiningMeeting)
        .on("joined-meeting", updateParticpants)
        .on("participant-joined", updateParticpants)
        .on("participant-updated", updateParticpants)
        .on("participant-left", updateParticpants)
        .on("error", handleError)
        // camera-error = device permissions issue
        .on("camera-error", handleDeviceError);
    };

    // Temporary show loading view while joining the call
    const handleJoiningMeeting = () => {
      loading.value = true;
    };
    /**
     * This is called any time a participant update registers.
     * In large calls, this should be optimized to avoid re-renders.
     * For example, track-started and track-stopped can be used
     * to register only video/audio/screen track changes.
     */
    const updateParticpants = (e) => {
      console.log("[EVENT] ", e);
      if (!callObject.value) return;

      const p = callObject.value.participants();
      count.value = Object.values(p).length;
      participants.value = Object.values(p);

      const screen = participants.value.filter((p) => p.screenVideoTrack);
      if (screen?.length && !screen.value) {
        console.log("[SCREEN]", screen);
        screen.value = screen[0];
      } else if (!screen?.length && screen.value) {
        screen.value = null;
      }
      loading.value = false;
    };

    // Show local error in UI when daily-js reports an error
    const handleError = (e) => {
      console.log("[ERROR] ", e);
      error.value = e?.errorMsg;
      loading.value = false;
    };

    // Show permissions error in UI to alert local participant
    const handleDeviceError = () => {
      showPermissionsError.value = true;
    };

    // Toggle local microphone in use (on/off)
    const handleAudioClick = () => {
      const audioOn = callObject.value.localAudio();
      callObject.value.setLocalAudio(!audioOn);
    };

    // Toggle local camera in use (on/off)
    const handleVideoClick = () => {
      const videoOn = callObject.value.localVideo();
      callObject.value.setLocalVideo(!videoOn);
    };

    // Toggle screen share
    const handleScreenshareClick = () => {
      if (screen.value?.local) {
        callObject.value.stopScreenShare();
        screen.value = null;
      } else {
        callObject.value.startScreenShare();
      }
    };

    // leave call, destroy call object, and reset local state values
    const leaveAndCleanUp = () => {
      if (screen.value?.local) {
        callObject.value.stopScreenShare();
      }
      callObject.value.leave().then(() => {
        callObject.value.destroy();

        screen.value = null;
        emit("leaveCall");
      });
    };

    onMounted(() => {
      initCall();
    });

    onUnmounted(() => {
      if (!callObject.value) return;
      // Clean-up event handlers
      callObject.value
        .off("joining-meeting", handleJoiningMeeting)
        .off("joined-meeting", updateParticpants)
        .off("participant-joined", updateParticpants)
        .off("participant-updated", updateParticpants)
        .off("participant-left", updateParticpants)
        .off("error", handleError)
        .off("camera-error", handleDeviceError);
    });
    return {
      loading,
      error,
      participants,
      leaveAndCleanUp,
      showPermissionsError,
      screen,
      handleVideoClick,
      handleAudioClick,
      handleScreenshareClick,
      count,
    };
  },
});
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Ropa+Sans&display=swap");
main {
  font-family: "Ropa Sans", sans-serif;
  background-color: #121a24;
  height: 100%;
  position: relative;
}
.wrapper {
  max-width: 1200px;
  margin: auto;
  padding: 0 16px;
  height: 100%;
}
.loading-spinner {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.tile-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.participants-container {
  display: flex;
  margin: 0 -20px;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  width: 100%;
  background-color: #121a24;
  height: inherit;
}
p {
  color: white;
}
.error-text {
  color: #e71115;
}
.full-height {
  height: 100%;
}
.error-button {
  color: #fff;
  background-color: #121a24;
  border: none;
  font-size: 12px;
  border: 1px solid #2b3f56;
  border-radius: 8px;
  padding: 8px 12px;
  cursor: pointer;
}
</style>
