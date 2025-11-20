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
        naviAnalyser: null, // 导航分析器
        searchAnalyser: null, // 搜索分析器 [新增]
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
          mapID: '1991048910850551809',
          mapThemeURL: 'https://lib.fengmap.com/theme/2001',
          defaultThemeName: '2001',
          modelSelectedEffect: false
        };

        window.map = new fengmap.FMMap(mapOptions);

        window.map.on('loaded', () => {
          console.log('地图加载完成');
          this.initNavigation();
          this.initSearch();

          window.executeRealNavigation = this.handleNavigationRequest; 
        });

        window.map.on('click', (event) => {
          console.log(event)
          const target = event.targets && event.targets[0];
          if (target) {
            const roomName = target.name;
            // 坐标数据
            const clickCoord = {
              x: event.coords.x,
              y: event.coords.y,
              groupID: event.level,
              name: roomName
            };
            console.log("选中地点:", roomName);
            
            // [关键] 如果UI层定义了回调，就调用它来填充输入框
            if (window.updateEndInput && roomName) {
              window.updateEndInput(roomName, clickCoord); 
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

      initNavigation() {
        if (!fengmap.FMNaviWalkAnalyser) {
          console.error("未找到 fengmap.FMNaviWalkAnalyser");
          return;
        }

        console.log("开始初始化导航分析器...");
        
        this.naviPromise = new Promise((resolve) => {
          // 【修改点 1】先定义一个变量接收 new 的结果
          const _analyserInstance = new fengmap.FMNaviWalkAnalyser({
            map: window.map
          }, () => {
            // 回调函数执行时，说明初始化好了
            console.log("导航分析器初始化成功！");
            
            // 【修改点 2】直接使用外部定义的 _analyserInstance 变量
            window.naviAnalyser = _analyserInstance;
            
            // 【修改点 3】把这个确定的实例 resolve 出去
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

        // 1. 解析起点
        let p1 = typeof startVal === 'string' ? await this.searchLocation(startVal) : startVal;
        // 2. 解析终点
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
          
          // 【修改点1】使用官方推荐的 Request 对象构造方式，避免属性拼写错误
          var request = new fengmap.FMSearchRequest();
          
          // 【修改点2】在 v3.0 SDK 中，必须使用 fengmap.FMType.MODEL，而不是 FMSearchType
          request.type = fengmap.FMType.MODEL; 
          
          request.keyword = keyword;

          this.searchAnalyser.query(request, (result) => {
            // 获取第一个匹配结果
            if (result && result.length > 0) {
              const target = result[0];
              // 返回标准坐标对象
              resolve({
                // 注意：不同版本返回的坐标属性可能不同，v3通常是 mapCoord 或直接在对象根目录下
                // 建议先打印一下 target 看看结构
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
            level: p1.groupID, // 确保这里有值，不是 undefined
            url: './img/start.png', 
            size: 32 
          },
          dest: { 
            x: p2.x, 
            y: p2.y, 
            level: p2.groupID, // 确保这里有值，不是 undefined
            url: './img/end.png', 
            size: 32 
          },
          mode: fengmap.FMNaviMode.MODULE_SHORTEST
        };

        console.log("发起路径规划请求:", request);

        // 【关键修改】添加第二个参数 (error) => { ... }
        window.naviAnalyser.route(
          request, 
          // 1. 成功回调
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
          // 2. 失败回调 (之前缺这个，所以没反应)
          (error) => {
            console.error("路径规划失败详情:", error);
            // error 可能是对象也可能是字符串，打印出来看
            alert("路径规划失败，请看控制台红色报错");
          }
        );
      },

      drawRouteLine(route) {
        console.log(">>> 规划成功，原始路径数据:", route);

        const segments = [];
        
        // 遍历所有路段
        route.subs.forEach(leg => {
          // 【修改点1】建议暂时去掉这个 if 判断，防止意外过滤掉有效路段
          // 除非你非常确定只画同层路径，否则先注释掉它，看看能不能画出来
          // if (leg.levels[0] === leg.levels[1]) { 
            
            let segment = new fengmap.FMSegment();
            
            // 【修改点2 - 核心】手动给所有点增加 Z 轴高度
            // 官方示例中明确指出需要设置 point.z，否则线会被地板遮挡
            const pointsWithHeight = leg.waypoint.points.map(p => {
              return {
                x: p.x,
                y: p.y,
                z: 1.5 // 抬高 1.5 米，确保浮在地板之上
              };
            });

            segment.points = pointsWithHeight;
            
            // 这里的 level 决定了线在切换楼层时是否显示
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
          // height: 2 // 【修改点3】建议去掉这个属性，改用上面点的 z 值控制，这样更稳定
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