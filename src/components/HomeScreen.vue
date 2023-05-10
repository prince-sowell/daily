<template>
  <main>
    <div class="wrapper">
      <h3>Daily demo</h3>
      <p>Demo a custom call interface built using Daily call object for Vue</p>

      <form class="join-form">
        <div class="name-container">
          <label>Your name</label>
          <q-input
            v-model="roomInfo.name"
            type="text"
            outlined
            :rules="[(val) => !!val || 'Field is required']"
            dense
          />
          <label>Daily URL</label>
          <q-input
            :rules="[(val) => !!val || 'Field is required']"
            dense
            v-model="roomInfo.url"
            type="text"
            placeholder="https://your-daily-domain.daily.co/room-name"
            outlined
          />
        </div>
        <div class="submit-container q-my-md">
          <q-btn no-caps unelevated color="primary" @click="joinWithName"
            >Start Session</q-btn
          >
        </div>
      </form>

      <p class="subtext">
        If prompted, select "Allow" to use your camera and mic for this call
      </p>
    </div>
  </main>
</template>

<script>
import { defineComponent, reactive } from "vue";

export default defineComponent({
  name: "HomeScreen",
  emits: ["joinCall"],
  setup(props, { emit }) {
    const roomInfo = reactive({
      name: "",
      url: "",
    });
    // Submit form with prop method defined in App.vue
    const joinWithName = () => {
      emit("joinCall", roomInfo);
    };
    return { roomInfo, joinWithName };
  },
});
</script>

<style scoped>
main {
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1;
  background-color: #fff;
}
.wrapper p {
  margin: 8px 0;
}
.join-form {
  display: flex;
  flex-direction: column;
  width: 300px;
  margin: auto;
}

.join-form label {
  font-size: 12px;
  color: #6b7785;
  text-align: left;
  margin: 8px 0 4px;
}
.join-form input {
  padding: 8px;
  border: 1px solid #c8d1dc;
  border-radius: 12px;
}
.join-form div {
  display: flex;
  justify-content: center;
}
/* .join-form .submit-container button {
  padding: 7px 16px 9px;
  font-size: 12px;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  margin-top: 16px;
  margin-bottom: 12px;
  background-color: #1bebb9;
  cursor: pointer;
} */
.join-form .name-container {
  flex-direction: column;
}
.subtext {
  font-size: 12px;
  color: #6b7785;
}
</style>
