<template>
  <div class="navigation" v-show="$store.state.isNavShow">
    <button @click="simulate">模拟导航</button>
    <button @click="arSimulate">开始导航</button>
  </div>
</template>

<script>
  export default {
    name: "Navigation",
    methods: {
      simulate() {
        let arCoordinates = this.$store.state.coordinates;
        if (!arCoordinates || arCoordinates.length === 0) {
          alert("没有路径数据！请先在控制台运行 window.autoNavigate() 生成路径。");
          return;
        }

        console.log("开始模拟导航...");
        
        let coordsClone = arCoordinates.concat();
        let length = coordsClone.length;

        for (let i = 0; i < length; i++) {
          (function (currentCoords) {
            setTimeout(function () {
              if (window.ar && window.ar.redrawAr) {
                window.ar.redrawAr(currentCoords);
                console.log("AR 模拟步进:", i);
              }
            }, 1000 * i);
          })(coordsClone.concat());
          
          coordsClone.shift();
        }
      },
      
      arSimulate() {
        console.log("当前导航路径:", this.$store.state.coordinates);
        this.$store.commit("switchArComponent");
      }
    }
  }
</script>

<style scoped>
  .navigation {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    text-align: center;
    margin: 10px;
    display: flex;
    z-index: 999;
  }

  .navigation button {
    height: 36px;
    font-size: 18px;
    font-weight: 500;
    border-radius: 10px;
    border: 1px solid #d0d0d0;
    outline: none;
  }

  .navigation button:nth-child(1) {
    flex: 3;
    background-color: #f7f7f7;
    color: #2a2a2a;
    margin-left: 20px;
    margin-right: 20px;
  }

  .navigation button:nth-child(2) {
    flex: 5;
    background-color: #4187ff;
    color: #fff;
    margin-right: 20px;
  }
</style>