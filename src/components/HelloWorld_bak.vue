<style scoped>
  .game-area-table {
    border-collapse: collapse;
  }
    .game-area-table td {
      width: 20px;
      height: 20px;
    }
  .current-block {
    background-color: red;
  }
  .done-block {
    background-color: gray;
  }
</style>

<template>
  <div class="game-area-table">
    <button @click="begin(gameArea, nextShape)">开始</button>
    <table cellspacing=0 cellpadding=0 border=1 >
      <tr v-for="(trItem, trIndex) in gameArea" :key="trIndex">
        <td 
          v-for="(tdItem, tdIndex) in trItem" 
          :key="tdIndex"
          :class="[(tdItem == 1) ? 'current-block' : '', 
                   (tdItem == 2) ? 'done-block' : '', 'block']"></td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      //游戏区域，0表示空白，1表示当前方块，2表示已下降的方块
      gameArea: [],
      shapeArray: [
        [
          { x: 0, y: 4 },
          { x: 1, y: 4 },
          { x: 2, y: 4 },
          { x: 3, y: 4 },
        ],
        [
          { x: 0, y: 5 },
          { x: 1, y: 5 },
          { x: 2, y: 5 },
          { x: 2, y: 4 },
        ],
        [
          { x: 0, y: 4 },
          { x: 1, y: 4 },
          { x: 2, y: 4 },
          { x: 2, y: 5 },
        ],
        [                 //O
          { x: 0, y: 4 },
          { x: 0, y: 5 },
          { x: 1, y: 4 },
          { x: 1, y: 5 },
        ],
        [                 //S
          { x: 0, y: 5 },
          { x: 0, y: 4 },
          { x: 1, y: 4 },
          { x: 2, y: 4 },
        ],
        [                 //Z
          { x: 0, y: 3 },
          { x: 0, y: 4 },
          { x: 1, y: 4 },
          { x: 1, y: 5 },
        ],
        [                 //T
          { x: 0, y: 3 },
          { x: 0, y: 4 },
          { x: 0, y: 5 },
          { x: 1, y: 4 },
        ]
      ],
      nextShape: [],
      time: null
    }
  },
  methods: {

    //生成0-6之间的随机数
    getRandom() {
      return (Math.floor(Math.random() * 20) + 1) % 7;
    },

    /**sdf 
     */
    initGameArea(length, width) {
      var gameArea = new Array(length);
      for(var i = 0; i < length; i++) {
        gameArea[i] = new Array(width);
      }
      for(var m = 0; m < length; m++) {
        for(var n = 0; n < width; n++) {
          gameArea[m][n] = 0;
        }
      }
      return gameArea;
    },
    
    //判断某位置是否在游戏区域内
    isBlockValid(gameArea, x, y) {
      if(x >= gameArea.length || x < 0 || y >= gameArea[x].length || y < 0) {
        return false;
      }
      return true;
    },
    //判断游戏区域的某位置是否已存在
    isBlockExist(gameArea, x, y) {
      if(gameArea[x][y] == 2) {
        return true;
      } 
      return false;
    },

    /**
     * 判断是否可以移动
     * offset表示不同方向的偏移量
     */
    canMove(gameArea, shape, xOffset, yOffset) {    
      var moveFlag = true;
      for(var i = 0; i < shape.length; i++) {
        var x = shape[i].x + xOffset;
        var y = shape[i].y + yOffset;
        if(!this.isBlockValid(gameArea, x, y) || this.isBlockExist(gameArea, x, y)) {
          moveFlag = false;
          break;
        }
      }
      return moveFlag;
    },
    
    /*是否触底
    isTouchBottom(gameArea, shape) {
      for(var i = 0; i < shape.length; i++) {
        var x = shape[i].x;
        var y = shape[i].y;
        if(gameArea[x][y] == 2 || shape[i].x == gameArea.length - 1) {

        }
      }
    },*/

    //刷新游戏区域
    refreshGameArea(gameAreaData, nextShape, value) {
      for(var i = 0; i < nextShape.length; i++) {
        this.$set(gameAreaData[nextShape[i].x], nextShape[i].y, value);
      }
    },
    move(nextShape, xOffset, yOffset) {
      for(var i = 0; i < nextShape.length; i++) {
        //this.$set(nextShape[i], "x", nextShape[i].x + 1);
        
        nextShape[i].x += xOffset;
        nextShape[i].y += yOffset;
      }
    },
    right() {

    },
    left() {

    },
    //复制形状
    copyShape(shape) {
      var tempShape = new Array(shape.length);
      for(var i = 0; i < shape.length; i++) {
        tempShape[i] = { x: 0, y: 0};
        tempShape[i].x = shape[i].x;
        tempShape[i].y = shape[i].y;
      }
      return tempShape;
    },
    //复制形状的值
    copyShapeValue(origin, dest) {
      for(var i = 0; i < origin.length; i++) {
        dest[i].x = origin[i].x;
        dest[i].y = origin[i].y;
      }
    },
    //旋转
    rotate(shape, gameArea) {
      var rotateFlag = true;
      //计算旋转中心点
      var cx = Math.round(
        shape.reduce(function(preValue, curValue) {
          return preValue + curValue.x;
        }, 0) / 4
      );
      var cy = Math.round(
        shape.reduce(function(preValue, curValue) {
          return preValue + curValue.y;
        }, 0) / 4
      ); 
      //console.log("中心点：(" + cx + "," + cy + ")");
      for(var i = 0; i < shape.length; i++) {
        //保存x坐标
        var tempX = shape[i].x;
        shape[i].x = cy + cx - shape[i].y;
        shape[i].y = cy - cx + tempX;
        //console.log(i + "after rotate: (" + shape[i].x + "," + shape[i].y + ")");
        if(!this.isBlockValid(gameArea, shape[i].x, shape[i].y)) {
          rotateFlag = false;
          break;
        }
      }
      return rotateFlag;
    },
    getNextShape() {

    },
    //消行
    deleteLine(gameArea, nextShape) {
      var lines = 0;
      for(var i = 0; i < gameArea.length; i++) {
        var j = 0;
        for(; j < gameArea[i].length; j++) {
          if(gameArea[i][j] == 0) 
            break;
        }
        if(j == gameArea[i].length) {
          lines++;
          if(i > 0) {
            for(var m = i - 1; m >= 0; m--) {
              //gameArea[m+1] = gameArea[m]
              for(var a = 0; a < gameArea[m+1].length; a++) {
                this.$set(gameArea[m+1], a, gameArea[m][a]);
              }
            }
          }
          for(var n = 0; n < gameArea[0].length; n++) {
            this.$set(gameArea[0], n, 0)
          }
        }
      }
      return lines;
    },
    isTop(gameArea, shape) {
      for(var i = 0; i < shape.length; i++) {
        var x = shape[i].x;
        var y = shape[i].y;
        if(gameArea[x][y] == 2)
          return true;
      }
      return false;
    },
    printArray(array) {
      for(var i = 0; i < array.length; i++) {
        console.log("(" + array[i].x + "," + array[i].y + ")");
      }
    },
    printGameArea(gameArea) {
      for(var i = 0; i < gameArea.length; i++) {
        var str = "[ ";
        for(var j = 0; j < gameArea[i].length; j++) {
          str += gameArea[i][j] + ", ";
        }
        str += "]";
        console.log(str)
      }
    },
    timerDown(gameArea, nextShape) {
      clearTimeout(this.time);
      var xOffset = 1, yOffset = 0;
      if(this.canMove(gameArea, nextShape, xOffset, yOffset)) {
        
        //清除原来的区域
        this.refreshGameArea(gameArea, nextShape, 0);
        this.move(nextShape, xOffset, yOffset);
        //更新区域
        this.refreshGameArea(gameArea, nextShape, 1);

      }
      
       else {
        this.refreshGameArea(gameArea, nextShape, 2);
        this.deleteLine(gameArea, nextShape);
        var random = this.getRandom(); 
        //由于数组是引用类型，所以需要copy一份，而不是直接指向
        this.copyShapeValue(this.shapeArray[random], nextShape);
        this.nextShape = nextShape;
        
        if(!this.isTop(gameArea, nextShape)) {
          this.refreshGameArea(gameArea, nextShape, 1);
        } else {
          return;
        }
      }
      setTimeout(() => {
        this.timerDown(gameArea, nextShape)
      }, 1000);
    },
    begin(gameArea, nextShape) {
      this.timer = setInterval(this.timerDown, 100, gameArea, nextShape)
    },
    opa() {
      console.log('s')
      clearTimeout(this.time);
      setTimeout(this.opa, 1000);
    }
  },
  mounted() {
    var valid = true;
    var _this = this;
    this.gameArea = this.initGameArea(18, 10);
    var gameArea = this.gameArea;
    var random = this.getRandom(); 
    //this.nextShape = this.shapeArray[random];
    //var nextShape = this.nextShape;
    var nextShape = this.copyShape(this.shapeArray[random]);
    this.refreshGameArea(gameArea, nextShape, 1);
    //this.timer = setInterval(this.timerDown, 100, gameArea, nextShape);
    document.onkeydown = function() {
      
      var e = window.event;
      if(e.keyCode == 37) {
        var xOffset = 0, yOffset = -1;
        if(_this.canMove(gameArea, nextShape, xOffset, yOffset)) {
          //清除原来的区域
          _this.refreshGameArea(gameArea, nextShape, 0);
          _this.move(nextShape, xOffset, yOffset);
          //更新区域
          _this.refreshGameArea(gameArea, nextShape, 1);
        }  
      }
      //旋转
      if(e.keyCode == 38) {
        var tempShape = _this.copyShape(nextShape);
        if(_this.rotate(nextShape, gameArea)) {
          //清除原来的区域
          _this.refreshGameArea(gameArea, tempShape, 0);
          //更新区域
          _this.refreshGameArea(gameArea, nextShape, 1);
        } else {
          for(var i = 0; i < tempShape.length; i++) {
            nextShape[i].x = tempShape[i].x;
            nextShape[i].y = tempShape[i].y;
          }
        }
        
      }
      //移动
      if(e.keyCode == 39) {
        var xOffset = 0, yOffset = 1;
        if(_this.canMove(gameArea, nextShape, xOffset, yOffset)) {
          //清除原来的区域
          _this.refreshGameArea(gameArea, nextShape, 0);
          _this.move(nextShape, xOffset, yOffset);
          //更新区域
          _this.refreshGameArea(gameArea, nextShape, 1);
        }  
      }
      //下落
      if(e.keyCode == 40) {
        var xOffset = 1, yOffset = 0;
        if(_this.canMove(gameArea, nextShape, xOffset, yOffset)) {
          //清除原来的区域
          _this.refreshGameArea(gameArea, nextShape, 0);
          _this.move(nextShape, xOffset, yOffset);
          //更新区域
          _this.refreshGameArea(gameArea, nextShape, 1);
        } else {
          //TODO 触底逻辑，后面修改
        }
      }
    }
    //this.time = setTimeout(_this.opa, 1000);
    this.time = setTimeout(() => {
      _this.timerDown(gameArea, nextShape)
    }, 1000);
  }
}
</script>




<!--
<template>
  <div class="game-area">
    <div v-for="(row, index) in gameArea" :key="index" class="row">
      <span 
        v-for="(item, itemIndex) in row" 
        :key="itemIndex" 
        :class="[(item == 1) ? 'current' : '', 
                 (item == 2) ? 'done' : '', 'block']"></span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      gameAreaData: [
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
        [ 2, 2, 2, 2, 0, 0, 2, 0, 0, 0 ],
      ],
      I: [
        { x: 0, y: 4 },
        { x: 1, y: 4 },
        { x: 2, y: 4 },
        { x: 3, y: 4 },
      ],
      J: [
        { x: 0, y: 5 },
        { x: 1, y: 5 },
        { x: 2, y: 5 },
        { x: 2, y: 4 },
      ],
      L: [
        { x: 0, y: 4 },
        { x: 1, y: 4 },
        { x: 2, y: 4 },
        { x: 2, y: 5 },
      ],
      O: [
        { x: 0, y: 4 },
        { x: 0, y: 5 },
        { x: 1, y: 4 },
        { x: 1, y: 5 },
      ],
      S: [
        { x: 0, y: 5 },
        { x: 0, y: 4 },
        { x: 1, y: 4 },
        { x: 2, y: 4 },
      ],
      Z: [
        { x: 0, y: 3 },
        { x: 1, y: 4 },
        { x: 1, y: 4 },
        { x: 2, y: 5 },
      ],
      T: [
        { x: 0, y: 3 },
        { x: 0, y: 4 },
        { x: 0, y: 5 },
        { x: 1, y: 4 },
      ],
    }
  },
  methods: {
    //判断是否可以下落
    canDown(position, next, gameArea) {    
      
    },
    //刷新游戏区域
    refreshGameArea(position, next, gameArea) {
      
    },
    //清除游戏区域
    clearGameArea(position, next, gameArea) {

    },
    down(position, next, gameArea) {
      
    },
    right() {

    },
    left() {

    },
    rotate(matrix) {

    }
  },
  mounted() {
    this.next = this.L;
    //this.refreshGameArea(this.position, this.next, this.gameArea);

    var _this = this;
    document.onkeydown = function() {
      var e = window.event;
      if(e.keyCode == 38) {
      }
      if(e.keyCode == 40) {
        _this.down(_this.position, _this.next, _this.gameArea);
      }
    }
  }
}
</script>

<style scoped>
.game-area {
  width: 200px;
  height: 400px;
  border: 1px solid black;
}
.block {
  width: 20px;
  height: 20px;
  box-sizing: border-box;
  display: inline-block;
  border: 1px solid black;
}
.current {
  background-color: red;
}
.done {
  background-color: gray;
}
.none {
  background-color: white;
}
.row {
  width: 200px;
  height: 20px;
}
</style>
-->