<template>
<v-touch>
  <div id='todo'>
    <Layout trackertype="todo" back="/" color="#aed8a4" v-bind:title="this.$route.params.title" type="3" style = "font-weight: bold" v-bind:textOnRight="this.editMode?'Save':'Done'" v-bind:onClickRightButton="edit">
      <div class="below-nav-bar wrapper">
        <div v-for="(todo,id) in todos" v-bind:key="id">
          <div style="width:100%; display:flex; flex-flow: row nowrap; justify-content: start; align-items:start;" >
            <input class="circular-checkbox" v-bind:checked="selected[id]" type="checkbox" v-on:change="check(id)"/>
          <v-touch v-on:tap="editItem(id)" v-on:swipeleft="showDeleteButton(id)" v-on:swiperight="cancelDelete" style="width:100%">
            <input v-bind:class="'bottom-border-input todo-item-container todoTextarea'+(selected[id]?' done':'')" v-model="todo.content"/>
            <div class="deleteButton">
            <b-button v-on:click="deleteItem(id)" v-show="deletingItem === id" size="sm" variant="danger">
                Delete
            </b-button>
            </div>
          </v-touch>
          </div>
          <div style="width: 100%; height:0.3em; "/>
        </div>
        <div class="add-icon-placeholder">
          <div v-show="this.editMode" class="add-icon" v-on:click="createTodo"/>
        </div>
        <div v-on:click="createTodo" class=" todo-item-container">
          <input class="bottom-border-input newTodoTextarea" placeholder="Tap to create a new item" v-model="newItem"/>
        </div>
      </div>
    </Layout>
  </div>
  </v-touch>
</template>

<script>
import Layout from './Layout.vue';
import Store from '../js/Store.js'
import Record from '../js/Record.js'
export default {
  name: 'CreateTracker',
  components:{
    Layout,
    
  },
  props: {
    
  },
  data () {
    return {
      selected: [], // Must be an array reference!
      deletingItem: -1,
      todos: [],
      editMode: false,
      creatingNew: false,
      newItem:''
    }
  },
  mounted:function(){
    let title = this.$route.params.title;
    let data = Store.getTracker('todo',title);
    if(data!==undefined){
      this.todos = data.records;
      for(let i=0;i<this.todos.length;i++){
        this.selected[i] = this.todos[i].detail===true?true:false;
      }
      if(data.records.length===0)this.createTodo();
      
    }
    
    this.updateTracker =function(){
      Store.updateTracker('todo',title,'detail',this.selected);
    };

    let todoWindow = document.querySelector('#todo');
      todoWindow.addEventListener('keydown',function(evt){
        if(evt.key==='Enter'){
          let input = document.querySelectorAll('.todoTextarea');
          input.forEach((item)=>{item.blur()})
        } 
      })
  

    let newTodo = document.querySelector('.newTodoTextarea');
    // console.log(newTodo)
    newTodo.addEventListener('keydown',(evt)=>{
      if(evt.key==='Enter'){
        // console.log('xxx')
        // newTodo.blur();
        this.finishCreateTodo();
      } 
    })
    
  },
  beforeDestroy:function(){
    this.finishCreateTodo();
   this.updateTracker();
  },
  computed:{
    
  },
  methods:{
    check: function(id){
      let selected = [...this.selected];
      selected[id] = selected[id]===true?false:true;
      this.selected = selected;
      //vue issue: cannot detect change in v-bind variables of reference type
    },
    deleteItem:function(id){
      this.todos.splice(id,1);
      this.selected.splice(id,1);
      this.deletingItem = -1;
    },
    showDeleteButton: function(id){
      this.deletingItem = id;
    },
    cancelDelete: function(){
      this.deletingItem =-1;
    },
    edit: function(){
      //right button
      this.editMode=!this.editMode;
      if(this.editMode===true){//edit
      if(document.querySelectorAll('.todoTextarea').length>0){
        (document.querySelectorAll('.todoTextarea'))[0].focus();
      }
        this.$router.push({path:`/`});
        
      }else{//done
        (document.querySelectorAll('.todoTextarea')).forEach((item)=>item.blur());
        this.finishCreateTodo();
        this.updateTracker();
        // this.$router.push({path:`/`});
      }
    },
    editItem: function(id){
      //each item
      this.editMode=true;
      document.querySelectorAll('.todoTextarea')[id].focus();
    },
    createTodo: function(){
      this.editMode = true;
      this.creatingNew = true;
      document.querySelector('.newTodoTextarea').focus();
    },
    closeCreateTodo: function(){
      this.editMode = false;
      this.creatingNew = false;
    },
    finishCreateTodo: function(){
      if(this.newItem==='')return;
      this.todos.push(new Record(new Date(),this.newItem, false));
      this.selected.push(false);
      this.newItem = '';
    },
    autoSize:function(){
      let el = this;
      let baseScrollHeight = el.scrollHeight;
      // console.log(el.rows,el.scrollHeight)

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.below-nav-bar{
  padding: 20px;
  text-align: left;
}

.bottom-border-input::before{
  content: " ";
}
.done{
  text-decoration: line-through;
}
.circular-checkbox{
  appearance: none;
  -webkit-appearance: none;
	background-color: white;
	border: 2px solid #aaaaaa;
  width: 1.6em;
  height: 1.5em;
	padding: 0;
	border-radius: 50%;
  margin-right: 0.3em;
	display: inline-block;
	position: relative;
}
.deleteButton{
  display: block;
  position: absolute;
  transform: translateY(-93%) scale(0.9);
  right: 20px;
}
/* .circular-checkbox:active, .circular-checkbox:checked:active {
	box-shadow: 0 1px 2px rgba(0,0,0,0.05), inset 0px 1px 3px rgba(0,0,0,0.1);
} */

.circular-checkbox:checked {
	background-color: #aed8a4;
	border: 2px solid #aed8a4;
	color: white;
  /* box-shadow: 0 1px 2px rgba(0,0,0,0.05), inset 0px -15px 10px -12px rgba(0,0,0,0.05), inset 15px 10px -12px rgba(255,255,255,0.1); */
}
.circular-checkbox:checked:before{
  content: '\2714' ;
  color: white;
  display: block; 
  position: absolute ; 
  top: 50% ; 
  left: 50% ; 
  transform: translateX(-50%) translateY(-50%);
  border-radius: 50% ; 
  /* border: 1px solid white;  */
}

.bottom-border-input{
  width: 100%;
  border:none;
  /* border-bottom: 1px solid #ffb619; */
  background-color: #f0f0f0
}
.todo-item-container{
  width: calc(100% - 40px);
  word-wrap: break-word;
  word-break: break-all;
  height: auto;
  display: inline-block;
  /* float:left; */
}

textarea {
  /* border: none; */
  resize: none;
}

.add-icon-placeholder{
  width: 1.5em; 
  height:1.5em; 
  margin-right: 0.3em;
  position: relative;
  display: inline-block;
}

.add-icon{
  width:100%;
  height:100%;
  border: 2px solid #aaaaaa;
  border-radius: 50%;
  background: white;
}
.add-icon::before{
  content: '';
  color:#aed8a4;
  display: block; 
  position: absolute ; 
  top: 50% ; 
  left: 50% ; 
  transform: translateX(-50%) translateY(-50%);
  font-weight: bold;
}
</style>