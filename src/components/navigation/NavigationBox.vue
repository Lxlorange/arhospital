<template>
  <div class="navigation" v-if="$store.state.isNavBoxShow">
    <div class="top">
      <div class="flex" :class="{ active: focusType === 'start' }" @click="setFocus('start')">
        <IconItem class="green">起</IconItem>
        <input 
          type="text" 
          v-model="startName" 
          @focus="setFocus('start')" 
          @input="startCoord = null" 
          placeholder="请输入或点击地图选点"
        >
      </div>
      
      <div class="flex" :class="{ active: focusType === 'end' }" @click="setFocus('end')">
        <IconItem class="red">终</IconItem>
        <input 
          type="text" 
          v-model="endName" 
          @focus="setFocus('end')" 
          @input="endCoord = null" 
          placeholder="请输入或点击地图选点"
        >
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
    components: { IconItem },
    data() {
      return {
        startName: "挂号大厅", 
        endName: "",           
        startCoord: {
           x: 12952371.703681946, 
           y: 4832045.853654381,
           groupID: 1,
           name: "挂号大厅"
        }, 
        endCoord: null,
        focusType: 'end'
      }
    },
    mounted() {
      window.updateNavSelection = (name, coord) => {
        if (this.focusType === 'start') {
          this.startName = name;
          this.startCoord = coord;
          console.log("已设置起点:", name);
          this.focusType = 'end'; 
        } else {
          this.endName = name;
          this.endCoord = coord;
          console.log("已设置终点:", name);
        }
      };
      const params = new URLSearchParams(window.location.search);
      const destination = params.get('dest');
      
      if (destination) {
        this.endName = destination;
        this.focusType = 'start';
      }
    },
    beforeDestroy() {
      window.updateNavSelection = null;
    },
    methods: {
      setFocus(type) {
        this.focusType = type;
        console.log("当前选点模式:", type === 'start' ? "选起点" : "选终点");
      },

      confirmClick() {
        if (typeof window.executeRealNavigation === 'function') {
          const p1 = this.startCoord ? this.startCoord : this.startName;
          const p2 = this.endCoord ? this.endCoord : this.endName;

          if(!p1 || !p2) {
             alert("请输入起点和终点");
             return;
          }

          window.executeRealNavigation(p1, p2);
          
          // this.$store.commit('switchNavBox');
          // this.$store.commit('switchNavButton');
        } else {
          alert("地图尚未加载完成");
        }
      },
      cancelClick() {
        this.$store.commit('switchNavBox');
        this.endName = "";
        this.endCoord = null;
        this.startCoord = null;
        // this.startName = ""; 
      }
    }
  }
</script>

<style scoped>
  .navigation {
    position: fixed;
    left: 0; right: 0; bottom: 0;
    background-color: #ffffff;
    z-index: 999;
    border-top-left-radius: 20px;
    border-top-right-radius: 20px;
    box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
  }
  .flex {
    display: flex;
    margin: 15px 30px;
    align-items: center;
    padding: 5px;
    border-radius: 10px;
    transition: background-color 0.3s;
  }

  .flex.active {
    background-color: #e6f0ff;
    border: 1px solid #4187ff;
  }

  .flex .green { background-color: var(--color-green); }
  .flex .red { background-color: var(--color-red); }

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
  
  .bottom { display: flex; justify-content: center; padding-bottom: 20px; }
  .bottom button { width: 130px; height: 40px; font-size: 18px; color: #fff; border-radius: 20px; border: none; margin: 0 15px; }
  .bottom button:nth-child(1) { background-color: #4187ff; }
  .bottom button:nth-child(2) { background-color: #e0e0e0; color: #4e4e4e; }
</style>