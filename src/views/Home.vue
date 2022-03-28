<template>
<div v-for="window in windows" :key="window">

<window 
:title="window.title"
:icon="window.icon" 
v-if="window.active"
v-show="window.visible"
@minus="window.visible = false;window.focus = false"
@close="window.active = false"
@focus="handleFocus(window)"
:style="`z-index: ${window.zindex};`"
><component  :is="window.component"></component></window>
</div>
  <div class="bar">
    <div 
      v-for="window in windows" 
      :key="window"
      class="bar-item"
      @click="()=>{
        if(!window.active)
        {
          window.active = true
          window.focus = true
        }
        window.visible = true
        handleFocus(window)
      }"
    > 
      <img :src="require('@/assets/' + window.icon + '')" class="icon" alt="">
      <div class="bar-underline" :class="{'active':window.active, 'selected':window.focus
      }"></div>
    </div>
  </div>
</template>
<script>
import { defineAsyncComponent } from "vue";
import { defineComponent } from 'vue'
import window from '@/components/window'
export default defineComponent({
  components: {
    window,
  },
  created(){
    for(const i in this.windows)
    {
      this.windows[i].zindex = i;
    }
  },
  data(){
    return {
      windows: [
        {
          title:'Config',
          icon:'gear.png',
          visible: false,
          active: false,
          focus:false,
          component:defineAsyncComponent(() => import('@/views/Config.vue'))
        },
        {
          title:'Solicitation',
          icon:'airplane.png',
          visible: false,
          active: false,
          focus:false,
          component:defineAsyncComponent(() => import('@/views/Solicitation.vue'))
        }
      ],
    }
  },
  methods:{
    handleFocus(window){
      for(const i in this.windows)
      {
        this.windows[i].sort = i;
      }
      this.windows.sort((a, b) => {
        return a.zindex - b.zindex
      })
      const cur = this.windows.findIndex(item => item.zindex == window.zindex);
      for(var i = cur;i<this.windows.length;i++)
      {
        this.windows[i].zindex = i-1;
        this.windows[i].focus = false;
      }
      this.windows[cur].zindex = this.windows.length-1;
      this.windows[cur].focus = true;
      this.windows.sort((a, b) => {
        return a.sort - b.sort
      })
    }
  }
})
</script>

<style scoped>
.bar{
  z-index: 1000;
  background-color: var(--medium);
  color:white;
  bottom: 0;
  position:fixed;
  --size: 40px;
  --padding: 5px;
  width: 100%;
  text-align: center;
  vertical-align: middle;
  height: calc(var(--size) + (var(--padding)*2));
  padding: var(--padding);
  border-top: 1px solid var(--bright);
  --bar-color:white;
}
.bar-item{
  height: 100%;
  width: var(--size);
  display: inline-block;
  vertical-align: middle;
  padding: 5px;
  cursor: pointer;
  margin-left: 3px;
  margin-right: 3px;
}
.bar-item:hover .bar-underline{
  width: 40px;
  margin-left: -5px;
}
.bar-underline.active{
  width: 40px;
  margin-left: -5px;
  border-color:var(--bar-color)
}
.bar-underline{
  border-bottom: 3px solid rgb(145, 145, 145);
  margin-top: 6px;
  margin-left: 12px;
  width: 5px;
  transition: width .2s ease-in-out, margin-left .2s ease-in-out, border-color .2s ease-in-out;
}
.bar-item img{
  height: 100%;
}
.selected{
  --bar-color:rgb(97, 130, 241);
}
</style>