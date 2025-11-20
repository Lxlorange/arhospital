<template>
  <div>
    <div id="webARModule" ref="Ar" v-show="$store.state.arComponentShow">
      <video id="ARModuleCameraVideo" autoplay="autoplay"></video>
      <canvas id="webGL3d" style="position: absolute;"></canvas>
      <div id="landmark"></div>
      <div id="compassLine"></div>
    </div>
    <div id='mapContainer'></div>
  </div>
</template>

<script>
  import VConsole from "vconsole"
  import openCamera from "./openCamera"

  export default {
    name: "IndoorMap",
    data() {
      return {
        map: null,
        naviAnalyser: null, // v3.0 路径分析器
        currentPosition: null
      }
    },
    created() {
      new VConsole()
    },
    mounted() {
      // 确保 fengmap 对象存在再初始化
      if (window.fengmap) {
        this.mapCreate()
      } else {
        alert("SDK加载失败，请检查 public/lib 下的文件是否完整")
      }
    },
    watch: {
      '$store.state.arComponentShow'(isShown) {
        if (isShown) openCamera();
      }
    },
    methods: {
      mapCreate() {
        const mapOptions = {
          container: document.getElementById('mapContainer'),
          appName: 'HospitalNav',
          key: 'e9ba251d9b1d897f99133b970b50650b',
          
          mapServerURL: './data/map/1991048910850551809', 
          
          mapThemeURL: 'https://lib.fengmap.com/theme/2001',
          defaultThemeName: '2001',
          
          modelSelectedEffect: false
        };

        window.map = new fengmap.FMMap(mapOptions);

        window.map.openMap({
          id: '1991048910850551809',
          error: (e) => {
            console.error(e);
            alert('地图打开失败，请检查控制台报错');
          }
        });

        window.map.on('loadComplete', () => {
          console.log('✅ 地图加载完成 (v3.0)');
          
          this.initNavigation();
          
          this.createControls();
          
          window.autoNavigate = this.testRoute; 
        });
        
        window.map.on('click', (e) => {
           console.log("点击坐标:", e.coords);
        });
      },

      initNavigation() {
        if (!fengmap.FMNaviAnalyser) return;

        window.naviAnalyser = new fengmap.FMNaviAnalyser({
          map: window.map
        });
        
        console.log("导航分析器已就绪");
      },
      
      createControls() {
        // 楼层控件
        new fengmap.FMToolbar({
           mode: '2d', 
           position: fengmap.FMControlPosition.RIGHT_TOP,
           offset: { x: 10, y: 100 }
        }).addTo(window.map);
      },

      calculateRoute(p1, p2) {
        if (!window.naviAnalyser) return;

        const request = {
          start: { x: p1.x, y: p1.y, groupID: p1.groupID, url: './img/start.png', size: 32 },
          end:   { x: p2.x, y: p2.y, groupID: p2.groupID, url: './img/end.png', size: 32 },
          mode: fengmap.FMNaviMode.MODULE_SHORTEST
        };

        // 计算路径
        const result = window.naviAnalyser.analyse(request);

        if (result && result.subs && result.subs.length > 0) {
          console.log("🚀 路径计算成功", result);
          
          // 提取坐标点给 AR 模块
          // 点集在 result.subs[0].points
          const routePoints = result.subs[0].points;
          
          this.$store.commit("getCoordinates", routePoints);
          
          alert("导航开始！请点击界面上的'模拟导航'按钮");
        } else {
          alert("路径计算失败，请确认路网是否连通");
        }
      },

      testRoute() {
         // 随便找两个点
         const c = window.map.center;
         const gid = window.map.focusGroupID;
         this.calculateRoute(
           { x: c.x - 5, y: c.y, groupID: gid }, 
           { x: c.x + 5, y: c.y, groupID: gid }
         );
      }
    }
  }
</script>

<style scoped>
  #mapContainer { width: 100vw; height: 100vh; background-color: #eee; }
  #webARModule { position: fixed; top: 0; left: 0; z-index: 999; }
  #ARModuleCameraVideo { position: absolute; width: 100vw; height: 100vh; object-fit: cover; }
  #webGL3d { position: absolute; width: 100vw; height: 100vh; pointer-events: none; }
  #landmark { position: fixed; width: 100vw; height: 100vh; pointer-events: none; }
  #compassLine { position: fixed; width: 100vw; bottom: 50px; overflow: hidden; }
</style>