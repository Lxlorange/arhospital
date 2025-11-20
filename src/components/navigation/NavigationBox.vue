<template>
  <div class="navigation" v-if="$store.state.isNavBoxShow">
    <div class="top">
      <div class="flex">
        <IconItem class="green">起</IconItem>
        <input type="text" v-model="startName" placeholder="请输入起点">
      </div>
      <div class="flex">
        <IconItem class="red">终</IconItem>
        <input type="text" v-model="endName" placeholder="请输入或点击地图">
      </div>
    </div>
    <div class="bottom">
      <button @click="confirmClick">开始导航</button>
      <button @click="cancelClick">取消</button>
    </div>
  </div>
</template>

<script>
  import IconItem from "./IconItem"

  export default {
    name: "NavigationBox",
    components: {
      IconItem
    },
    data() {
      return {
        startName: "挂号大厅", // 默认起点
        endName: "",           // 终点
        startCoord: null,      // 如果是点击获取的，存坐标对象
        endCoord: null         // 如果是点击获取的，存坐标对象
      }
    },
    mounted() {
      // [关键] 注册全局回调，让 IndoorMap 点击事件能更新这里
      window.updateEndInput = (name, coord) => {
        this.endName = name;
        this.endCoord = coord; // 保存精确坐标
      };
    },
    beforeDestroy() {
      window.updateEndInput = null;
    },
    methods: {
      confirmClick() {
        if (typeof window.executeRealNavigation === 'function') {
          console.log("开始规划路径:", this.startName, "->", this.endName);
          
          const p1 = this.startCoord ? this.startCoord : this.startName;
          const p2 = this.endCoord ? this.endCoord : this.endName;

          // 调用 IndoorMap.vue 里的逻辑
          window.executeRealNavigation(p1, p2);
          
          // UI 交互处理
          this.$store.commit('switchNavBox');
          this.$store.commit('switchNavButton');
        } else {
          alert("地图尚未加载完成，请稍后再试");
        }
      },
      
      cancelClick() {
        this.$store.commit('switchNavBox');
        this.endName = "";
        this.endCoord = null;
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
    background-color: #ffffff;
    z-index: 999; /* 确保在最上层 */
    border-top-left-radius: 20px;
    border-top-right-radius: 20px;
    box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
  }

  .flex {
    display: flex;
    margin: 15px 30px;
    align-items: center;
  }

  .flex .green {
    background-color: var(--color-green);
  }

  .flex .red {
    background-color: var(--color-red);
  }

  .flex input {
    width: 80%;
    margin-left: 20px;
    border-radius: 30px;
    border: 1px solid var(--color-border);
    font-size: 16px;
    padding: 8px 20px;
    outline: none;
    background-color: #f5f5f5;
  }

  .bottom {
    display: flex;
    justify-content: center;
    padding-bottom: 20px;
  }

  .bottom button {
    width: 130px;
    height: 40px;
    font-size: 18px;
    color: #fff;
    font-weight: 500;
    border-radius: 20px;
    border: none;
    outline: none;
    margin: 0 15px;
  }

  .bottom button:nth-child(1) {
    background-color: #4187ff;
    box-shadow: 0 4px 10px rgba(65, 135, 255, 0.3);
  }

  .bottom button:nth-child(2) {
    background-color: #e0e0e0;
    color: #4e4e4e;
  }
</style>