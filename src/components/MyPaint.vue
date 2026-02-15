<template>
  <v-container fluid>
    <v-sheet elevation="5" width="100%" height="100%">
      <v-row align="center" justify="center">            
        <canvas id="canvasFrame" :width="canvasWidth" :height="canvasHeight"></canvas>
      </v-row> 
    </v-sheet>
  
    <div>
      <v-card class="py-2 px-1" elevation="2" width="220px" style="position: absolute; top: -1px; left: -1px;">
        <v-row align="center" justify="start">
          <v-col sm="1">
            <v-menu offset-y>
              <template v-slot:activator="{ on, attrs }">
                 <v-icon dense v-bind="attrs" v-on="on">mdi-menu</v-icon>
              </template>
              <v-list>              
                <v-list-item> <v-icon @click="doNewPage">mdi-file-plus-outline</v-icon> </v-list-item>
                <v-list-item><v-icon @click="doOpenFile">mdi-folder-open-outline</v-icon> </v-list-item>
                <v-list-item><v-icon @click="doSaveAsFile">mdi-floppy</v-icon></v-list-item>
                <v-list-item><v-icon @click="doExportPng">mdi-export</v-icon></v-list-item>      
              </v-list>
            </v-menu> 
          </v-col>
          <v-col sm="8">     
            <v-select dense hide-details rounded height="20px" :items="pagesList" v-model="pageSel" item-value="pageNo" item-text="pageName" return-object @change="doChangePage"/>
          </v-col>
          <v-col sm="1">
            <v-menu offset-y>
              <template v-slot:activator="{ on, attrs }">
                 <v-icon small v-bind="attrs" v-on="on">mdi-dots-vertical</v-icon>
              </template>
              <v-list>              
                <v-list-item> <v-icon small @click="doRenamePage(pageSel)">mdi-rename-box-outline</v-icon> </v-list-item>
                <v-list-item> <v-icon small @click="doDeletePage(pageSel)">mdi-window-close</v-icon> </v-list-item>               
              </v-list>
            </v-menu> 
          </v-col>
        </v-row>
      </v-card>
    </div>

    <div>
      <v-card class="py-2 px-1" elevation="2" width="135px" style="position: fixed; top: -3px; right: -2px;"> 
        <v-btn-toggle dense shaped group color="primary" v-model="toggle_exclusive">
          <v-btn x-small width="8"><v-icon @click="doTextBold" small>mdi-format-bold</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="doTextItalic" small>mdi-format-italic</v-icon></v-btn>      
          <v-btn x-small width="8"><v-icon @click="doTextFontSize(-2)" small>mdi-format-annotation-minus</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="doTextFontSize(2)" small>mdi-format-annotation-plus</v-icon></v-btn> 
          <input type="color" v-model="fontColor" @change="setFontColor(fontColor)"/>           
        </v-btn-toggle>    
      </v-card>
    </div>

    <div>
      <v-card class="py-2 px-1" elevation="2" width="250px" style="position: fixed; bottom: -3px; left: -2px;"> 
        <v-btn-toggle dense shaped group color="primary" v-model="toggle_exclusive">
          <v-btn x-small width="8"><v-icon @click="setBg" large>mdi-billboard</v-icon></v-btn>      
          <v-btn x-small width="8"><v-icon @click="zoomOutIn(1)" small>mdi-magnify-plus</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="zoomOutIn(-1)" small>mdi-magnify-minus</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="zoomOutIn(0)" small>mdi-fit-to-page-outline</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="doObjsGroup()" small>mdi-group</v-icon> </v-btn>
          <v-btn x-small width="8"><v-icon @click="doCopy" small>mdi-content-copy</v-icon> </v-btn>
          <v-btn x-small width="8"><v-icon @click="doPaste" small>mdi-content-paste</v-icon> </v-btn>      
          <v-btn x-small width="8"><v-icon @click="handMode(20)" small>mdi-undo</v-icon></v-btn>      
          <v-btn x-small width="8"><v-icon @click="handMode(21)" small>mdi-redo</v-icon></v-btn>           
        </v-btn-toggle>    
      </v-card>
    </div>

    <div>
      <v-card class="py-2 px-1" elevation="15" width="830px" style="position: fixed; bottom: -3px; left: 500px;"> 
        <v-btn-toggle dense shaped group color="primary" v-model="toggle_exclusive">
          <v-btn x-small width="8"><v-icon @click="handMode(0)" dense>mdi-cursor-default-outline</v-icon></v-btn>      
          <v-btn x-small width="8"><v-icon @click="handMode(19)" dense>mdi-hand-back-right-outline</v-icon></v-btn>    
          <v-btn x-small width="8"><v-icon @click="setFillFlag" dense>mdi-arrange-send-backward</v-icon></v-btn>
          
          <v-btn x-small width="8"><v-avatar color="red" size="20" @click="setPenColor('red')"></v-avatar></v-btn>
          <v-btn x-small width="8"><v-avatar color="green" size="20" @click="setPenColor('green')"></v-avatar></v-btn>
          <v-btn x-small width="8"><v-avatar color="yellow" size="20" @click="setPenColor('yellow')"></v-avatar></v-btn>
          <v-btn x-small width="8"><v-avatar color="black" size="20" @click="setPenColor('black')"></v-avatar></v-btn>
          <v-btn x-small width="8"><v-avatar color="white" size="20" @click="setPenColor('white')"></v-avatar></v-btn>
          <input type="color" v-model="penColor" @change="setPenColor(penColor)"/>    
          
          <v-btn x-small width="8"><v-icon @click="handMode(1)" dense>mdi-pencil</v-icon></v-btn>  
          <v-btn x-small width="8"><v-icon @click="handMode(15)" dense>mdi-brush</v-icon></v-btn>        

          <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
               <v-icon dense v-bind="attrs" v-on="on">mdi-format-line-weight </v-icon>
            </template>
            <v-list>
              <v-list-item>          
                <v-btn x-small width="8" @click="setPenWidth(1)"><v-avatar color="black" size="3"></v-avatar> </v-btn>                
                <v-btn x-small width="8" @click="setPenWidth(5)"><v-avatar color="black" size="5"></v-avatar> </v-btn>
                <v-btn x-small width="8" @click="setPenWidth(10)"><v-avatar color="black" size="10"></v-avatar> </v-btn>
                <v-btn x-small width="8" @click="setPenWidth(20)"><v-avatar color="black" size="20"></v-avatar> </v-btn>
              </v-list-item>
              <v-list-item>          
                <input type="number" v-model="penWidth" @change="setPenWidth(penWidth)"/>  
              </v-list-item>
            </v-list>
          </v-menu>
                
          <v-btn x-small width="8"><v-icon @click="penDash" dense>mdi-current-dc</v-icon></v-btn>      
          <v-btn x-small width="8"><v-icon @click="handMode(3)" large>mdi-note-text-outline</v-icon></v-btn>
          <v-btn x-small width="8"><v-icon @click="handMode(4)" dense>mdi-checkbox-blank-outline</v-icon></v-btn>  
          <v-btn x-small width="8"><v-icon @click="handMode(5)" dense>mdi-checkbox-blank-circle-outline</v-icon></v-btn>  
          <v-btn x-small width="8"><v-icon @click="handMode(8)" dense>mdi-minus</v-icon></v-btn>              
          <v-btn x-small width="8"><v-icon @click="handMode(10)" dense>mdi-arrow-right-thin</v-icon></v-btn>  
          <v-btn x-small width="8"><v-icon @click="handMode(13)" dense>mdi-format-color-text</v-icon> </v-btn>

          <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
               <v-icon dense v-bind="attrs" v-on="on">mdi-file-image</v-icon>
            </template>
            <v-list>              
              <v-list-item>          
                <v-file-input id="idImgFile" label="Carica immagine" accept="image/*" @change="insertImageFile"/>
              </v-list-item>
            </v-list>
          </v-menu>    

          <v-btn x-small width="8"><v-icon @click="handMode(14)" dense>mdi-spray</v-icon> </v-btn>                  
          <v-btn x-small width="8"><v-icon @click="doRemove" dense>mdi-delete-circle</v-icon> </v-btn>        
        </v-btn-toggle>    
      </v-card>
    </div>   
  </v-container>  
</template>

<script>
import localforage from 'localforage'
import { fabric } from 'fabric'

export default {
  name: 'MyPaint',

  data: () => ({      
    canvasWidth: 1900,
    canvasHeight: 1000,   
    lastPointer: null,
    scale: 1,
    penColor: '#000000',
    fontColor: '#000000',
    penWidth: 2,      
    myCanvas: null,      
    earseFlag: false,
    mouseEventFlag: false,
    drawingObj: null,
    drawType: 0,
    fillFlag: false,
    strokeFlag: true,      
    defaultNoteRect: {        
      angle: 3,
      fill: '#FFF9C4',
      shadow: 'black 3px 5px 3px',
    },
    toggle_exclusive: 0,
    clipboard2: null,
    pagesList: [{pageNo: 1, pageName: 'Page1'}],
    pageSel: {pageNo: 1, pageName: 'Page1'},  
  }),

  mounted() {                
    this.getMyCanvas()
    this.getPagesList()
    window.addEventListener('resize', this.getScreenSize)
    this.getScreenSize()
  },

  destroyed() {
    window.removeEventListener('resize', this.getScreenSize)
  },

  methods: {
    getScreenSize() {
      this.canvasWidth = window.innerWidth
      this.canvasHeight = window.innerHeight
      if (this.myCanvas) {
        this.myCanvas.setWidth(this.canvasWidth)
        this.myCanvas.setHeight(this.canvasHeight)
      }
    },

    getMyCanvas() {
      this.myCanvas = new fabric.Canvas('canvasFrame', {
        stopContextMenu: true,
        fireRightClick: true
      })
      this.myCanvas.on("mouse:down", this.mousedownEvent)
      this.myCanvas.on("mouse:move", this.mousemoveEvent)    
      this.myCanvas.on("mouse:up", this.mouseupEvent)    
      this.myCanvas.on("mouse:out", this.mouseoutEvent)      
    },

    // --- LOGICA DI SALVATAGGIO SENZA CRYPTO ---

    saveCanvasData(pageObj) {                  
      const key = `MyCanvas_${pageObj.pageNo}_${pageObj.pageName}`
      const jsonData = JSON.stringify(this.myCanvas.toJSON())
      localforage.setItem(key, jsonData)
        .then(() => console.log('Salvato:', key))
        .catch(err => console.error(err))
    },

    getCanvasData(pageObj) {
      const key = `MyCanvas_${pageObj.pageNo}_${pageObj.pageName}`
      localforage.getItem(key).then(value => { 
        if (value) { 
          this.myCanvas.loadFromJSON(value, () => {
            this.myCanvas.renderAll()
          })
        } else {
          this.cleanCanvas()
        }
      }).catch(err => console.log(err))          
    },

    getPagesList() {  
      const that = this
      localforage.keys().then(keys => {
        const filtered = keys.filter(k => k.startsWith('MyCanvas_'))
        if (filtered.length > 0) {
          that.pagesList = filtered.map(k => {
            const parts = k.split('_')
            return { pageNo: parts[1], pageName: parts[2] }
          })
          that.pageSel = that.pagesList[0]
          that.getCanvasData(that.pageSel)
        }
      })
    },

    // --- GESTIONE STRUMENTI E DISEGNO ---

    handMode(i) {
      this.drawType = i
      this.myCanvas.isDrawingMode = (i === 1 || i === 14 || i === 15)
      this.myCanvas.selection = (i === 0)
      this.myCanvas.skipTargetFind = (i !== 0)

      if (this.myCanvas.isDrawingMode) {
        this.setDrawType(i)
      }

      if (i === 20) this.unDo()
      if (i === 21) this.reDo()
    },

    setDrawType(type) {
      if (type === 1) this.myCanvas.freeDrawingBrush = new fabric.PencilBrush(this.myCanvas)
      if (type === 14) this.myCanvas.freeDrawingBrush = new fabric.SprayBrush(this.myCanvas)
      if (type === 15) {
        const b = new fabric.PencilBrush(this.myCanvas)
        b.width = 10
        this.myCanvas.freeDrawingBrush = b
      }
      this.myCanvas.freeDrawingBrush.color = this.penColor
      this.myCanvas.freeDrawingBrush.width = parseInt(this.penWidth)
    },

    mousedownEvent(e) {
      this.mouseEventFlag = true        
      this.lastPointer = this.myCanvas.getPointer(e.e)
      this.drawingObj = null        
    },

    mousemoveEvent(e) {
      if (!this.mouseEventFlag || this.myCanvas.isDrawingMode) return
      const pointer = this.myCanvas.getPointer(e.e)
      let newObj = null

      switch (this.drawType) {
        case 3: newObj = this.drawNoteRect(pointer); break
        case 4: newObj = this.drawRect(pointer); break
        case 5: newObj = this.drawCircle(pointer); break
        case 8: newObj = this.drawLine(pointer); break
        case 10: newObj = this.drawArrow(pointer); break
        case 13: newObj = this.drawText(pointer); break
        case 19: {
          const delta = new fabric.Point(e.e.movementX, e.e.movementY)
          this.myCanvas.relativePan(delta)
          return
        }

      }

      if (this.drawingObj) this.myCanvas.remove(this.drawingObj)
      if (newObj) {
        this.myCanvas.add(newObj)
        this.drawingObj = newObj          
      }    
    },

    mouseupEvent() {
      this.mouseEventFlag = false
      if (this.drawingObj) {
        this.drawingObj.setCoords()
        this.saveCanvasData(this.pageSel)
      }
    },

    mouseoutEvent() {
      this.saveCanvasData(this.pageSel)
    },

    // --- FUNZIONI OGGETTI ---

    drawRect(p) {
      return new fabric.Rect({
        left: Math.min(this.lastPointer.x, p.x),
        top: Math.min(this.lastPointer.y, p.y),
        width: Math.abs(p.x - this.lastPointer.x),
        height: Math.abs(p.y - this.lastPointer.y),
        fill: this.fillFlag ? this.penColor : 'transparent',
        stroke: this.penColor,
        strokeWidth: this.penWidth
      })
    },

    drawText(p) {
      return new fabric.IText('Testo', {
        left: p.x,
        top: p.y,
        fontFamily: 'Arial',
        fill: this.fontColor,
        fontSize: 20
      })
    },

    insertImageFile(file) {
      if (!file) return
      const reader = new FileReader()
      reader.onload = (f) => {
        fabric.Image.fromURL(f.target.result, (img) => {
          img.scaleToWidth(200)
          this.myCanvas.add(img)
          this.myCanvas.renderAll()
        })
      }
      reader.readAsDataURL(file)
    },

    setPenColor(c) {
      this.penColor = c
      if (this.myCanvas.freeDrawingBrush) this.myCanvas.freeDrawingBrush.color = c
      const active = this.myCanvas.getActiveObject()
      if (active) {
        active.set('stroke', c)
        if (this.fillFlag) active.set('fill', c)
        this.myCanvas.renderAll()
      }
    },

    setPenWidth(w) {
      this.penWidth = w
      if (this.myCanvas.freeDrawingBrush) this.myCanvas.freeDrawingBrush.width = parseInt(w)
    },

    doRemove() {
      this.myCanvas.getActiveObjects().forEach(obj => this.myCanvas.remove(obj))
      this.myCanvas.discardActiveObject().renderAll()
      this.saveCanvasData(this.pageSel)
    },

    cleanCanvas() {
      this.myCanvas.clear()
    },

    // --- STUB METODI MANCANTI (da implementare se serve logica complessa) ---
    doNewPage() { this.cleanCanvas(); const n = this.pagesList.length + 1; this.pagesList.push({pageNo: n, pageName: 'Page'+n}) },
    doChangePage() { this.getCanvasData(this.pageSel) },
    zoomOutIn() { /* logica zoom */ },
    doUndo() { /* logica undo */ }
  }
}
</script>