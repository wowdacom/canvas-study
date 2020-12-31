<template>
  <div class="hello">
    <div class="container">
      <div class="left">
        <ul class="menu">
          <li
            @click="changeOptions(item.id)"
            :key="item.id"
            v-for="item in menu"
          >
            {{ item.name }}
          </li>
          <li><input type="text" /></li>
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
import { onMounted } from "vue";

export default {
  name: "HelloWorld",
  data() {
    return {
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
      return this[this.currentOption];
    },
  },
  setup() {
    const boardInfo = {
      width: 641,
      height: 461,
    };
    var board = null,
      background = null;

    onMounted(() => {
      // console.log(background);
      background = new fabric.Canvas("background", {
        width: boardInfo.width,
        height: boardInfo.height,
      });

      board = new fabric.Canvas("board", {
        width: boardInfo.width,
        height: boardInfo.height,
      });

      const dotlinesLatitude = [],
        dotlinesLongitude = [];

      for (let i = 0; i < boardInfo.height; i = i + 10) {
        dotlinesLatitude.push(
          new fabric.Line([0, i, boardInfo.width, i], {
            strokeDashArray: [1],
            stroke: "black",
          })
        );
      }

      for (let j = 0; j < boardInfo.width; j = j + 10) {
        dotlinesLongitude.push(
          new fabric.Line([j, 0, j, boardInfo.height], {
            strokeDashArray: [1],
            stroke: "black",
            selectable: false,
          })
        );
      }

      background.add(...dotlinesLatitude);
      background.add(...dotlinesLongitude);

      board.on("object:modified", (e) => {
        if (
          e.target.getBoundingRect().left < 0 &&
          e.target.getBoundingRect().top < 0
        ) {
          e.target.left = 0;
          e.target.top = 0;
          e.target.setCoords();
        } else if (
          e.target.getBoundingRect().left < 0 &&
          e.target.getBoundingRect().top >
            boardInfo.height - e.target.height * e.scaleX
        ) {
          e.target.left = 0;
          e.target.top = boardInfo.height - e.target.height * e.scaleY;
          e.target.setCoords();
        } else if (
          e.target.getBoundingRect().top < 0 &&
          e.target.getBoundingRect().left >
            boardInfo.width - e.target.width * e.scaleX
        ) {
          e.target.top = 0;
          e.target.left = boardInfo.width - e.target.width * e.scaleX;
          e.target.setCoords();
        } else if (
          e.target.getBoundingRect().left >
            boardInfo.width - e.target.width * e.scaleX &&
          e.target.getBoundingRect().top >
            boardInfo.height - e.target.height * e.scaleY
        ) {
          e.target.left = boardInfo.width - e.target.width * e.scaleX;
          e.target.top = boardInfo.height - e.target.height * e.scaleY;
          e.target.setCoords();
        } else if (e.target.getBoundingRect().left < 0) {
          e.target.left = 0;
          e.target.setCoords();
        } else if (e.target.getBoundingRect().top < 0) {
          e.target.top = 0;
          e.target.setCoords();
        } else if (
          e.target.getBoundingRect().left >
          boardInfo.width - e.target.width * e.scaleX
        ) {
          e.target.left = boardInfo.width - e.target.width * e.scaleX;
          e.target.setCoords();
        } else if (
          e.target.getBoundingRect().top >
          boardInfo.height - e.target.height * e.scaleY
        ) {
          e.target.top = boardInfo.height - e.target.height * e.scaleY;
          e.target.setCoords();
        }
      });
    });

    function addImg(url) {
      fabric.Image.fromURL(url, (oImg) => {
        board.add(oImg);
      });
    }
    function addText(text) {
      let t = new fabric.Textbox(text);
      board.add(t);
    }
    function clean() {
      board.clear();
    }

    function copy() {
      board.getActiveObject().clone((cloned) => {
        cloned.clone((clonedObj) => {
          board.discardActiveObject();
          clonedObj.set({
            left: clonedObj.left + 50,
            top: clonedObj.top + 50,
            evented: true,
          });
          if (clonedObj.type === "activeSelection") {
            // active selection needs a reference to the canvas.
            clonedObj.canvas = board;
            clonedObj.forEachObject((obj) => {
              board.add(obj);
            });
            // this should solve the unselectability
            clonedObj.setCoords();
          } else {
            board.add(clonedObj);
          }
          cloned.top += 50;
          cloned.left += 50;
          board.setActiveObject(clonedObj);
          board.requestRenderAll();
        });
      });
    }

    function remove() {
      let delElement =
        board.getActiveObject()._objects !== undefined
          ? board.getActiveObject()._objects
          : [board.getActiveObject()];
      delElement.forEach((element) => {
        board.remove(element);
      });
      board.discardActiveObject();
      board.requestRenderAll();
    }

    function rotateLeft() {
      let currentElement = board.getActiveObject(),
        angle = currentElement.angle;

      currentElement.rotate(angle - 45);
      board.requestRenderAll();
    }

    function rotateRight() {
      let currentElement = board.getActiveObject(),
        angle = currentElement.angle;
      currentElement.rotate(angle + 45);
      board.requestRenderAll();
    }

    return {
      addImg,
      addText,
      clean,
      copy,
      remove,
      rotateLeft,
      rotateRight,
    };
  },
  mounted() {},
  methods: {
    changeOptions(id) {
      this.currentOption = id;
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
        border: solid 1px red;
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
          border: solid 1px green;
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
