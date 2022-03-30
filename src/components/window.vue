<template>
    <div 
      ref="window" 
      class="window" 
      :style="maximized?'border-radius:0px':''"  
      @mousedown="$emit('focus')"
    >
      <div ref='header' class="header" :drag="true">
        <div class="title"><img class = "icon" ref="icon" :src="require('@/assets/' + icon + '')">{{ title }}</div>
        <div @mousedown="preventDrag">
          <div class="nav-button close-button" :style="maximized?'border-radius:0px':''"  @click="$emit('close')"><div>×</div></div>
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
        draggable: false,
        drag_handle: null,
        pos1: 0,
        pos2: 0,
        pos3: 0,
        pos4: 0,
        resizable: true,
        top: null,
        left: null,
        right: null,
        bottom: null,
        corner1: null,
        corner2: null,
        corner3: null,
        corner4: null,
        offsetX: null,
        offsetY: null,
        startX: null,
        startW: null,
        maxX: null,
        minW: 100,
        minH:100,
        startY: null,
        startH: null,
        maxY: null,
        maximized: true,
        element: null,
        position:{
          left: 'calc(50% - 800px / 2)',
          top: 'calc(50% - 500px / 2)',
          width: '800px',
          height: '500px',
        }
      }
    },
    mounted(){
      this.dragElement(this.$refs.window, this.$refs.header);
      this.resizeElement(this.$refs.window);
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
        this.set_draggable(false)
        this.set_resizable(false)
      },
      handleMinimize(){
        this.maximized = false
        const window = this.$refs.window
        window.style.left = this.position.left
        window.style.top = this.position.top
        window.style.width = this.position.width
        window.style.height = this.position.height
        this.set_draggable(true)
        this.set_resizable(true)
      },
      set_draggable(value){
        this.draggable = value
        if(this.draggable)
        {
          this.drag_handle.style.cursor = 'move'
        }
        else
        {
          this.drag_handle.style.cursor = 'default'
        }
    },
    dragMouseDown(e) {
      if(e.button !== 0) return
      if(!this.draggable) return
      e = e || window.event;
      e.preventDefault();
      // get the mouse cursor position at startup:
      this.pos3 = e.clientX;
      this.pos4 = e.clientY;
      document.onmouseup = this.closeDragElement;
      // call a function whenever the cursor moves:
      document.onmousemove = this.elementDrag;
    },
    elementDrag(e) {
      e = e || window.event;
      e.preventDefault();
      // calculate the new cursor position:
      this.pos1 = this.pos3 - e.clientX;
      this.pos2 = this.pos4 - e.clientY;
      this.pos3 = e.clientX;
      this.pos4 = e.clientY;
      // set the element's new position:
      this.element.style.top = (this.element.offsetTop - this.pos2) + "px";
      this.element.style.left = (this.element.offsetLeft - this.pos1) + "px";
    },
    closeDragElement() {
      /* stop moving when mouse button is released:*/
      document.onmouseup = null;
      document.onmousemove = null;
    },
    dragElement(element, header = null) {
      this.drag_handle = header || element;
      this.element = element
      this.drag_handle.onmousedown = this.dragMouseDown;
      this.drag_handle.style.cursor = 'move';
    },
    set_resizable(value)
    {
        this.resizable = value
        if(this.resizable)
        {
            this.corner1.style.cursor = 'nw-resize'
            this.corner2.style.cursor = 'ne-resize'
            this.corner3.style.cursor = 'sw-resize'
            this.corner4.style.cursor = 'se-resize'
            this.top.style.cursor = 'n-resize'
            this.left.style.cursor = 'w-resize'
            this.right.style.cursor = 'e-resize'
            this.bottom.style.cursor = 's-resize'
        }
        else
        {
            this.corner1.style.cursor = 'default'
            this.corner2.style.cursor = 'default'
            this.corner3.style.cursor = 'default'
            this.corner4.style.cursor = 'default'
            this.top.style.cursor = 'default'
            this.left.style.cursor = 'default'
            this.right.style.cursor = 'default'
            this.bottom.style.cursor = 'default'
        }
    },
    resizeElement(element, minW = 100, minH = 100, size = 20)
    {
        this.element = element
        this.top = document.createElement('div');
        this.top.style.width = '100%';
        this.top.style.height = size + 'px';
        this.top.style.backgroundColor = 'transparent';
        this.top.style.position = 'absolute';
        this.top.style.top = - (size/2) + 'px';
        this.top.style.left = '0px';
        this.top.style.cursor = 'n-resize';

        this.top.addEventListener('mousedown',this.resizeYNegative)

        element.appendChild(this.top);

        this.bottom = document.createElement('div');
        this.bottom.style.width = '100%';
        this.bottom.style.height = size + 'px';
        this.bottom.style.backgroundColor = 'transparent';
        this.bottom.style.position = 'absolute';
        this.bottom.style.bottom = - (size/2) + 'px';
        this.bottom.style.left = '0px';
        this.bottom.style.cursor = 'n-resize';

        this.bottom.addEventListener('mousedown',this.resizeYPositive)

        element.appendChild(this.bottom);

        this.left = document.createElement('div');
        this.left.style.width = size + 'px';
        this.left.style.height = '100%';
        this.left.style.backgroundColor = 'transparent';
        this.left.style.position = 'absolute';
        this.left.style.top = '0px';
        this.left.style.left = - (size/2) + 'px';
        this.left.style.cursor = 'e-resize';

        this.left.addEventListener('mousedown',this.resizeXNegative)

        element.appendChild(this.left);

        this.right = document.createElement('div');
        this.right.style.width = size + 'px';
        this.right.style.height = '100%';
        this.right.style.backgroundColor = 'transparent';
        this.right.style.position = 'absolute';
        this.right.style.top = '0px';
        this.right.style.right = - (size/2) + 'px';
        this.right.style.cursor = 'e-resize';

        this.right.addEventListener('mousedown',this.resizeXPositive)

        element.appendChild(this.right);


        this.corner1 = document.createElement('div');
        this.corner1.style.width = size + 'px';
        this.corner1.style.height = size + 'px';
        this.corner1.style.backgroundColor = 'transparent';
        this.corner1.style.position = 'absolute';
        this.corner1.style.top = - (size/2) + 'px';
        this.corner1.style.left = - (size/2) + 'px';
        this.corner1.style.cursor = 'nw-resize';

        this.corner1.addEventListener('mousedown',this.resizeXNegative)
        this.corner1.addEventListener('mousedown',this.resizeYNegative)
        
        element.appendChild(this.corner1);

        this.corner2 = document.createElement('div');
        this.corner2.style.width = size + 'px';
        this.corner2.style.height = size + 'px';
        this.corner2.style.backgroundColor = 'transparent';
        this.corner2.style.position = 'absolute';
        this.corner2.style.top = - (size/2) + 'px';
        this.corner2.style.right = - (size/2) + 'px';
        this.corner2.style.cursor = 'ne-resize';

        this.corner2.addEventListener('mousedown',this.resizeXPositive)
        this.corner2.addEventListener('mousedown',this.resizeYNegative)

        element.appendChild(this.corner2);

        this.corner3 = document.createElement('div');
        this.corner3.style.width = size + 'px';
        this.corner3.style.height = size + 'px';
        this.corner3.style.backgroundColor = 'transparent';
        this.corner3.style.position = 'absolute';
        this.corner3.style.bottom = - (size/2) + 'px';
        this.corner3.style.left = - (size/2) + 'px';
        this.corner3.style.cursor = 'sw-resize';

        this.corner3.addEventListener('mousedown',this.resizeXNegative)
        this.corner3.addEventListener('mousedown',this.resizeYPositive)

        element.appendChild(this.corner3);

        this.corner4 = document.createElement('div');
        this.corner4.style.width = size + 'px';
        this.corner4.style.height = size + 'px';
        this.corner4.style.backgroundColor = 'transparent';
        this.corner4.style.position = 'absolute';
        this.corner4.style.bottom = - (size/2) + 'px';
        this.corner4.style.right = - (size/2) + 'px';
        this.corner4.style.cursor = 'se-resize';

        this.corner4.addEventListener('mousedown',this.resizeXPositive)
        this.corner4.addEventListener('mousedown',this.resizeYPositive)

        element.appendChild(this.corner4);
    },
    get_int_style(key)
    {
        return parseInt(window.getComputedStyle(this.element).getPropertyValue(key));
    },
    resizeXPositive(e) {
      if(e.button !== 0) return
      if(!this.resizable) return;
      e = e || window.event;
      e.preventDefault();
      const {clientX} = e;
      this.offsetX = clientX - this.element.offsetLeft - this.get_int_style('width');
      document.addEventListener('mouseup', this.resizeXPositiveCloseDragElement)
      document.addEventListener('mousemove', this.resizeXPositiveElementDrag)
    },
    resizeXPositiveElementDrag(e) {
        const {clientX} = e;
        let x = clientX - this.element.offsetLeft - this.offsetX
        if(x < this.minW) x = this.minW;
        this.element.style.width =  x + 'px';
    },
    resizeXPositiveCloseDragElement() {
      document.removeEventListener("mouseup", this.resizeXPositiveCloseDragElement);  
      document.removeEventListener("mousemove", this.resizeXPositiveElementDrag);
    },
    resizeYPositive(e) 
    {
      if(e.button !== 0) return
      if(!this.resizable) return;
      e = e || window.event;
      e.preventDefault();
      const {clientY} = e;
      this.offsetY = clientY - this.element.offsetTop - this.get_int_style('height');

      document.addEventListener('mouseup',this.resizeYPositiveCloseDragElement)
      document.addEventListener('mousemove',this.resizeYPositiveElementDrag)
    },
    resizeYPositiveCloseDragElement() 
    {
      document.removeEventListener("mouseup", this.resizeYPositiveCloseDragElement);  
      document.removeEventListener("mousemove", this.resizeYPositiveElementDrag);
    },
    resizeYPositiveElementDrag(e) 
    {
        const {clientY} = e;
        let y =  clientY - this.element.offsetTop - this.offsetY;
        if(y < this.minH) y = this.minH;
        this.element.style.height = y + 'px';
    },
    resizeXNegative(e) {
      if(e.button !== 0) return
      if(!this.resizable) return;
      e = e || window.event;
      e.preventDefault();
      const {clientX} = e;
      this.startX = this.get_int_style('left')
      this.startW = this.get_int_style('width')
      this.offsetX = clientX - this.startX;
      this.maxX = this.startX + this.startW - this.minW

      document.addEventListener('mouseup',this.resizeXNegativeCloseDragElement)
      document.addEventListener('mousemove',this.resizeXNegativeElementDrag)
    },
    resizeXNegativeElementDrag(e) {
          const {clientX} = e;
          let x = clientX - this.offsetX
          let w = this.startW + this.startX - x
          if(w < this.minW) w = this.minW;
          if(x > this.maxX) x = this.maxX;
          this.element.style.left = x + 'px';
          this.element.style.width = w + 'px';
    },
    resizeXNegativeCloseDragElement() {
      document.removeEventListener("mouseup", this.resizeXNegativeCloseDragElement);  
      document.removeEventListener("mousemove", this.resizeXNegativeElementDrag);
    },
    resizeYNegative(e) {
      if(e.button !== 0) return
      if(!this.resizable) return;
      e = e || window.event;
      e.preventDefault();
      const {clientY} = e;
      this.startY = this.get_int_style('top')
      this.startH = this.get_int_style('height')
      this.offsetY = clientY - this.startY;
      this.maxY = this.startY + this.startH - this.minH 

      document.addEventListener('mouseup',this.resizeYNegativeCloseDragElement,false)
      document.addEventListener('mousemove',this.resizeYNegativeElementDrag,false)
    },
    resizeYNegativeElementDrag(e) {
        const {clientY} = e;
        let y =  clientY - this.offsetY
        let h = this.startH + this.startY - y
        if(h < this.minH) h = this.minH;
        if(y > this.maxY) y = this.maxY;
        this.element.style.top = y + 'px';
        this.element.style.height = h + 'px';
    },
    resizeYNegativeCloseDragElement() {
      document.removeEventListener("mouseup", this.resizeYNegativeCloseDragElement);  
      document.removeEventListener("mousemove", this.resizeYNegativeElementDrag);
    }
  },
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
  height: calc(100% - 30px); 
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