<template>
  <div v-if="nested == false" class="component-container">


    <div class="component-container-title">{{structure.propertyLabel}}</div>
    <div class="component-container-input-container">
        <div class="component-container-fake-input no-upper-right-border-radius no-lower-right-border-radius no-upper-border temp-icon-search">          
          <form autocomplete="off" v-on:submit.prevent>
            <div style=" display: flex; height: 100%">
            <!-- <input autocomplete="off" v-bind:value="activeSelect"  type="text" disabled style="width: 95%; border:none; height: 90%; font-size: 1.5em; padding: 0.1em; position: relative; background: none; color: lightgray"> -->
              
              <div v-for="(avl,idx) in activeLookupValue" :key="idx" class="selected-value-container">
                  <span style="padding-right: 0.3em; font-weight: bold">{{avl['http://www.w3.org/2000/01/rdf-schema#label']}}<span class="uncontrolled" v-if="!avl.uri">(uncontrolled)</span></span>
                  <span @click="removeValue(idx)" style="border-left: solid 1px black; padding: 0 0.5em; font-size: 1em; cursor: pointer;">x</span>
              </div>
              <input bfeType="EditSimpleLookupComponent-unnested" ref="lookupInput"  :id="assignedId" autocomplete="off" v-on:blur="blur" v-bind:value="activeValue"  type="text" @focus="autoFocus($event)" @keydown="keyDownEvent($event)" @keyup="keyUpEvent($event)" class="input-single selectable-input">
            </div>
          </form>
        </div>
      <div v-if="displayAutocomplete==true" class="autocomplete-container">
        <ul>
          <li v-for="(item, idx) in displayList" :data-idx="idx" v-bind:key="idx" @click="clickAdd">
              <span v-if="item==activeSelect" :data-idx="idx" class="selected">{{item}}</span>
              <span v-else :data-idx="idx">{{item}}</span>
          </li>
        </ul>
      </div>
    </div>

<!--             <pre>{{JSON.stringify(structure,null,2)}}</pre>
 -->
  </div>


  <div v-else style="position: relative;">


      <div  v-bind:class="['component-container-fake-input no-upper-right-border-radius no-lower-right-border-radius no-upper-border temp-icon-search']" :style="{'background-color': (structure.dynamic) ? 'auto' : 'auto' }">          
        <form autocomplete="off" v-on:submit.prevent style="">
          <div style="">
          <!-- <input autocomplete="off" v-bind:value="activeSelect"  type="text" disabled style="width: 95%; border:none; height: 90%; font-size: 1.5em; padding: 0.1em; position: relative; background: none; color: lightgray"> -->
            <div class="component-nested-container-title component-nested-container-title-simple-lookup" >{{structure.propertyLabel}}</div>            
            <div style="display: flex">
              <div v-for="(avl,idx) in activeLookupValue" :key="idx" class="selected-value-container-nested">
                  <span style="padding-right: 0.3em; font-weight: bold">{{avl['http://www.w3.org/2000/01/rdf-schema#label']}}<span class="uncontrolled" v-if="!avl.uri">(uncontrolled)</span></span>
                  <span @click="removeValue(idx)" style="border-left: solid 1px black; padding: 0 0.5em; font-size: 1em; cursor: pointer;">x</span>
              </div>




              <input bfeType="EditSimpleLookupComponent-nested" ref="lookupInput"  :id="assignedId" autocomplete="off" v-on:blur="blur" v-bind:value="activeValue"  type="text" @focus="autoFocus($event)" @keydown="keyDownEvent($event)"  @keyup="keyUpEvent($event)"  class="input-nested selectable-input">
            </div>              

          </div>
          

        </form>
      </div>
    <div v-if="displayAutocomplete==true" class="autocomplete-container">
      <ul>
        <li v-for="(item, idx) in displayList" :data-idx="idx" v-bind:key="idx" @click="clickAdd">
            <span v-if="item==activeSelect" :data-idx="idx" class="selected">{{item}}</span>
            <span v-else :data-idx="idx">{{item}}</span>
        </li>
      </ul>
    </div>

<!--             <pre>{{JSON.stringify(structure,null,2)}}</pre>
 -->
  </div>



</template>

<script>


import { mapState } from 'vuex'
import uiUtils from "@/lib/uiUtils"


export default {
  name: "EditSimpleLookupComponent",
  props: {
    structure: Object,
    parentStructure: Array,
    nested: Boolean,
    profileName: String,
    activeTemplate: Object,
    ptGuid: String,
    parentStructureObj: Object,
    profileCompoent: String

  },
  data: function() {
    return {
      displayAutocomplete: false,
      uri: this.structure.valueConstraint.useValuesFrom[0],
      displayList: [],
      activeFilter: '',
      activeSelect: '',
      activeValue: '',
      doubleDelete: false,
      activeLookupValue: []


    }
  },
  created: function(){


    let currentUserValue = this.activeProfile.rt[this.profileName].pt[this.profileCompoent].userValue



    let possibleLiteralProperties = ['http://www.w3.org/1999/02/22-rdf-syntax-ns#value', 'http://www.w3.org/2000/01/rdf-schema#label', 'http://id.loc.gov/ontologies/bibframe/code','http://www.loc.gov/mads/rdf/v1#authoritativeLabel']
    

    // find the level we are going to work with, it could be the root level or it 
    // could be like Role and stored as a blank node in the data structure
    // let useValue = null


    if (currentUserValue && currentUserValue['@root'] && currentUserValue['@root'] == this.structure.propertyURI ){


      // it is the root user value, we need to find its URI if it has one and what the label is
      // but the label could be stored all over and the URI might be stored at the label level as well :(

      let foundLabelProperties = possibleLiteralProperties.filter((p) => { return (Object.keys(currentUserValue).indexOf(p) > -1) ? true : false })

      // does it have any data?
      if (currentUserValue['@id'] || foundLabelProperties.length > 0){

        // so it has at least a URI or a label property

        // try to find the label

        let label = null
        let labelGuid = null
        let uri = null
        let uriGuid = null

        for (let p of foundLabelProperties){

          if (currentUserValue[p]){
            for (let value of currentUserValue[p]){
              if (value[p]){
                label = value[p]
                labelGuid = value['@guid']
              }
              // it could also store the URI in the #value or something
              if (value['@id']){
                uri= value['@id']
                uriGuid = value['@guid']
              }
            }
          }
        }

        if (currentUserValue['@id'] && uri &&  currentUserValue['@id'] != uri){
          console.warn('---------------------------------------------')
          console.warn('There is a URI at the root level and also in the label? Which to use?')
          console.warn(currentUserValue)
          console.warn(this.structure)
          console.warn('---------------------------------------------')
        }

        if (currentUserValue['@id']){
          uri = currentUserValue['@id']
          uriGuid = currentUserValue['@guid']
        }


        if (!label){

          // no label was found, try to make one from the URI
          if (uri){
            label = uri.split('/').slice(-1)[0]
          }

        }

        // we just use rdf:label internally here, but it could be any of the label properties above
        // we keep the guid so the predicate doesn't really matter
        this.activeLookupValue.push({
          'http://www.w3.org/2000/01/rdf-schema#label' : label,
          uri : uri,
          uriGuid: uriGuid,
          labelGuid: labelGuid

        })


      }

      


    }else if (this.parentStructureObj && currentUserValue[this.parentStructureObj.propertyURI]){




        for (let childProperty of currentUserValue[this.parentStructureObj.propertyURI]){

          let label = null
          let labelGuid = null
          let uri = null
          let uriGuid = null


          let foundLabelProperties = possibleLiteralProperties.filter((p) => { return (Object.keys(childProperty).indexOf(p) > -1) ? true : false })

          // likely just stored at the first level
          if (childProperty['@id']){
            uri = childProperty['@id']
            uriGuid = childProperty['@guid']
          }

          for (let p of foundLabelProperties){

            if (childProperty[p]){
              for (let value of childProperty[p]){
                if (value[p]){
                  label = value[p]
                  labelGuid = value['@guid']
                }
                // it could also store the URI in the #value or something
                if (value['@id']){
                  uri= value['@id']
                  uriGuid = value['@guid']
                }
              }
            }
          }

          if (!label){

            // no label was found, try to make one from the URI
            if (uri){
              label = uri.split('/').slice(-1)[0]
            }

          }

          this.activeLookupValue.push({
            'http://www.w3.org/2000/01/rdf-schema#label' : label,
            uri : uri,
            uriGuid: uriGuid,
            labelGuid: labelGuid

          })

        }


    }else if (this.parentStructureObj && currentUserValue['@root'] && currentUserValue['@root'] == this.parentStructureObj.propertyURI && currentUserValue[this.structure.propertyURI]) {


      // its at the first level


      for (let childProperty of currentUserValue[this.structure.propertyURI]){

        let label = null
        let labelGuid = null
        let uri = null
        let uriGuid = null


        let foundLabelProperties = possibleLiteralProperties.filter((p) => { return (Object.keys(childProperty).indexOf(p) > -1) ? true : false })

        // likely just stored at the first level
        if (childProperty['@id']){
          uri = childProperty['@id']
          uriGuid = childProperty['@guid']
        }

        for (let p of foundLabelProperties){

          if (childProperty[p]){
            for (let value of childProperty[p]){
              if (value[p]){
                label = value[p]
                labelGuid = value['@guid']
              }
              // it could also store the URI in the #value or something
              if (value['@id']){
                uri= value['@id']
                uriGuid = value['@guid']
              }
            }
          }
        }

        if (!label){

          // no label was found, try to make one from the URI
          if (uri){
            label = uri.split('/').slice(-1)[0]
          }

        }

        this.activeLookupValue.push({
          'http://www.w3.org/2000/01/rdf-schema#label' : label,
          uri : uri,
          uriGuid: uriGuid,
          labelGuid: labelGuid

        })

      }


    }else{

      // this.activeLookupValue.push({
      //   'http://www.w3.org/2000/01/rdf-schema#label' : 'No DATA'
      // })

    }
    
    



  },

  computed: mapState({
    lookupLibrary: 'lookupLibrary',
    activeInput: 'activeInput',
    activeProfile: 'activeProfile',
    assignedId (){
      return uiUtils.assignID(this.structure,this.parentStructure)
    },  
    // to access local state with `this`, a normal function must be used
    lookupVocab (state) {
      // let uri = this.structure.valueConstraint.useValuesFrom[0]

      // let returnVal = []
      // Object.keys(state.lookupLibrary).forEach((s)=>{
      
      // })
      
      // if (state.lookupLibrary[this.structure.valueConstraint.useValuesFrom[0]]){
      
      //   return state.lookupLibrary[this.structure.valueConstraint.useValuesFrom[0]]
      // }else{
      //   return []
      // }
      return state.lookupLibrary[this.structure.valueConstraint.useValuesFrom[0]]

      
    }
  }),  
  methods:{

    fakeContainerFocus: function(event){

        // 

        return event
    },

    removeValue: function(idx){


      let toRemove = this.activeLookupValue.splice(idx,1)
      toRemove = toRemove[0]
      console.log(toRemove)


      this.$store.dispatch("removeValueSimple", { self: this, idGuid: toRemove.uriGuid, labelGuid: toRemove.labelGuid }).then(() => {
       
      })  



      // if (this.activeLookupValue.length>1){
      //   this.activeLookupValue.splice(-1,1)
      // }else{
      //   this.activeLookupValue = []
      // }

    },  

    // Takes the list of values from this lookup uri and filters it based on the input
  
    filter: function(){

      this.displayList = []
      this.activeSelect = ''
      Object.keys(this.lookupLibrary[this.uri]).forEach((v)=>{

        // the list has a special key metdata that contains more info
        if (v==='metadata'){return false}

        // no filter yet show first 25
        if (this.activeFilter.trim()===''){
          this.lookupLibrary[this.uri][v].forEach((x)=>{
            // if (this.displayList.length<=25){
              if (this.displayList.indexOf(x)==-1){
                this.displayList.push(x)  
              }
            // }
          })          
        }else{

          // loop through each one, each is a array, so each element of array
          this.lookupLibrary[this.uri][v].forEach((x)=>{
            
            // simple includes value check
            if (x.toLowerCase().startsWith(this.activeFilter.toLowerCase())){
                if (this.displayList.indexOf(x)==-1){
                  this.displayList.push(x)    
                }
            }       

            if (x.toLowerCase().includes(' (' +this.activeFilter.toLowerCase())){
                if (this.displayList.indexOf(x)==-1){
                  this.displayList.push(x)    
                }
            }  



          })
        }


      })



      this.displayList.sort()

      
      // take the first hit and make it the autocomplete text
      if (this.displayList.length>0 && this.activeFilter.length>0){
        this.activeSelect = this.displayList[0]
        this.displayAutocomplete = true
      }
      if (this.displayList.length==0){
        this.displayList.push('No Match - Press [Enter] to add uncontrolled value')
        this.displayAutocomplete = true
      }
      if (this.activeFilter.length==0){
        this.displayAutocomplete = true
      }

      if (this.displayAutocomplete){        
        this.$store.dispatch("disableMacroNav")
      }else{
        this.$store.dispatch("enableMacroNav")
      }

    },
    autoFocus: function(event){

      // let the rest of the app know what is the active input right now
      this.$store.dispatch("setActiveInput", { self: this, id: event.target.id, profileCompoent: this.profileCompoent, profileName: this.profileName }).then(()=>{

        // now focus the side bars
        uiUtils.focusSidebars()


      })
      // assing the input value to the filter value
      this.activeFilter = event.target.value;
      // tell the store to load this specific lookup table into memory
      this.$store.dispatch("fetchLookupValues", { self: this, url: this.structure.valueConstraint.useValuesFrom[0] }).then(() => {
        
        // if there is already a value don't open up the full list, they can type ahead but dont open everything
        // if (this.activeLookupValue.length==0){
          // this.filter()
        // }
      })    
      
      
      

    },
    keyUpEvent: function(event){


      if (event && event.key && (event.key!=='ArrowUp' && event.key !=='ArrowDown' && event.key!=='Escape' && event.key!=='Backspace' && event.key!=='Enter' && event.key!=='PageUp' && event.key!=='PageDown' && event.ctrlKey == false)){
        

        // if we already have a value, do not let it add another
        // if (this.activeLookupValue != null){
        //   event.target.value = ''
        //   event.preventDefault()
        //   return false
        // }



        this.activeValue = event.target.value.trimStart()
        this.doubleDelete = false
        this.activeValue = event.target.value.trimStart()
        this.activeFilter = event.target.value.trimStart()
        this.displayAutocomplete = true
        this.$store.dispatch("disableMacroNav")
        this.filter()




      }





    },    
    keyDownEvent: function(event){
      

      this.activeValue = event.target.value

      if (event && event.key && this.displayAutocomplete == true && (event.key==='ArrowUp' || event.key==='ArrowDown')){
        this.doubleDelete = false
        if (this.displayList.length<=1){
          return false
        }
        if (!this.displayAutocomplete) this.displayAutocomplete = true

        this.activeFilter =''
        this.activeValue = ''
        // if there is nothing selected yet then pick the first one
        if (this.activeSelect.trim()=='' && this.displayList.length>0){
          this.activeSelect = this.displayList[0]
          this.activeValue = this.displayList[0]


        }else{

          // check if there is one further from the actively selected item
          for (let step=0; step<this.displayList.length;step++){
            if (this.displayList[step]===this.activeSelect){

              if (event.key==='ArrowDown'){
                if (step+1 < this.displayList.length){
                  this.activeSelect = this.displayList[step+1]
                  this.activeValue = this.displayList[step+1]
                  break
                }
              }
              if (event.key==='ArrowUp'){
                if (step-1 >= 0){
                  this.activeSelect = this.displayList[step-1]
                  this.activeValue = this.displayList[step-1]
                  break
                }
              }

            }

          }
        }

        return false
      // }else if (event && event.key && this.displayAutocomplete == false && (event.key==='ArrowUp' || event.key==='ArrowDown')){

      }else if (event && event.key && event.key==='Escape'){
        this.doubleDelete = false
        this.activeFilter = ''
        this.activeValue = ''
        this.displayAutocomplete = false
      
      }else if (event && event.key && event.key==='Backspace'){

        if (!this.doubleDelete && this.activeValue === ''){
          this.doubleDelete = true
          return false
        }

        if (this.activeValue == '' && this.doubleDelete){
          this.doubleDelete = false
          // were gonna delete the last one          
          if (this.activeLookupValue.length>0){                       
            this.removeValue(-1)
          }
          
          this.doubleDelete = false
          this.displayAutocomplete = false
        }else if (this.activeValue == ''){
          this.activeValue
        }

      }else if (event && event.key && event.key==='Enter'){
        this.doubleDelete = false

        let metadata = this.lookupLibrary[this.uri].metadata.values

        // find the active selected in the data
        Object.keys(metadata).forEach((key)=>{

          let idx = metadata[key].displayLabel.indexOf(this.activeSelect)
          if (idx >-1){
            // this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':metadata[key].label[idx],URI:metadata[key].uri})
            this.activeFilter = ''
            this.activeValue = ''
            this.activeSelect = ''
            this.displayAutocomplete=false
            event.target.value = ''
            // this.$store.dispatch("addValueLiteral", { self: this, profileComponet: this.profileCompoent, structure: this.structure, template:this.activeTemplate, value:this.activeLookupValue }).then(() => {
             
            // })               
            let parentURI = (this.parentStructureObj) ? this.parentStructureObj.propertyURI : null 


            this.$store.dispatch("setValueSimple", { self: this, ptGuid: this.ptGuid, parentURI: parentURI, URI: this.structure.propertyURI, valueURI: metadata[key].uri, valueLabel:metadata[key].label[idx]}).then((resultData) => {
              this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':resultData.valueLabel, uri: resultData.valueURI, uriGuid: resultData.guid, labelGuid:resultData.guid})
            })


          }
          // let data = this.lookupLibrary[this.uri].metadata[v]
          
          // let idx = data.defaultsisplayLabel.indexOf(this.activeSelect)
          // if (idx > -1){
          //   this.structure.valueConstraint.defaults.push({defaultLiteral:data.label[idx],defaultURI:data.uri[idx]})
          // }
        })


        if (event.target.value == ''){
          this.submitField()
        }

        // if there is a value still that means the value did not match a item in the list
        // so add the value as a uncontrolled value
        if (event.target.value !== ''){  
          
          this.activeFilter = ''
          this.activeValue = ''
          this.activeSelect = ''
          this.displayAutocomplete=false


          let parentURI = (this.parentStructureObj) ? this.parentStructureObj.propertyURI : null 

          this.$store.dispatch("setValueSimple", { self: this, ptGuid: this.ptGuid, parentURI: parentURI, URI: this.structure.propertyURI, valueURI: null, valueLabel:event.target.value}).then((resultData) => {
            this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':resultData.valueLabel, uriGuid: resultData.guid, labelGuid:resultData.guid})
          })
  
          this.submitField()

        }


        event.preventDefault()


      }else if (event.target.value == ''){


            this.activeFilter = ''
            this.activeValue = ''
            this.activeSelect = ''
            this.displayAutocomplete=false



      }


      if (this.displayAutocomplete){        
        this.$store.dispatch("disableMacroNav")
      }else{
        this.$store.dispatch("enableMacroNav")
      }




    },

    blur: function(){

     // we want to hide the menu on deblur but not if they just click an item in the list

     this.$store.dispatch("enableMacroNav") 
     setTimeout(()=>{ 
      this.displayAutocomplete=false 
      this.$store.dispatch("enableMacroNav") 
    },500)


    },

    clickAdd: function(event){

      this.displayAutocomplete=false

      let label = this.displayList[event.target.dataset.idx]

      let metadata = this.lookupLibrary[this.uri].metadata.values

      // find the active selected in the data
      Object.keys(metadata).forEach((key)=>{

        let idx = metadata[key].displayLabel.indexOf(label)
        if (idx >-1){
          // this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':,URI:})
          // this.activeFilter = ''
          // this.activeValue = ''
          // this.activeSelect = ''
          // this.displayAutocomplete=false
          // // this.$store.dispatch("addValueLiteral", { self: this, profileComponet: this.profileCompoent, structure: this.structure, template:this.activeTemplate, value:this.activeLookupValue }).then(() => {
           
          // // })        

          // this.$store.dispatch("setValueSimple", { self: this, ptGuid: this.ptGuid, parentURI: this.parentStructureObj.propertyURI, URI: this.structure.propertyURI, valueURI: metadata[key].uri, valueLabel:metadata[key].label[idx]}).then((resultData) => {
          //   this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':resultData.valueLabel, uri: resultData.valueURI, uriGuid: resultData.guid, labelGuid:resultData.guid})
          // })



          this.activeFilter = ''
          this.activeValue = ''
          this.activeSelect = ''
          this.displayAutocomplete=false
          event.target.value = ''
          // this.$store.dispatch("addValueLiteral", { self: this, profileComponet: this.profileCompoent, structure: this.structure, template:this.activeTemplate, value:this.activeLookupValue }).then(() => {
           
          // })               
          let parentURI = (this.parentStructureObj) ? this.parentStructureObj.propertyURI : null 


          this.$store.dispatch("setValueSimple", { self: this, ptGuid: this.ptGuid, parentURI: parentURI, URI: this.structure.propertyURI, valueURI: metadata[key].uri, valueLabel:metadata[key].label[idx]}).then((resultData) => {
            this.activeLookupValue.push({'http://www.w3.org/2000/01/rdf-schema#label':resultData.valueLabel, uri: resultData.valueURI, uriGuid: resultData.guid, labelGuid:resultData.guid})
          })



          this.$store.dispatch("enableMacroNav")    



        }


      })

      // refocus
      this.$refs.lookupInput.focus()



    },

    submitField: uiUtils.globalMoveDown  


  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

    <!-- <pre>{{JSON.stringify(structure,null,2)}}</pre> -->

<style scoped>

.input-single{
  width: 95%;
  border:none;
  font-size: 1.5em;
  min-height: 2em;
  max-height: 2em;  
  background:none;
}
.input-nested{
  width: 95%;
  border: none;
  font-size: 1.5em;
  padding: 0.1em;
  background: none;

}
.component-nested-container-title-simple-lookup{
  /*top: -4.5em;*/
}

li::before{
  content: '';
}

li{
  padding:0.1em;
}

li span{
  padding:0.1em;
}
input{
  outline: none;
}
.uncontrolled{
  margin-left: 0.25em;
  color: darkred;
}
.component-container-fake-input:focus-within {
  border: solid 1px #a6acb7;
  background-color: #dfe5f1;

  
}
.selected-value-container{
  margin: 0.95em;
  border: solid 1px;
  border-radius: 0.5em;
  padding: 0.35em;
  font-size: 0.75em;
  background-color: whitesmoke;
  white-space: nowrap;

}
.selected-value-container-nested{
  margin: 0.25em;
  border: solid 1px;
  border-radius: 0.5em;
  padding: 0.35em;
  font-size: 0.75em;
  background-color: whitesmoke;
  white-space: nowrap;
  display: inline;
}
.selected{
  border:solid 4px lightblue;
  border-radius: 5px;
}
.autocomplete-container{
  padding: 0.45em;
  position: absolute;
  z-index: 100;
  background-color: white;
  border-radius: 15px;
  -webkit-box-shadow: 0px 5px 7px -1px rgba(150,150,150,1);
  -moz-box-shadow: 0px 5px 7px -1px rgba(150,150,150,1);
  box-shadow: 0px 5px 7px -1px rgba(150,150,150,1);  
}
.autocomplete-container li{
  cursor: pointer;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 0 10px;
}
a {
  color: #42b983;
}
form{
  height: 100%;
}
</style>
