<template>
  <div class="hello">
    <div class="container">
      <div class="left">
        <ul class="menu">
          <li @click="changeOptions(item.id)" :key="item.id" v-for="item in menu">
            {{ item.name }}
          </li>
        </ul>
        <ul v-if="currentOption === 'words'" class="options">
          <li class="option" :key="item.name" v-for="item in currentOptions">
            <div @click="addText(item)" class="option-text">
              {{ item }}
            </div>
          </li>
        </ul>
        <ul v-else class="options">
          <li class="option" :key="item.name" v-for="item in currentOptions">
            <div class="option-img">
              <img @click="addImg(item.imgSrc)" :src="item.imgSrc" alt="" />
            </div>
            {{ item.name }}
          </li>
        </ul>
      </div>
      <div class="right">
        <div class="buttons">
          <button @click="clean">清除</button>
          <button @click="copy">複製</button>
          <button @click="remove">刪除</button>
          <button @click="rotateLeft">向左旋轉45度</button>
          <button @click="rotateRight">向右旋轉45度</button>
          <button @click="save">儲存我的圖片</button>
        </div>
        <div class="canvas-container">
          <canvas id="board"></canvas>
          <canvas id="background"></canvas>
        </div>
      </div>
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
      currentOption: "fundamental",
      menu: [
        {
          name: "墻/骨架",
          id: "fundamental",
        },
        {
          name: "核心建築",
          id: "kernel",
        },
        {
          name: "文字",
          id: "words",
        },
      ],
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
      kernel: [
        {
          name: "馬桶",
          imgSrc: require("../assets/toilet.svg"),
        },
        {
          name: "洗手臺",
          imgSrc: require("../assets/sink.svg"),
        },
        {
          name: "浴盆",
          imgSrc: require("../assets/bathtub.svg"),
        },
        {
          name: "水盆",
          imgSrc: require("../assets/basin.svg"),
        },
        {
          name: "沙發",
          imgSrc: require("../assets/sofa.svg"),
        },
        {
          name: "桌子",
          imgSrc: require("../assets/table.svg"),
        },
        {
          name: "床",
          imgSrc: require("../assets/bed.svg"),
        },
        {
          name: "植物",
          imgSrc: require("../assets/plant.svg"),
        },
        {
          name: "洗衣機",
          imgSrc: require("../assets/washing-machine.svg"),
        },
        {
          name: "廚具",
          imgSrc: require("../assets/kitchenware.svg"),
        },
      ],
      words: [
        "主臥室",
        "臥室",
        "書房",
        "玄關",
        "客廳",
        "廚房",
        "餐廳",
        "陽台",
        "浴室",
        "和室",
        "樓梯",
        "庭院",
      ],
    };
  },
  props: {
    msg: String,
  },
  computed: {
    // a computed getter
    currentOptions() {
      // `this` points to the vm instance
      return this[this.currentOption]
    }
  },
  setup(props) {
    console.log(props.title);
  },
  mounted() {
    this.board = new fabric.Canvas("board", {
      width: this.boardInfo.width,
      height: this.boardInfo.height,
    });

    this.board.on("object:modified", (e) => {
      console.log(e);
      if (e.target.getBoundingRect().left < 0 && e.target.getBoundingRect().top < 0) {
        e.target.left = 0;
        e.target.top = 0;
        e.target.setCoords();
      } else if (
        e.target.getBoundingRect().left < 0 &&
        e.target.getBoundingRect().top >
          this.boardInfo.height - e.target.height * e.scaleX
      ) {
        e.target.left = 0;
        e.target.top = this.boardInfo.height - e.target.height * e.scaleY;
        e.target.setCoords();
      } else if (
        e.target.getBoundingRect().top < 0 &&
        e.target.getBoundingRect().left >
          this.boardInfo.width - e.target.width * e.scaleX
      ) {
        e.target.top = 0;
        e.target.left = this.boardInfo.width - e.target.width * e.scaleX;
        e.target.setCoords();
      } else if (
        e.target.getBoundingRect().left >
          this.boardInfo.width - e.target.width * e.scaleX &&
        e.target.getBoundingRect().top >
          this.boardInfo.height - e.target.height * e.scaleY
      ) {
        e.target.left = this.boardInfo.width - e.target.width * e.scaleX;
        e.target.top = this.boardInfo.height - e.target.height * e.scaleY;
        e.target.setCoords();
      } else if (e.target.getBoundingRect().left < 0) {
        e.target.left = 0;
        e.target.setCoords();
      } else if (e.target.getBoundingRect().top < 0) {
        e.target.top = 0;
        e.target.setCoords();
      } else if (
        e.target.getBoundingRect().left >
        this.boardInfo.width - e.target.width * e.scaleX
      ) {
        e.target.left = this.boardInfo.width - e.target.width * e.scaleX;
        e.target.setCoords();
      } else if (
        e.target.getBoundingRect().top >
        this.boardInfo.height - e.target.height * e.scaleY
      ) {
        e.target.top = this.boardInfo.height - e.target.height * e.scaleY;
        e.target.setCoords();
      }
    });

    this.initBackgroundBoard();
  },
  methods: {
    changeOptions(id) {
      this.currentOption = id
    },
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
      let _self = this;
      fabric.Image.fromURL(url, (oImg) => {
        _self.board.add(oImg);
      });
    },
    addText(text) {
      let _self = this;
      let t = new fabric.Textbox(text);
      _self.board.add(t);
    },
    clean() {
      this.board.clear();
    },
    copy() {
      this.board.getActiveObject().clone((cloned) => {
        cloned.clone((clonedObj) => {
          this.board.discardActiveObject();
          clonedObj.set({
            left: clonedObj.left + 50,
            top: clonedObj.top + 50,
            evented: true,
          });
          if (clonedObj.type === "activeSelection") {
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
        });
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
  .container {
    display: flex;
    .left,
    .right {
      width: 50%;
    }
    .left {
      .menu {
        display: flex;
        justify-content: center;
        li {
          list-style: none;
          padding: 10px;
          border: solid 1px gray;
          cursor: pointer;
        }
      }
      .options {
        display: flex;
        padding: 0;
        flex-wrap: wrap;
        li {
          list-style: none;
          margin: 5px;
          text-align: center;
          .option-img {
            border: solid 1px gray;
          }
          .option-text {
            border: solid 1px gray;
            width: 60px;
            line-height: 60px;
            
          }
        }
      }
    }
    .right {
      .buttons {
        width: 100%;
      }
      .canvas-container {
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
  }
  // .container {
  //   position: relative;
  //   width: 641px;
  //   height: 461px;
  //   .canvas-container:nth-child(1) {
  //     z-index: 1;
  //     position: absolute !important;
  //   }
  //   .canvas-container:nth-child(2) {
  //     z-index: 0;
  //     position: absolute !important;
  //   }
  // }
}
</style>
