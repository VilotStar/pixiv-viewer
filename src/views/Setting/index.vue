<template>
  <div class="setting">
    <van-cell center title="Local Cache" :label="size.local | bytes">
      <template #right-icon>
        <van-button type="primary" size="small" @click="clearCache('local')">Clear</van-button>
      </template>
    </van-cell>
    <van-cell center title="Session Cache" :label="size.session | bytes">
      <template #right-icon>
        <van-button type="info" size="small" @click="clearCache('session')">Clear</van-button>
      </template>
    </van-cell>
    <van-cell center title="Show R-18" label="Contains nudity or sexual depictions">
      <template #right-icon>
        <van-switch :value="currentSETTING.r18" @input="onR18Change($event, 1)" size="24" />
      </template>
    </van-cell>
    <van-cell center title="Show R-18 Gore" label="Contains gory or disgusting content">
      <template #right-icon>
        <van-switch :value="currentSETTING.r18g" @input="onR18Change($event, 2)" size="24" />
      </template>
    </van-cell>
  </div>
</template>

<script>
import { Cell, Switch, Button, Dialog } from "vant";
import { mapState, mapActions } from "vuex";
import { LocalStorage, SessionStorage } from "@/utils/storage";
export default {
  name: "Setting",
  data() {
    return {
      currentSETTING: {
        r18: false,
        r18g: false
      },
      size: {
        local: 0,
        session: 0
      }
    };
  },
  computed: {
    ...mapState(["SETTING"])
  },
  watch: {
    $route() {
      this.calcCacheSize();
    }
  },
  methods: {
    onR18Change(checked, type) {
      let name;
      if (type === 1) name = "R-18";
      if (type === 2) name = "R-18G";

      if (checked) {
        Dialog.confirm({
          message: `Are you sure you want to enable ${name} content?`,
          confirmButtonColor: "black",
          cancelButtonColor: "#1989fa",
          closeOnPopstate: true
        })
          .then(() => {
            if (type === 1) this.currentSETTING.r18 = checked;
            if (type === 2) {
              this.currentSETTING.r18g = checked;
              setTimeout(() => {
                Dialog.alert({
                  message: `Please note that turning on the ${name} switch may have an irreversible effect on your physical and mental health, you have been warned!`
                });
              }, 200);
            }
          })
          .catch(() => {
            console.log("Operation Cancelled");
          });
      } else {
        if (type === 1) this.currentSETTING.r18 = checked;
        if (type === 2) this.currentSETTING.r18g = checked;
      }
    },
    calcCacheSize() {
      this.size.local = LocalStorage.size;
      this.size.session = SessionStorage.size;
    },
    clearCache(type) {
      let showName;
      switch (type) {
        case "local":
          showName = "Local Cache";
          break;

        case "session":
          showName = "Session Cache";
          break;

        default:
          break;
      }
      Dialog.confirm({
        message: `Are you sure you want to clean up ${showName}? This will reload the content from the network`,
        confirmButtonColor: "black",
        cancelButtonColor: "#1989fa",
        closeOnPopstate: true
      }).then(() => {
        if (type === "local") LocalStorage.clear();
        if (type === "session") SessionStorage.clear();

        this.calcCacheSize();
        this.$toast.success("Completed Cleanup");
      });
    },
    ...mapActions(["saveSETTING"])
  },
  filters: {
    bytes(bytes) {
      bytes = Number(bytes);
      if (bytes === 0) return "0 B";

      const k = 1024;
      const dm = 0;
      const sizes = ["B", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];

      const i = Math.floor(Math.log(bytes) / Math.log(k));

      return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + " " + sizes[i];
    }
  },
  mounted() {
    this.currentSETTING = JSON.parse(JSON.stringify(this.SETTING));
    this.calcCacheSize();
  },
  updated() {
    this.saveSETTING(JSON.parse(JSON.stringify(this.currentSETTING)));
  },
  components: {
    [Cell.name]: Cell,
    [Button.name]: Button,
    [Switch.name]: Switch
  }
};
</script>

<style lang="stylus" scoped>
.setting {
}
</style>
