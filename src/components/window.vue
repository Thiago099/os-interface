<template>
    <div ref="window" class="window" :class="{'maximized':maximized}"  @mousedown="$emit('focus')">
      <div ref='header' class="header" :drag="true">
        <div class="title"><img class = "icon" ref="icon" :src="require('@/assets/' + icon + '')">{{ title }}</div>
        <div @mousedown="preventDrag">
          <div class="nav-button close-button" @click="$emit('close')"><div>×</div></div>
          <div class="nav-button maximize-button" v-if="maximized" @click="handleMinimize()"><div>🗗</div></div>
          <div class="nav-button minimize-button" v-else @click="handleMaximize()"><div>□</div></div>
          <div class="nav-button minus-button" @click="$emit('minus')"><div>-</div></div>
        </div>
      </div>
      <div class="content">
        <slot>aaa</slot>
      </div>
    </div>
</template>
<script>
import {dragElement, set_draggable} from './plugins/drag.js'
import {resizeElement, set_resizable} from './plugins/resize.js'
  export default {
    props: {
      title: {
        type: String,
        default: 'title'
      },
      icon:{
        type: String,
      },
    },
    data(){
      return {
        maximized: true,
        position:{
          left: 'calc(50% - 800px / 2)',
          top: 'calc(50% - 500px / 2)',
          width: '800px',
          height: '500px',
        }
      }
    },
    mounted(){
      dragElement(this.$refs.window, this.$refs.header);
      resizeElement(this.$refs.window);
      this.handleMinimize();
      this.handleMaximize()
    },
    methods:{
      preventDrag(event){
        event.cancelBubble = true
        event.preventDefault()
      },
      handleMaximize(){
        this.maximized = true
        const window = this.$refs.window
        this.position.left = window.style.left
        this.position.top = window.style.top
        this.position.width = window.style.width
        this.position.height = window.style.height
        window.style.left = 0
        window.style.top = 0
        window.style.width = '100%'
        window.style.height = '100%'
        set_draggable(false)
        set_resizable(false)
      },
      handleMinimize(){
        this.maximized = false
        const window = this.$refs.window
        window.style.left = this.position.left
        window.style.top = this.position.top
        window.style.width = this.position.width
        window.style.height = this.position.height
        set_draggable(true)
        set_resizable(true)
      }
    }
  }
</script>
<style scoped>
.window {
  position: absolute;
  z-index: 9;
  background-color: var(--dark);
  border: 1px solid var(--bright);
  overflow: hidden;
  left: calc(50% - 800px / 2);
  top: calc(50% - 500px / 2);
  width: 800px;
  height: 500px;
  border-radius: 5px;
  color:white;
}

.content{
  margin-top: 0px;
  overflow: auto;
  width: 100%;
  height: calc(100% - 45px); 
  padding: 10px;
}
.header > .title{
  margin-top: -8px;
}
.header {
  --drag: 1;
  overflow: hidden;

  border-radius: 5px 5px 0 0;
  padding: 10px;
  z-index: 10;
  background-color: var(--medium);
  border:1px solid var(--medium);
  color: #fff;
  height: 30px;
  
}
.nav-button{
  cursor: pointer;
  float: right;
  font-size: 1.6em;
  width: 30px;
  height: 30px;
  text-align: center;
  /* overflow: hidden; */
  transition: background-color .2s;
  background-color: var(--medium);
  position: absolute; top: 0px; right: 0px;
}
.nav-button:hover{
    background-color: var(--bright);
}
.close-button{
    border-radius: 0 5px 0 0;
}
.close-button > div{
  margin-top: -7px;
}
.close-button:hover{
  background-color: red;
}
.maximize-button{
  font-size: 14pt;
  margin-right: 30px;
}
.minimize-button{
  font-size: 15pt;
  margin-right: 30px;
}
.minimize-button > div{
  margin-top: -4px;
}
.maximize-button > div{
  margin-top: -1px;
}
.minus-button{
  font-size: 30pt;
  margin-right: 60px;
}
.minus-button > div{
  margin-top: -20px;
}

.icon{
  width: 15px;
  margin-top: -3px;
  margin-left: -3px;
  margin-right: 5px;
}
.footer {
   position: fixed;
   left: 0;
   bottom: 0;
   width: 100%;
   text-align: center;
}
</style>