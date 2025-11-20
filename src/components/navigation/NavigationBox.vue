<template>
  <div class="navigation" v-if="$store.state.isNavBoxShow">
    <div class="top">
      <div class="flex">
        <IconItem class="green">起</IconItem>
        <input type="text" id="startInput" value="挂号大厅">
      </div>
      <div class="flex">
        <IconItem class="red">终</IconItem>
        <input type="text" id="endInput" value="">
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
    methods: {
      confirmClick() {
        if (typeof window.autoNavigate === 'function') {
          console.log("调用全局演示导航...");
          window.autoNavigate(); 
          
          this.$store.commit('switchNavBox');
          this.$store.commit('switchNavButton');
        } else {
          alert("地图尚未加载完成，请稍后再试");
        }
      },
      
      cancelClick() {
        this.$store.commit('switchNavBox');
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