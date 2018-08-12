<style scoped>
    ul {
        padding: 0px;
        margin: 0px;
        list-style-type: none;
        text-align: left;
    }
    .main {
        border: 1px solid red;
        padding: 5px;
    }
    .game-area-table {
        display: inline-block;
        vertical-align: top;
    }
    td {
        width: 20px;
        height: 20px;
    }
    .current-block {
        background-color: red;
    }
    .done-block {
        background-color: gray;
    }
    .main-right {
        margin-left: 10px;
        display: inline-block; 
        vertical-align: top;
    }
    .my-button {
        width: 50px;
        height: 50px;
        border-radius: 50px;
        box-shadow: inset -1px -1px 6px #000;
    }
</style>

<template>
    <div class="main">
        <div class="game-area-table">
            <table cellspacing=0 cellpadding=0 border=1 >
                <tr v-for="(trItem, trIndex) in gameArea" :key="trIndex">
                    <td 
                        v-for="(tdItem, tdIndex) in trItem"
                        :key="tdIndex"
                        :class="[(tdItem == 1) ? 'current-block' : '',
                            (tdItem == 2) ? 'done-block' : '', 'block']">
                    </td>
                </tr>
            </table>
        </div> 
        <div class="main-right">
            <table cellspacing=0 cellpadding=0 border=1 >
                <tr v-for="(trItem, trIndex) in previewShapeArea" :key="trIndex">
                    <td 
                        v-for="(tdItem, tdIndex) in trItem" 
                        :key="tdIndex"
                        :class="[(tdItem == 1) ? 'current-block' : '', 'block']">
                    </td>
                </tr>
            </table>
            <div>
                <ul>
                    <li>分数：{{score}}</li>
                    <li>级别：{{level}}</li>
                    <li>时间间隔： {{executeTime}}</li>
                </ul>
            </div>
        </div>
        <div>
            <ul>
                <li>
                    <my-button></my-button>
                </li>
                <li>
                    <my-button></my-button>
                    <my-button></my-button>
                </li>
                <li>
                    <my-button></my-button>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import MyButton from './button/index.vue'
    export default {
        name: 'MyTetris', 
        components: {
            MyButton,
        },
        data () {
            return {
                msg: 'Welcome to Your Vue.js App',
                // 游戏区域，0表示空白，1表示当前方块，2表示已下降的方块
                gameArea: [],
                // 七种图形
                shapeArray: [
                    [                 // I
                        { x: 0, y: 0 },
                        { x: 1, y: 0 },
                        { x: 2, y: 0 },
                        { x: 3, y: 0 },
                    ],
                    [                 // J
                        { x: 0, y: 1 },
                        { x: 1, y: 1 },
                        { x: 2, y: 1 },
                        { x: 2, y: 0 },
                    ],
                    [                 // L
                        { x: 0, y: 0 },
                        { x: 1, y: 0 },
                        { x: 2, y: 0 },
                        { x: 2, y: 1 },
                    ],
                    [                 // O
                        { x: 0, y: 0 },
                        { x: 0, y: 1 },
                        { x: 1, y: 0 },
                        { x: 1, y: 1 },
                    ],
                    [                 // S
                        { x: 0, y: 1 },
                        { x: 0, y: 0 },
                        { x: 1, y: 0 },
                        { x: 2, y: 0 },
                    ],
                    [                 // Z
                        { x: 0, y: 0 },
                        { x: 0, y: 1 },
                        { x: 1, y: 1 },
                        { x: 1, y: 2 },
                    ],
                    [                 // T
                        { x: 0, y: 0 },
                        { x: 0, y: 1 },
                        { x: 0, y: 2 },
                        { x: 1, y: 1 },
                    ]
                ],
                previewShapeArea: [], // 下一块图形预览区域
                nextShape: [],        // TODO 当前图形（命名有错误，后续改进）
                previewShape: [],     // 下一块图形
                timingExecution: null,// 定时任务
                gameStatus: 0,        // 游戏状态，0表示未开始，1表示运行中，2表示结束
                lines: 0,             // 消行行数
                score: 0
            }
        },
        /**
         * 等级、下落速率、图形整体偏移量
         * 通过计算属性得出
         */
        computed: {
            level: function() {
                if(this.score >= 10000) 
                    return 10;
                else 
                    return parseInt(this.score / 1000 % 10 + 1);
            },
            executeTime: function() {
                return (11- this.level) * 100;
            },
            shapeOffset: function() {
                return parseInt((this.gameAreaWidth - 2) / 2);
            }
        },
        props: {
            // 游戏区域相关属性
            gameAreaLength: {
                type: Number,
                default: 18
            },
            gameAreaWidth: {
                type: Number,
                default: 10
            },
        },
        methods: {

            // 生成0-6之间的随机数
            getRandom() {
                return (Math.floor(Math.random() * 20) + 1) % 7;
            },
            
            // 初始化游戏区域
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

            // 判断某位置是否在游戏区域内
            isBlockValid(gameArea, x, y) {
                if(x >= gameArea.length || x < 0 || y >= gameArea[x].length || y < 0) {
                    return false;
                }
                return true;
            },

            // 判断游戏区域的某位置是否已有方块
            isBlockExist(gameArea, x, y) {
                if(this.isBlockValid(gameArea, x, y)) {
                    if(gameArea[x][y] == 2) {
                        return true;
                    } else {
                        return false;
                    }
                } return true;
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
                    if(this.isBlockExist(gameArea, x, y)) {
                        moveFlag = false;
                        break;
                    }
                }
                return moveFlag;
            },

            // 判断是否到顶函数
            isTop(gameArea, shape) {
                for(var i = 0; i < shape.length; i++) {
                    var x = shape[i].x;
                    var y = shape[i].y;
                    if(gameArea[x][y] == 2)
                        return true;
                }
                return false;
            },

            /** 
             *移动函数
             *通过设置x，y方向上的偏移量，控制上下左右
             */
            move(gameArea, nextShape, xOffset, yOffset) {
                //如果可以移动
                if(this.canMove(gameArea, nextShape, xOffset, yOffset)) {
                    // 清除原来的区域
                    this.refreshGameArea(gameArea, nextShape, 0);
                    // 更新区域
                    for(var i = 0; i < nextShape.length; i++) {
                        nextShape[i].x += xOffset;
                        nextShape[i].y += yOffset;
                    }
                    this.refreshGameArea(gameArea, nextShape, 1);

                } else {
                    // 如果是下落状态
                    if(xOffset == 1 && yOffset == 0) {
                        // 设置游戏区域颜色为灰色
                        this.refreshGameArea(gameArea, nextShape, 2);
                        // 获取消行行数及分数
                        this.lines = this.deleteLine(gameArea, nextShape);
                        this.score = this.getScore(this.score, this.lines);
                        var random = this.getRandom();
                        // 把下一块预览区域内的图形赋值给nextShape
                        this.copyShapeValue(this.previewShape, nextShape);
                        this.nextShape = nextShape;
                        this.makeShapeOffset(nextShape, 0, this.shapeOffset);
                        this.refreshGameArea(this.previewShapeArea, this.previewShape, 0)
                        // 刷新下一块预览区域
                        this.copyShapeValue(this.shapeArray[random], this.previewShape);
                        this.refreshGameArea(this.previewShapeArea, this.previewShape, 1);
                        // 如果到顶，结束
                        if(!this.isTop(gameArea, nextShape)) {
                            this.refreshGameArea(gameArea, nextShape, 1);
                        } else {
                            this.gameStatus = 2;
                            return 2;
                        }
                    }
                }
                return 1;
            },

            /**
             * 刷新游戏区域
             * 在Vue中，为了实现双向绑定，对于数组类型来说
             * 需要用Vue.$set函数，才能侦测到数组的变化（对象类型亦是如此）
             */
            refreshGameArea(gameAreaData, nextShape, value) {
                for(var i = 0; i < nextShape.length; i++) {
                    // 三个参数分别表示数组对象，改变的位置，新值
                    this.$set(gameAreaData[nextShape[i].x], nextShape[i].y, value);
                }
            },

            /**
             * 复制形状函数
             * 这是因为数组是引用类型，不能直接用等号赋值
             * 需要重新开辟内存空间
             * 这是一个巨大的坑，要注意
             */
            copyShape(shape) {
                var tempShape = new Array(shape.length);
                for(var i = 0; i < shape.length; i++) {
                    tempShape[i] = { x: 0, y: 0};
                    tempShape[i].x = shape[i].x;
                    tempShape[i].y = shape[i].y;
                }
                return tempShape;
            },

            // 复制形状的值
            copyShapeValue(origin, dest) {
                for(var i = 0; i < origin.length; i++) {
                    dest[i].x = origin[i].x;
                    dest[i].y = origin[i].y;
                }
            },

            /** 
             * 使图形偏移
             * 主要是针对nextShpae
             * 从游戏区域中间开始往下落
             */
            makeShapeOffset(shape, xOffset, yOffset) {
                for(var i = 0; i < shape.length; i++) {
                    this.$set(shape[i], 'x', shape[i].x + xOffset);
                    this.$set(shape[i], 'y', shape[i].y + yOffset);
                }
                  return shape;
            },

            // 旋转函数
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
                
                for(var i = 0; i < shape.length; i++) {
                    // 保存x坐标，因为下一步的x坐标已经改变，不再是原来的坐标了
                    var tempX = shape[i].x;
                    shape[i].x = cy + cx - shape[i].y;
                    shape[i].y = cy - cx + tempX;
                    // 判断旋转后的点是否在游戏区域内
                    if(this.isBlockExist(gameArea, shape[i].x, shape[i].y)) {
                        rotateFlag = false;
                        break;
                    }
                }
                return rotateFlag;
            },
            
            /**
             * 消行函数 
             * 先进行行遍历，如果这行有空格，退出继续下一行
             * 否则从这行开始，上面一行整行下移
             * 最后第一行赋值0
             */
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

            // 获取分数
            getScore(score, lines) {
                switch(lines) {
                    case 0:
                        break;
                    case 1:
                        score += 10;
                        break;
                    case 2:
                        score += 40;
                        break;
                    case 3: 
                        score += 90;
                        break;
                    case 4: 
                        score += 160;
                        break;
                    default:
                        break;
                }
                return score;
            },
            
            // 打印数组
            printArray(array) {
                for(var i = 0; i < array.length; i++) {
                    console.log("(" + array[i].x + "," + array[i].y + ")");
                }
            },

            // 打印游戏区域
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
            
            // 初始化形状数组
            initShapeArea(length, width) {
                var shape = new Array(length);
                for(var i = 0; i < length; i++) {
                    shape[i] = new Array(width);
                }
                return shape;
            },

            /**
             * 定时向下落
             * 先清除上一次定时任务
             * 再重新开始定时任务
             * 防止程序的运行时间影响定时任务的周期
             */
            timingExecuteDown(gameArea, nextShape) {
                clearTimeout(this.time);
                var xOffset = 1, yOffset = 0;
                this.move(gameArea, nextShape, xOffset, yOffset);
                if(this.gameStatus == 1) {
                    this.time = setTimeout(() => {
                        this.timingExecuteDown(gameArea, nextShape);
                    }, this.executeTime);
                }
            },
        },
        mounted() {
            this.gameStatus = 1;
            this.gameArea = this.initGameArea(this.gameAreaLength, this.gameAreaWidth);
            var gameArea = this.gameArea;
            var random = this.getRandom(); 
            var nextShape = this.copyShape(this.shapeArray[random]);
            this.makeShapeOffset(nextShape, 0, this.shapeOffset);
            var previewShape = this.copyShape(this.shapeArray[this.getRandom()]);
            this.previewShape = previewShape;
            var previewShapeArea = this.initGameArea(4,4);
            this.previewShapeArea = previewShapeArea;
            this.refreshGameArea(previewShapeArea, previewShape, 1);
            this.refreshGameArea(gameArea, nextShape, 1);
            var _this = this;
            // 键盘事件
            document.onkeydown = function() {
                // 判断游戏状态，如果不在游戏中，则退出不响应键盘事件
                if(_this.gameStatus != 1) 
                    return ;
                var e = window.event;
                var xOffset = 0, yOffset = 0;
                // 左移
                if(e.keyCode == 37) {
                    xOffset = 0;
                    yOffset = -1;
                    _this.move(gameArea, nextShape, xOffset, yOffset);
                }
                //旋转
                if(e.keyCode == 38) {
                    // 先保存下一个图形的坐标数据
                    var tempShape = _this.copyShape(nextShape);
                    /**
                     * 判断变换后的形状是否还在游戏区域内
                     * 如果不在需要还原下一个图形的坐标数据
                     */
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
                // 右移
                if(e.keyCode == 39) {
                    xOffset = 0;
                    yOffset = 1; 
                    _this.move(gameArea, nextShape, xOffset, yOffset);
                }
                // 下移
                if(e.keyCode == 40) {
                    xOffset = 1;
                    yOffset = 0;
                    _this.move(gameArea, nextShape, xOffset, yOffset);
                }
            }
            // 自动下降
            this.time = setTimeout(() => {
                _this.timingExecuteDown(gameArea, nextShape)
            }, _this.executeTime);
            
        }
    }
</script>