<template>
  <div class="toolBar">
    <!-- <div id="displayBar" class="icon" @click="switchDisplay">
      <img src="~@/assets/img/2D.png" alt="" v-show="isActive">
      <img src="~@/assets/img/3D.png" alt="" v-show="!isActive">
    </div> -->
    <div id="arBar" class="icon" @click="switchAr">
      <img src="~@/assets/img/AR.png" alt="">
    </div>
    <div id="navigationBar" class="icon" @click="switchNav">
      <img src="~@/assets/img/path.png" alt="">
    </div>
    <div></div>
  </div>
</template>

<script>
  export default {
    name: "ToolBar",
    data() {
      return {
        isActive: false 
      }
    },
    methods: {
      switchDisplay() {
        if (!window.map || !window.fengmap) {
          console.warn("地图尚未初始化");
          return;
        }

        this.isActive = !this.isActive;
        const mode = this.isActive ? fengmap.FMViewMode.MODE_3D : fengmap.FMViewMode.MODE_2D;

        console.log("切换视图模式:", this.isActive ? "3D" : "2D");
        window.map.setViewMode({
            mode: mode,
            animate: true
        });
      },

      switchNav() {
        this.$store.commit('switchNav')
        const arMod = document.getElementById("webARModule");
        if(arMod) arMod.style.zIndex = "0";
      },
      switchAr() {
        this.$store.commit("switchArComponent")
      }
    }
  }
</script>

<style scoped>
  .toolBar{
    padding-top: 15px;
    padding-bottom: 15px;
    border-radius: 15px;
    background-color: #fff;
    position: fixed;
    top:350px;
    left: 15px;
    box-shadow: 1px 1px 1px #d4d4d4;
  }
  .icon{
    width: 36px;
    height: 36px;
    background-color: #fff;
    color: #fff;
    text-align: center;
    line-height: 26px;
    padding: 5px;
    box-shadow: 0 0 1px #d2d2d2;
  }
  .icon img{
    width: 28px;
    height: 28px;
  }
</style>