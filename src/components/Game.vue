<template>
  <div class="game">
    <div class="noteboard clearfix">
      <div class="time">时间：{{time}}</div>

      <div class="score">得分：{{score}}</div>
    </div>
    <div class="gamestage">
      <div class="updown-wrap">
        <div class="updown-pos" ref="objgroup">
          <div class="updown" >
            
              <div :class="item.className" v-for="item in items" ref=objs></div>
            
          </div>
        </div>
      </div>
      <div class="btn-group">

        <div :class="btn.className" v-for="btn in btns" @click="btnclick(btn.type)"></div>

      </div>
      <div v-show="wran" class="warn warn-ani">{{time}}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'game',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      time: 30,
      score: 0,
      objDownY: -105,
      objDownDY: 0,
      moveCount: 0,
      items: [],
      classNames: ['yellow','blue','red'],
      objClass: ['obj obj1','obj obj2','obj obj3','obj obj4','obj obj5'],
      btns: [],
      moved: false,
      lastObjL: 0,
      wrongbtn: false,
      gameover: false,
      wran: false,
    }
  },
  methods: {
    btnclick (p) {
      
      if (!this.moved && !this.gameover) {//动画结束，执行下一个动画

        let lastItem = this.items[this.items.length - 1 - this.moveCount]

        if(lastItem.type == p){
          this.score += 1
          this.moved = true
          this.render()
        } else { // 点击错误
          this.wrongClick()
        }
      }
    },
    wrongClick () {
      this.moved = true
      let lastItem = this.items[this.items.length - 1 - this.moveCount]
      let lastObj = this.$refs.objs[lastItem.index]
      lastObj.className += ' wrong-ani'
      for (var i = 0; i < this.btns.length; i++) {
        this.btns[i].className = this.btns[i].className + ' gray'
      }
      let vm = this
      setTimeout(function () {
        for (var i = 0; i < vm.btns.length; i++) {
          let claname = vm.btns[i].className
          vm.btns[i].className = claname.substring(0,claname.length - 5)
        }
        lastObj.className = lastObj.className.substring(0,lastObj.className.length - 10)
        vm.moved = false
      },500)
    },
    objDown () {
      let objGroup = this.$refs.objgroup
      this.objDownY += 5
      this.objDownDY += 5
      objGroup.style.top = this.objDownY * 0.01 + 'rem'
    },
    moveLastObj () {
      let lastItem = this.items[this.items.length - 1 - this.moveCount]
      let lastObj = this.$refs.objs[lastItem.index]
      if (lastItem.type < 1) {
        this.lastObjL += -15
      } else if (lastItem.type > 1) {
        this.lastObjL += 15
      } else {
        this.lastObjL += 0
      }
      
      lastObj.style.left = this.lastObjL * 0.01 + 'rem'
    },
    refreshObj (o) {
      o.y += 105
      this.$refs.objs[o.index].style.top = o.y * 0.01 + 'rem'
    },
    setObj (items,i) {
      let obj = {}
      let random = this.RandomNumBoth(0,2)
      obj.name = 'obj' + (i + 1)
      obj.index = i
      obj.y = i * 105
      obj.className = 'obj obj' + (i + 1) + ' ' + this.classNames[random]
      let type = 0
      // 根据按钮颜色对应的左中右 设置当前对象的左中右
      for (var i = 0; i < this.btns.length; i++) {
        if(this.btns[i].className == 'btn ' + this.classNames[random]){
          type = i
        }
      }

      obj.type = type
      items.push(obj)
    },
    resetLastObj () {

      let random = this.RandomNumBoth(0,2)
      let index = this.items.length - 1 - this.moveCount
      let lastItem = this.items[index]
      lastItem.className = 'obj obj' + (index + 1) + ' ' + this.classNames[random]
      // 根据按钮颜色对应的左中右 设置当前对象的左中右
      let type = 0
      for (var i = 0; i < this.btns.length; i++) {
        if(this.btns[i].className == 'btn ' + this.classNames[random]){
          type = i
        }
      }
      lastItem.type = type
      // 最后一个对象移动到顶点
      let lastObj = this.$refs.objs[lastItem.index]
      lastObj.style.top = 0 + 'rem'
      lastObj.style.left = 0 + 'rem'
      lastItem.y = 0

    },
    RandomNumBoth(Min,Max) {
      var Range = Max - Min;
      var Rand = Math.random();
      var num = Min + Math.round(Rand * Range); //四舍五入
      return num;
    },
    setBtn () {
      for (var i = 0; i < 3; i++) {
        let btn = {}
        btn.className = 'btn ' + this.classNames[i]
        this.btns.push(btn)
      }
    },
    render () {
      if (this.objDownDY < 105) {

        this.objDown()
        this.moveLastObj()
        requestAnimationFrame(this.render)

      } else {
        this.objDownDY = 0 //重置
        this.lastObjL = 0 //重置
        this.objDownY = -105 // 重置

        let objGroup = this.$refs.objgroup
        objGroup.style.top = this.objDownY * 0.01 + 'rem'

        for (var i = 0; i < this.items.length; i++) {
          this.refreshObj(this.items[i])
        }

        this.resetLastObj()

        this.moved = false // 标记一次动画结束
        this.moveCount = (this.moveCount + 1) > this.items.length - 1 ? 0 : this.moveCount + 1 // 移动次数加一 循环
        
      }
    }
  },
  mounted () {
      if (!Array.prototype.shuffle) {
        Array.prototype.shuffle = function() {
            for(var j, x, i = this.length; i; j = parseInt(Math.random() * i), x = this[--i], this[i] = this[j], this[j] = x);
            return this;
        }
      }
      // 初始化按钮数组
      this.setBtn()
      // 按钮乱序
      this.btns.shuffle()
      // 根据乱序结果设置左中右
      for (var i = 0; i < this.btns.length; i++) {
        this.btns[i].type = i
      }
      // 初始化元素
      for (var i = 0; i < 5; i++) {
        this.setObj(this.items,i)
      }
      let vm = this
      vm.si = setInterval(function(){
          if(vm.time >= 1){
            vm.time -= 1
            if (vm.time <= 10 && !vm.wran) {
              vm.wran = true
            }
          } else {
            clearInterval(vm.si)
            vm.gameover = true
            vm.wran = false
            sessionStorage.setItem('score',vm.score)
            vm.$router.push('/over')
          }
        },1000)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.game{
  height: 100%;
  padding-top: 1rem;
  box-sizing: border-box;
}
.noteboard div{
  float:left;
  width: 50%;
  padding:.25rem;
  box-sizing: border-box;
  text-align: center;
}

.time{
  text-align: right;
}
.clearfix{
  overflow: hidden;
}
.gamestage{
  position: relative;
}
.warn{
  width:1rem;
  height: 1rem;
  /*background: rgba(122,23,23,0.5);*/
  border-radius: 30%;
  line-height: 1rem;
  text-align: center;
  position: absolute;
  bottom: 1.5rem;
  left: 1.5rem;
  font-size: 24px;
}
.warn-ani{
  animation: warn-ani 1s linear infinite;
}
@keyframes warn-ani{
  0%{
    transform: scale(1);
  }
  100%{
    transform: scale(1.4);
  }
}
.updown-wrap{
  width: 1.5rem;
  height: 4.25rem;
  overflow: hidden;
  margin: auto;
  position: relative;
}
.updown-pos{
  width: 1rem;
  height: 5.3rem;
  position: absolute;
  left:.25rem;
  top: -1.05rem;
}
.updown{
  width: 1rem;
  height: 5.3rem;
  position: absolute;
}
.obj1{
  top:0;
}
.obj2{
  top:1.05rem;
}
.obj3{
  top:2.1rem;
}
.obj4{
  top:3.15rem;
}
.obj5{
  top:4.2rem;
}
.obj6{
  top:5.25rem;
}
.obj{
  width: 1rem;
  height: 1rem;
  margin-top: .05rem;
  float:left;
  background: yellow;
  position: absolute;
}
.btn-group{
  width: 3.3rem;
  height: 1rem;
  margin: .25rem auto;
}
.btn{
  width: 1rem;
  height: 1rem;
  float: left;
  margin: 0 .05rem;
  border-radius: 50%;
  background: green;
  text-align: center;
  line-height: 1rem;
}
.yellow{
  background: yellow;
}
.red{
  background: red;
}
.blue{
  background: blue;
}
.gray{
  background: gray;
}
.wrong-ani{
  animation: wrong-ani .25s linear;
}

@keyframes wrong-ani{
  0%{
    transform: translate(2px,2px);
  }
  5%{
    transform: translate(-2px,2px);
  }
  10%{
    transform: translate(-2px,-2px);
  }
  15%{
    transform: translate(2px,-2px);
  }
  20%{
    transform: translate(2px,2px);
  }
  25%{
    transform: translate(-2px,2px);
  }
  30%{
    transform: translate(-2px,-2px);
  }
  35%{
    transform: translate(2px,-2px);
  }
  40%{
    transform: translate(2px,2px);
  }
  45%{
    transform: translate(-2px,2px);
  }
  50%{
    transform: translate(-2px,-2px);
  }
  55%{
    transform: translate(2px,-2px);
  }
  60%{
    transform: translate(2px,2px);
  }
  65%{
    transform: translate(-2px,2px);
  }
  70%{
    transform: translate(-2px,-2px);
  }
  75%{
    transform: translate(2px,-2px);
  }
  80%{
    transform: translate(2px,2px);
  }
  85%{
    transform: translate(-2px,2px);
  }
  90%{
    transform: translate(-2px,-2px);
  }
  95%{
    transform: translate(2px,-2px);
  }
  100%{
    transform: translate(0,0);
  }
}
</style>
