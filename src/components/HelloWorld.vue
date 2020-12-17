<template>
  <div class="hello">
    <div class="container">
      <canvas id="board"></canvas>
      <canvas id="background"></canvas>
    </div>
    <div class="options">
      <ul>
        <li :key="item.imgSrc" @click="addImg(item.imgSrc)" v-for="item in fundamental">
          <img :src="item.imgSrc" alt="" />{{ item.name }}
        </li>
      </ul>
    </div>
    <div class="buttons">
      <button @click="clean">清除</button>
      <button @click="copy">複製</button>
      <button @click="remove">刪除</button>
      <button @click="rotateLeft">向左旋轉45度</button>
      <button @click="rotateRight">向右旋轉45度</button>
      <button @click="save">儲存我的圖片</button>
    </div>
  </div>
</template>

<script>
import { fabric } from "fabric";

export default {
  name: "HelloWorld",
  data() {
    return {
      boardInfo: {
        width: 641,
        height: 461,
      },
      fundamental: [
        {
          name: "牆",
          imgSrc: require("../assets/wall.svg"),
        },
        {
          name: "陽台",
          imgSrc: require("../assets/balcony.svg"),
        },
        {
          name: "弧形",
          imgSrc: require("../assets/arc.svg"),
        },
        {
          name: "門",
          imgSrc: require("../assets/door.svg"),
        },
        {
          name: "窗",
          imgSrc: require("../assets/window.svg"),
        },
        {
          name: "開口",
          imgSrc: require("../assets/interval.svg"),
        },
      ],
    };
  },
  props: {
    msg: String,
  },
  setup(props) {
    console.log(props.title)
  },
  mounted() {
    // console.log(this.board)
    this.board = new fabric.Canvas("board", {
      width: this.boardInfo.width,
      height: this.boardInfo.height,
    });
    console.log(this)
    this.initBackgroundBoard();
  },
  methods: {
    initBackgroundBoard() {
      let _self = this;

      let background = new fabric.Canvas("background", {
        width: _self.boardInfo.width,
        height: _self.boardInfo.height,
      });

      let dotlinesLatitude = [],
        dotlinesLongitude = [];

      for (let i = 0; i < _self.boardInfo.height; i = i + 10) {
        dotlinesLatitude.push(
          new fabric.Line([0, i, _self.boardInfo.width, i], {
            strokeDashArray: [1],
            stroke: "black",
          })
        );
      }

      for (let j = 0; j < _self.boardInfo.width; j = j + 10) {
        dotlinesLongitude.push(
          new fabric.Line([j, 0, j, _self.boardInfo.height], {
            strokeDashArray: [1],
            stroke: "black",
            selectable: false,
          })
        );
      }

      background.add(...dotlinesLatitude);
      background.add(...dotlinesLongitude);
    },
    addImg(url) {
      let _self = this
      fabric.Image.fromURL(url, (oImg) => {
        _self.board.add(oImg);
      });
    },
    clean() {
      this.board.clear();
    },
    copy() {
      this.board.getActiveObject().clone((cloned) => {
          cloned.clone((clonedObj)=>{
          this.board.discardActiveObject();
          clonedObj.set({
            left: clonedObj.left + 50,
            top: clonedObj.top + 50,
            evented: true,
          });
          if (clonedObj.type === 'activeSelection') {
            // active selection needs a reference to the canvas.
            clonedObj.canvas = this.board;
            clonedObj.forEachObject((obj) => {
              this.board.add(obj);
            });
            // this should solve the unselectability
            clonedObj.setCoords();
          } else {
            this.board.add(clonedObj);
          }
          cloned.top += 50;
          cloned.left += 50;
          this.board.setActiveObject(clonedObj);
          this.board.requestRenderAll();
        })
      });
    },
    remove() {
      let delElement =
        this.board.getActiveObject()._objects !== undefined
          ? this.board.getActiveObject()._objects
          : [this.board.getActiveObject()];
      delElement.forEach((element) => {
        this.board.remove(element);
      });
      this.board.discardActiveObject();
      this.board.requestRenderAll();
    },
    rotateLeft() {
      let currentElement = this.board.getActiveObject(),
        angle = currentElement.angle;

      currentElement.rotate(angle + 45);
      this.board.requestRenderAll();
    },
    rotateRight() {
      let currentElement = this.board.getActiveObject(),
        angle = currentElement.angle;
      currentElement.rotate(angle - 45);
      this.board.requestRenderAll();
    },
    save() {},
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
.hello {
  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }
  .container {
    position: relative;
    width: 641px;
    height: 461px;
    .canvas-container:nth-child(1) {
      z-index: 1;
      position: absolute !important;
    }
    .canvas-container:nth-child(2) {
      z-index: 0;
      position: absolute !important;
    }
  }
}
</style>
