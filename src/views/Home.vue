<template>
  <div v-for="window in windows" :key="window">
    <window 
    :title="window.title"
    :icon="window.icon" 
    v-if="window.active"
    v-show="window.visible"
    @minus="window.visible = false;window.focus = false;autoFocus();"
    @close="handleClose(window)"
    @focus="handleFocus(window)"
    :style="`z-index: ${window.zindex};`"
    ><component  :is="window.component"></component></window>
  </div>
  <div class="bar">
    <div 
      v-for="(window, index) in windows" 
      :key="index"
      class="bar-item"
      @click="handleClick(window)"
      @contextmenu="event=>{
          event.preventDefault();
          event.stopPropagation();
          window.menu = true;
          // next tick
          $nextTick(()=>{
           this.$refs.dropdown[0].focus();
          });
          
        }"
    > 
    <div 
      class="dropdown" 
      :style="`--height:${(window.active?options_active:options).length*25}px;--width:200px;}`"
      v-if="window.menu" 
      @blur="window.menu = false"
      ref="dropdown"
      @click="event=>
        {
          event.preventDefault();
          event.stopPropagation();
        }"
      tabindex="0" 
      >
      <p 
        v-for="{name, fun, icon} in window.active?options_active:options" 
        :key="name" 
        @click="window.menu = false;fun(window);"
      >
      <img v-if="icon == '{icon}'" :src="require('@/assets/' + window.icon + '')" class="icon" alt="">
      <i v-if="icon!=undefined && icon!='{icon}'" :class="`fa fa-${icon}`"></i> {{ name.replace(/{title}/g, window.title) }}
      </p>
    </div>

      <img :src="require('@/assets/' + window.icon + '')" class="icon" alt="">
      <div class="bar-underline" :class="{'active':window.active, 'selected':window.focus}"></div>
    </div>
    <div class="minimize-button"
      @click="()=>{
        windows.forEach(window=>{
          window.visible = false;
          window.focus = false
        });
        }"
    >

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

  data(){
    return {
      options:[
        {
          icon:"{icon}",
          name:"{title}",
          fun: this.handleOpen
        }
      ],
      options_active:[
        {
          icon:"{icon}",
          name:"{title}",
          fun: this.handleOpen
        },
        {
          icon:"times",
          name:"Close window",
          fun: this.handleClose
        }
      ],
      windows: [
        {
          title:'Config',
          icon:'gear.png',
          component:defineAsyncComponent(() => import('@/views/Config.vue'))
        },
        {
          title:'Solicitation',
          icon:'airplane.png',
          component:defineAsyncComponent(() => import('@/views/Solicitation.vue'))
        }
      ],
    }
  },

  created(){
    for(const i in this.windows)
    {
      this.windows[i].zindex = i;
      this.windows[i].active = false;
      this.windows[i].visible = false;
      this.windows[i].focus = false;
      this.windows[i].menu = false;
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
    },
    
    autoFocus()
    {
      var focus = false
      var maxZindex = 0
      var maxZindexWindow = null
      for(const i in this.windows)
      {
        if(this.windows[i].visible)
        {
          if(this.windows[i].focus)
          {
            focus = true;
          }
          if(this.windows[i].zindex >= maxZindex)
          {
            maxZindex = this.windows[i].zindex
            maxZindexWindow = this.windows[i]
          }
        }
      }
      if(!focus && maxZindexWindow)
      {
        maxZindexWindow.focus = true
      }
    },

    clearFocus()
    {
      for(const i in this.windows)
      {
        this.windows[i].focus = false;
      }
    },

    handleClick(window)
    {
        if(!window.active)
        {
          this.clearFocus()
          window.active = true
          window.visible = true
          window.focus = true
        }
        else
        if(!window.focus)
        {
          this.clearFocus()
          window.focus = true
          window.visible = true
        }
        else
        {
          if(!window.visible)
          {
            this.clearFocus()
            window.visible = true
            window.focus = true
          }
          else
          {
            window.visible = false
            window.focus = false
            this.autoFocus()
            return
          }
        }
        this.handleFocus(window)
    },

    handleClose(window)
    {
      window.active = false;
      window.visible = false;
      window.focus = false;
      this.autoFocus();
    },

    handleOpen(window)
    {
        this.clearFocus()
        window.active = true
        window.visible = true
        window.focus = true
        this.handleFocus(window)
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

.dropdown{
  background-color:var(--medium);
  border:1px solid var(--bright);
  width:var(--width);
  height:calc(20px + var(--height));
  margin-left: calc(20px - var(--width)/2);
  margin-top:calc(-30px - var(--height));
  margin-bottom: 10px;
  padding-top:10px;
  cursor:default;
  border-radius:5px;
  text-align: left;
}
.dropdown:focus{
  outline:none;
}
.dropdown p{
  margin-bottom: 0px;
  padding-left: 15px;
  cursor: pointer;
}
.dropdown p i{
  margin-right: 5px;
}
.dropdown p img{
  margin-right: 2px;
  margin-left: -1px;
  margin-top:-2px;
  width:12pt
}

.dropdown p:hover{
  background-color: var(--bright);
}

.minimize-button{
  border: 1px solid var(--bright);
  height:calc(100% + 12px);
  width:15px;
  float:right;
  margin-left:-20px;
  margin-top:-6px;
  margin-right: -5px;
  cursor: pointer;
}

.minimize-button:hover{
  background-color: var(--bright);
}
</style>