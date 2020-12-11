<template>
  <div style="width: 100%; height: 100%">
    <div ref="earthContainer" style="width: 100%; height: 100%"></div>
    <div style="position: absolute; left: 18px; top: 18px">
      <button>
        {{ message }} 相机位置(经度/纬度/高度)：{{
          position.map((e, i) => e.toFixed(2)).join("/")
        }}
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";

@Options({
  props: {
    message: String,
    imageAlpha: Number,
    position: [Number, Number, Number],
  },
})

export default class EarthComp extends Vue {
  message: string  = "页面加载于 " + new Date().toLocaleString();
  _earth: any = undefined; // 注意：Earth和Cesium的相关变量放在vue中，必须使用下划线作为前缀！
  imageAlpha: number = 1.0;
  position: [number, number, number] = [0, 0, 0];

  // 1.1 资源创建
  mounted() {
    // 1.1.1 创建地球
    // @ts-ignore
    var earth = new XE.Earth(this.$refs.earthContainer);

    // 1.1.2 添加默认地球影像
    earth.sceneTree.root = {
      children: [
        {
          czmObject: {
            name: "默认离线影像",
            xbsjType: "Imagery",
            xbsjImageryProvider: {
              createTileMapServiceImageryProvider: {
                // @ts-ignore
                url: XE.HTML.cesiumDir + "Assets/Textures/NaturalEarthII",
                fileExtension: "jpg",
              },
              type: "createTileMapServiceImageryProvider",
            },
          },
        },
      ],
    };

    this._earth = earth;

    // @ts-ignore
    XE.MVVM.bindPosition(this, "position", earth.camera, "position");

    // @ts-ignore
    // 仅为调试方便用
    window.earth = earth;
  }

  // 1.2 资源销毁
  beforeDestroy() {
    // vue程序销毁时，需要清理相关资源
    this._earth = this._earth && this._earth.destroy();
  }
}
</script>