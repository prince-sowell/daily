<template>
  <div class="controls">
    <div class="devices">
      <button @click="handleAudioClick">
        <template v-if="participant.audio">
          <img class="icon" :src="micOn" alt="" />
        </template>
        <template v-else>
          <img class="icon" :src="micOff" alt="" />
        </template>
      </button>

      <button @click="handleVideoClick">
        <template v-if="participant.video">
          <img class="icon" :src="videoOn" alt="" />
        </template>
        <template v-else>
          <img class="icon" :src="videoOff" alt="" />
        </template>
      </button>

      <template v-if="supportsScreenshare">
        <button :disabled="disableScreenShare" @click="handleScreenshareClick">
          <img class="icon" :src="screenShare" alt="" />
        </button>
      </template>
    </div>

    <button class="leave" @click="$emit('leaveCall')">
      <img class="icon" :src="leave" alt="" />
    </button>
  </div>
</template>

<script>
import daily from "@daily-co/daily-js";

import leave from "src/assets/leave_call.svg";
import micOff from "src/assets/mic_off.svg";
import micOn from "src/assets/mic_on.svg";
import screenShare from "src/assets/screenshare.svg";
import videoOff from "src/assets/vid_off.svg";
import videoOn from "src/assets/vid_on.svg";
import { ref, onMounted, defineComponent } from "vue";

export default defineComponent({
  name: "CallControls",
  props: [
    "participant",
    "handleVideoClick",
    "handleAudioClick",
    "handleScreenshareClick",
    "disableScreenShare",
  ],

  emits: ["leaveCall"],
  setup() {
    const supportsScreenshare = ref(false);
    onMounted(() => {
      supportsScreenshare.value = daily.supportedBrowser().supportsScreenShare;
    });
    return {
      leave,
      micOn,
      micOff,
      screenShare,
      videoOn,
      videoOff,
      supportsScreenshare,
    };
  },
});
</script>

<style scoped>
.controls {
  position: absolute;
  bottom: 12px;
  left: 8px;
  justify-content: space-between;
  display: flex;
  width: calc(100% - 16px);
}

.devices {
  border-radius: 12px;
  background-color: #121a24;
  opacity: 0.85;
  padding: 14px 10px 15px;
}
button {
  background-color: transparent;
  border: none;
  cursor: pointer;
}
button:not(.leave) {
  margin: 0 4px;
  width: 36px;
  height: 26px;
}
button.leave {
  background-color: #f63135;
  opacity: 0.85;
  padding: 14px 16px 15px;
  border-radius: 12px;
}
button:disabled {
  cursor: not-allowed;
  opacity: 0.4;
}
.icon {
  height: 24px;
}
</style>
