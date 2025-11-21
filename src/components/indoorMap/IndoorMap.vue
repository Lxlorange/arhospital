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
        naviAnalyser: null,
        searchAnalyser: null,
        currentPosition: null
      }
    },
    created() {
      new VConsole()
    },
    mounted() {
      if (window.fengmap) {
        this.mapCreate()
      } else {
        alert("SDK加载失败")
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
          key: 'ddc2639ce5ff01d57e9d4a61f8ea67a5',
          // mapURL: './data/1991048910850551809',
          mapID: '1991048910850551809',
          mapThemeURL: 'https://lib.fengmap.com/theme/2001',
          defaultThemeName: '2001',
          modelSelectedEffect: false,
          floorSpace: 5
        };

        window.map = new fengmap.FMMap(mapOptions);

        window.map.on('loaded', () => {
          console.log('地图加载完成');
          if (window.map.bound) {
            window.map.setFitView(window.map.bound, {
              animate: true,
              duration: 0.1,
              finish: () => {
                console.log("视角自动适配完成");
                // window.map.zoomIn(); 
              }
            });
          }
          console.log(window.map.getFloorSpace())
          this.addFloorControl();
          this.initNavigation();
          this.initSearch();

          window.executeRealNavigation = this.handleNavigationRequest; 
        });

        window.map.on('click', (event) => {
          console.log(event)
          const target = event.targets && event.targets[0];
          if (target) {
            const roomName = target.name;
            const clickCoord = {
              x: event.coords.x,
              y: event.coords.y,
              groupID: event.level,
              name: roomName
            };
            console.log("选中地点:", roomName);
            
            if (window.updateNavSelection && roomName) {
              window.updateNavSelection(roomName, clickCoord); 
            }
          }
        });
      },

      initSearch() {
        if (!fengmap.FMSearchAnalyser) return;
          this.searchAnalyser = new fengmap.FMSearchAnalyser({
            map: window.map
          });
        },

      addFloorControl() {
        if (fengmap.FMToolbar) {
            var scrollFloorCtlOpt = {
              position: fengmap.FMControlPosition.LEFT_TOP,
              offset: { x: 15, y: 80 }, 
              floorModeControl: true,
              needAllLayerBtn: true,
              viewModeControl: true, 
              floorButtonCount: 5 
          };
          
          var scrollFloorControl = new fengmap.FMToolbar(scrollFloorCtlOpt);
          scrollFloorControl.addTo(window.map);

          var zoomBar = new fengmap.FMZoomBar({
                position: fengmap.FMControlPosition.RIGHT_TOP,
                offset: {
                    x: -10,
                    y: 20
                },
            });

        zoomBar.addTo(window.map);
          console.log("楼层控件加载成功");
        } else {
          console.error("未找到 fengmap.FMToolbar，请检查 index.html 是否引入了 UI 插件");
        }
      },

      initNavigation() {
        if (!fengmap.FMNaviWalkAnalyser) {
          console.error("未找到 fengmap.FMNaviWalkAnalyser");
          return;
        }

        console.log("开始初始化导航分析器...");
        
        this.naviPromise = new Promise((resolve) => {
          const _analyserInstance = new fengmap.FMNaviWalkAnalyser({
            map: window.map,
            // threshold: 100  
          }, () => {
            console.log("导航分析器初始化成功！");
            window.naviAnalyser = _analyserInstance;
            resolve(_analyserInstance);
          });
        });
      },

    async handleNavigationRequest(startVal, endVal) {
      if (!this.naviPromise) {
        alert("导航功能无法启动：缺少导航插件脚本");
        return;
      }

      try {
        const analyser = await this.naviPromise; 

        if (!analyser) {
          alert("导航模块初始化失败");
          return;
        }

        let p1 = typeof startVal === 'string' ? await this.searchLocation(startVal) : startVal;
        let p2 = typeof endVal === 'string' ? await this.searchLocation(endVal) : endVal;

        if (!p1 || !p2) {
          alert("无法找到起点或终点位置");
          return;
        }

        this.calculateRoute(p1, p2);

      } catch (e) {
        console.error("导航流程异常:", e);
        alert("路径规划出错，请查看控制台");
      }
    },

      searchLocation(keyword) {
        return new Promise((resolve, reject) => {
          if (!this.searchAnalyser) return resolve(null);
          
          var request = new fengmap.FMSearchRequest();
          
          request.type = fengmap.FMType.MODEL; 
          
          request.keyword = keyword;

          this.searchAnalyser.query(request, (result) => {
            if (result && result.length > 0) {
              const target = result[0];
              resolve({
                x: target.mapCoord ? target.mapCoord.x : target.x, 
                y: target.mapCoord ? target.mapCoord.y : target.y,
                groupID: target.groupID,
                name: target.name
              });
            } else {
              console.warn("未找到地点:", keyword);
              resolve(null);
            }
          });
        });
      },

      calculateRoute(p1, p2) {
        const request = {
          start: { 
            x: p1.x, 
            y: p1.y, 
            level: p1.groupID,
            url: './img/start.png', 
            size: 32 
          },
          dest: { 
            x: p2.x, 
            y: p2.y, 
            level: p2.groupID,
            url: './img/end.png', 
            size: 32 
          },
          mode: fengmap.FMNaviMode.MODULE_SHORTEST,
          priority: fengmap.FMNaviPriority.PRIORITY_DEFAULT
        };

        console.log("发起路径规划请求:", request);

        window.naviAnalyser.route(
          request, 
          (result) => {
            console.log("开始画线", result);
            if (result && result.subs && result.subs.length > 0) {
               this.drawRouteLine(result);
               const routePoints = result.subs[0].points;
               this.$store.commit("getCoordinates", routePoints);
            } else {
               alert("路径计算结果为空");
            }
          },
          (error) => {
            console.error("路径规划失败详情:", error);
            alert("路径规划失败，请看控制台红色报错");
          }
        );
      },

      drawRouteLine(route) {
        console.log(">>> 规划成功，原始路径数据:", route);

        const segments = [];
        
        route.subs.forEach(leg => {
            
            let segment = new fengmap.FMSegment();
            
            const pointsWithHeight = leg.waypoint.points.map(p => {
              return {
                x: p.x,
                y: p.y,
                z: 1.5 // 抬高 1.5 米，确保浮在地板之上
              };
            });

            segment.points = pointsWithHeight;
            
            segment.level = leg.levels[0]; 
            
            segments.push(segment);
          // }
        });
        
        console.log(">>> 处理后的线段数据(segments):", segments);

        if (segments.length === 0) {
          alert("路径数据生成为空！可能是被过滤条件筛掉了");
          return;
        }
        
        if (this.naviLine) this.naviLine.remove();

        this.naviLine = new fengmap.FMLineMarker({
          segments: segments,
          color: '#4187ff', 
          width: 6,
        });
        
        this.naviLine.addTo(window.map);
      },
      
    }
  }
</script>

<style scoped>
  #mapContainer { width: 100vw; height: 100vh; background-color: #eee; }
  #webARModule { position: fixed; top: 0; left: 0; z-index: 999; pointer-events: none; }
  #ARModuleCameraVideo { position: absolute; width: 100vw; height: 100vh; object-fit: cover; }
  #webGL3d { position: absolute; width: 100vw; height: 100vh; pointer-events: none; }
  #landmark { position: fixed; width: 100vw; height: 100vh; pointer-events: none; }
  #compassLine { position: fixed; width: 100vw; bottom: 50px; overflow: hidden; }
</style>