<template>
  <v-container fluid>
    <div class="folder-div">      
      <div v-for="(item) in folderItems" :key="item.id">        
        <v-card class="py-1 px-0" :class="[item.sel?'folder-item-sel':'folder-item']" elevation="15" :color="item.color" @click="changePage(item.id)">
          <v-card-text>{{item.id}}</v-card-text>
        </v-card>     
      </div>
    </div>
  
    <div v-if="viewMode == false ">
      <v-card elevation="15" color="#FFF9C4" height="90vh" width="90%" style="position: relative; top: 5px; left: 60px;" light>           
        <div class="main-container">
          <div class="editor-container editor-container_document-editor editor-container_include-minimap editor-container_include-style" ref="editorContainerElement">        
            <div class="editor-container__toolbar" ref="editorToolbarElement"></div>
            <div class="editor-container__minimap-wrapper">
              <div class="editor-container__editor-wrapper">
                <div class="editor-container__editor">
                  <div ref="editorElement">
                    <ckeditor :editor="editor" v-model="editorData" :config="editorConfig" @ready="onReady" @blur="saveWordData(currentPageNo)"></ckeditor> 
                  </div>
                </div>
              </div>          
            </div>
          </div>
        </div>      
      </v-card>
    </div>    

    <div v-else>
      <v-card class="py-2 px-2" elevation="2" width="90%" style="position: relative; top: 5px; left: 60px;">          
        <v-card-title>
          Page {{currentPageNo}}
        </v-card-title>  
        <div v-html="editorData" class="ck-content"></div>
      </v-card>
    </div>  
  
    <div>
      <v-card class="py-2 px-1" elevation="2" width="40px" style="position: fixed; top: -3px; right: -2px; display: flex; flex-direction: column; gap: 10px; align-items: center;"> 
        <v-icon dense @click="doNewFile" title="Nuovo">mdi-file-outline</v-icon>
        <v-icon dense @click="doOpenFile" title="Apri">mdi-folder-open-outline</v-icon>
        <v-icon dense @click="doSaveAsFile" title="Esporta">mdi-share</v-icon>
        <v-icon dense @click="swiftPageViewMode" title="Vista/Edit">mdi-book-open-variant-outline</v-icon>
      </v-card>
    </div>
  </v-container>  
</template>

<script>
import localforage from 'localforage'   
import '../css/style.css'  
import '../css/ck5-style.css'  
// CryptoJS rimosso

import CKEditor from '@ckeditor/ckeditor5-vue2'  
import {
  DecoupledEditor, AccessibilityHelp, Alignment, Autoformat, AutoImage, AutoLink, Autosave, BalloonToolbar, 
  Base64UploadAdapter, BlockQuote, BlockToolbar, Bold, Code, CodeBlock, Essentials, FindAndReplace, 
  FontBackgroundColor, FontColor, FontFamily, FontSize, FullPage, GeneralHtmlSupport, Heading, Highlight, 
  HorizontalLine, HtmlComment, HtmlEmbed, ImageBlock, ImageCaption, ImageInline, ImageInsert, 
  ImageInsertViaUrl, ImageResize, ImageStyle, ImageTextAlternative, ImageToolbar, ImageUpload, Indent, 
  IndentBlock, Italic, Link, LinkImage, List, ListProperties, Markdown, MediaEmbed, Mention, PageBreak, 
  Paragraph, PasteFromMarkdownExperimental, PasteFromOffice, RemoveFormat, SelectAll, ShowBlocks, 
  SourceEditing, SpecialCharacters, SpecialCharactersArrows, SpecialCharactersCurrency, SpecialCharactersEssentials, 
  SpecialCharactersLatin, SpecialCharactersMathematical, SpecialCharactersText, Strikethrough, Style, 
  Subscript, Superscript, Table, TableCaption, TableCellProperties, TableColumnResize, TableProperties, 
  TableToolbar, TextPartLanguage, TextTransformation, Title, TodoList, Underline, Undo
} from 'ckeditor5'

export default {
  name: 'MyWord',
  components: {
    ckeditor: CKEditor.component,
  },
  data: () => ({                
    viewMode: false,
    editor: DecoupledEditor, 
    editorData: '',
    editorConfig: {
      toolbar: {
        items: [
          'undo','redo','|','sourceEditing','showBlocks','findAndReplace','selectAll','textPartLanguage','|',
          'heading','style','|','fontSize','fontFamily','fontColor','fontBackgroundColor','|',
          'bold','italic','underline','strikethrough','subscript','superscript','code','removeFormat','|',
          'specialCharacters','horizontalLine','pageBreak','link','insertImage','insertImageViaUrl','mediaEmbed',
          'insertTable','highlight','blockQuote','codeBlock','htmlEmbed','|','alignment','|',
          'bulletedList','numberedList','todoList','outdent','indent','|','accessibilityHelp'
        ],
        shouldNotGroupWhenFull: false,            
      },
      plugins: [ 
        AccessibilityHelp, Alignment, Autoformat, AutoImage, AutoLink, Autosave, BalloonToolbar, 
        Base64UploadAdapter, BlockQuote, BlockToolbar, Bold, Code, CodeBlock, Essentials, FindAndReplace, 
        FontBackgroundColor, FontColor, FontFamily, FontSize, FullPage, GeneralHtmlSupport, Heading, 
        Highlight, HorizontalLine, HtmlComment, HtmlEmbed, ImageBlock, ImageCaption, ImageInline, 
        ImageInsert, ImageInsertViaUrl, ImageResize, ImageStyle, ImageTextAlternative, ImageToolbar, 
        ImageUpload, Indent, IndentBlock, Italic, Link, LinkImage, List, ListProperties, Markdown, 
        MediaEmbed, Mention, PageBreak, Paragraph, PasteFromMarkdownExperimental, PasteFromOffice, 
        RemoveFormat, SelectAll, ShowBlocks, SourceEditing, SpecialCharacters, SpecialCharactersArrows, 
        SpecialCharactersCurrency, SpecialCharactersEssentials, SpecialCharactersLatin, 
        SpecialCharactersMathematical, SpecialCharactersText, Strikethrough, Style, Subscript, 
        Superscript, Table, TableCaption, TableCellProperties, TableColumnResize, TableProperties, 
        TableToolbar, TextPartLanguage, TextTransformation, Title, TodoList, Underline, Undo 
      ],
      // Altre configurazioni CKEditor...
      fontFamily: { supportAllValues: true },
      fontSize: { options: [10, 12, 14, 'default', 18, 20, 22], supportAllValues: true },
      htmlSupport: { allow: [{ name: /^.*$/, styles: true, attributes: true, classes: true }] },
      image: { toolbar: ['toggleImageCaption','imageTextAlternative','|','imageStyle:inline','imageStyle:wrapText','imageStyle:breakText','|','resizeImage'] },
      table: { contentToolbar: ['tableColumn', 'tableRow', 'mergeTableCells', 'tableProperties', 'tableCellProperties'] },
    },
    currentPageNo: '1',              
    folderItems: [
      {id:'1',color:'#dddddd',sel:true},{id:'2',color:'#aadddd',sel:false},{id:'3',color:'#dddddd',sel:false},
      {id:'4',color:'#eecc00',sel:false},{id:'5',color:'#dddddd',sel:false},{id:'6',color:'#dddddd',sel:false},
      {id:'7',color:'#ddaaaa',sel:false},{id:'8',color:'#dddddd',sel:false},{id:'9',color:'#dddddd',sel:false},
      {id:'A',color:'#888888',sel:false},{id:'B',color:'#8888aa',sel:false},{id:'C',color:'#66dddd',sel:false},
      {id:'D',color:'#dddddd',sel:false},{id:'E',color:'#dddddd',sel:false},{id:'F',color:'#dddddd',sel:false},
      {id:'G',color:'#dddddd',sel:false},{id:'H',color:'#11dd55',sel:false},{id:'I',color:'#dddddd',sel:false},
      {id:'J',color:'#dddddd',sel:false},{id:'K',color:'#9900dd',sel:false},
    ],             
  }),

  mounted() {      
    this.getWordData(this.currentPageNo)      
  },
  
  methods: {
    getWordData(pageNo) {
      localforage.getItem('Word_' + pageNo).then(value => {
        this.editorData = value === null ? '' : value
        this.currentPageNo = pageNo                
      }).catch(err => console.error(err))              
    },

    saveWordData(pageNo) {
      localforage.setItem('Word_' + pageNo, this.editorData)
        .then(() => console.log('Salvataggio automatico completato'))
        .catch(error => console.error(error))
    },

    changePage(pageNo) {                
      this.getWordData(pageNo)            
      this.folderItems.forEach(item => {
        item.sel = (item.id === pageNo)
      })    
    },

    swiftPageViewMode() {
      this.viewMode = !this.viewMode         
    },      
      
    // --- ESPORTAZIONE FILE IN CHIARO ---
    doSaveAsFile() {
      const dataContent = this.editorData
      const blob = new Blob([dataContent], { type: 'text/html' })  
      const dataURL = URL.createObjectURL(blob)    
      const exportFileLink = document.createElement('a')
      exportFileLink.download = `Page_${this.currentPageNo}_export.html`
      exportFileLink.href = dataURL
      document.body.appendChild(exportFileLink)
      exportFileLink.click()
      document.body.removeChild(exportFileLink)
      URL.revokeObjectURL(dataURL)
    },

    // --- IMPORTAZIONE FILE IN CHIARO ---
    doOpenFile() {
      if (!confirm('L\'apertura di un file sovrascriverà la pagina corrente. Confermi?')) return
      
      const that = this
      const inputFile = document.createElement('input')
      inputFile.type = 'file'
      inputFile.accept = '.html,.txt,.123notes_word' // Accetta anche la vecchia estensione
      
      inputFile.addEventListener('change', function(event) {
        const file = event.target.files[0]
        if (!file) return
        
        const reader = new FileReader()    
        reader.onload = function(e) {
          // Carica il contenuto direttamente nell'editor
          that.editorData = e.target.result
          // Salva subito per persistere il cambiamento
          that.saveWordData(that.currentPageNo)
        }
        reader.readAsText(file)
      })
      inputFile.click()
    },

    doNewFile() {
      if (!confirm('Vuoi svuotare il contenuto della pagina corrente?')) return
      this.editorData = ''
      this.saveWordData(this.currentPageNo)
    },

    onReady(editor) {
      // Inserisce la toolbar nel contenitore dedicato
      if (this.$refs.editorToolbarElement) {
        Array.from(this.$refs.editorToolbarElement.children).forEach(child => child.remove())      
        this.$refs.editorToolbarElement.appendChild(editor.ui.view.toolbar.element)      
      }
    }
  }
}
</script>

<style scoped>
/* Aggiungi qui eventuali stili specifici per MyWord */
.folder-div {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-bottom: 10px;
}
.folder-item, .folder-item-sel {
  cursor: pointer;
  min-width: 40px;
  text-align: center;
}
.folder-item-sel {
  border: 2px solid #2196F3 !important;
}
</style>