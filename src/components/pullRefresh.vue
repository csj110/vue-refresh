<template>
  <div class="pull-refresh" ref="pullF" :style="{height: pullHeight}">
    <div class="pull-info" :style="{height: tipHeight}" v-if="!refreshing">
      <img
        src="../img/refresh_arrow.png"
        :style="{transform:`rotate(${arrowDeg}deg)`,transition:transition}"
      >
      <span>{{pullTip}}</span>
    </div>
    <div class="pull-info" :style="{height: tipHeight}" v-else>
      <img src="../img/loading.gif">
      <span>{{refreshTip}}</span>
    </div>
    <div
      class="pull-con"
      ref="pull"
      @touchstart="touchstart"
      @touchmove="touchmove"
      @touchend="touchend"
      :style="{transform:`translateY(${moveY}px)`,transition:transition}"
    >
      <slot></slot>
      <div class="pull-up-info" :style="{height: tipHeight,}" v-if="loading">
        <img src="../img/loading.gif">
        <span>{{loadingTip}}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "pullRefresh",
  props: {
    refreshing: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: true
    },
    onRefresh: {
      type: Function,
      default: function() {}
    },
    onLoading: {
      type: Function,
      default: function() {}
    },
    tipHeight: {
      type: String,
      default: "50px"
    },
    pullHeight: {
      type: String,
      default: "100vh"
    },
    pullTip: {
      type: String,
      default: "下拉即可刷新"
    },
    loadingTip: {
      type: String,
      default: "正在加载"
    },
    refreshTip: {
      type: String,
      default: "正在刷新"
    }
  },
  data() {
    return {
      clientY: 0,
      moveY: 0,
      disabled: false,
      LoadDisabled: false,
      arrowDeg: 0,
      transition: "all 0.2s",
      loading: false
    };
  },
  computed: {
    getPullInfoHeight: function() {
      return parseFloat(
        getComputedStyle(document.querySelector(".pull-info")).height
      );
    }
  },
  methods: {
    touchstart(e) {
      if (this.$refs.pullF.scrollTop == 0 || (this.$refs.pullF.scrollTop+this.$refs.pullF.windowHeight==this.$refs.pullF.scrollHeight)) {
        this.transition = "all 0s";
        this.clientY = e.touches[0].clientY;
      }else {
        this.disabled = true
      }
    },
    touchmove(e) {
      let moveY = e.touches[0].clientY - this.clientY;
      const infoHeight = this.getPullInfoHeight;
      const maxRange = infoHeight * 2;
      if (!this.disabled && moveY > 0) {
        e.preventDefault();
        this.moveY = moveY;
        this.moveY >= maxRange && (this.moveY = maxRange);
        this.arrowDeg =
          this.moveY < infoHeight
            ? 0
            : this.moveY < maxRange
            ? ((this.moveY - infoHeight) / infoHeight) * 180
            : 180;
      };
      if(!this.disabled && moveY < 0){
        e.preventDefault();
        this.moveY = moveY;
        -1*this.moveY >= maxRange && (this.moveY = -1*maxRange);
      }
    },
    touchend() {
      const infoHeight = this.getPullInfoHeight;
      if (this.moveY >= infoHeight) {
        this.moveY = infoHeight;
        this.disabled = true;
        this.transition = "all 0.2s";
        this.onRefresh && this.onRefresh();
      } else {
        this.moveY = 0;
        this.disabled = false;
      }
    }
  },
  watch: {
    refreshing: function(nv, ov) {
      if (!nv && ov) {
        this.clientY = 0;
        this.moveY = 0;
        this.disabled = false;
        // this.arrowDeg = 0;
        this.transition = "all 0.2s";
      } else if (nv && !ov) {
        this.moveY = this.getPullInfoHeight;
        this.disabled = true;
        // this.arrowDeg = 180;
      }
    },
    loading: function(nv, ov) {
      if (!nv && ov) {
        this.clientY = 0;
        this.moveY = 0;
        this.disabled = false;
        // this.arrowDeg = 0;
        this.transition = "all 0.2s";
      } else if (nv && !ov) {
        this.moveY = this.getPullInfoHeight;
        this.disabled = true;
        // this.arrowDeg = 180;
      }
    }
  }
};
</script>

<style scoped>
.pull-refresh {
  position: relative;
  overflow-y: auto;
  height: 100vh;
}
.pull-con {
  display: relative;
}
.pull-info {
  display: flex;
  display: -webkit-flex;
  justify-content: center;
  -webkit-justify-content: center;
  align-items: center;
  -webkit-align-items: center;
  position: absolute;
  top: 0;
  width: 100%;
}
.pull-up-info {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  bottom: 0;
  width: 100%;
  height: 50px;
}
.pull-info img,
.pull-up-info img {
  width: 20px;
  height: 20px;
  margin-right: 10px;
}
.pull-info span,
.pull-up-info span {
  color: #333333;
  font-size: 14px;
}
</style>
