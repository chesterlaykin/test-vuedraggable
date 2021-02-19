<template>
    <div>
        <div class="wrap">
            <h2>Draggable test</h2>

            <p>Select scenario</p>
            <button type="button" class="btn btn1" @click="loadScenario('scenario_1')">Scenario 1</button>
            <button type="button" class="btn btn1" @click="loadScenario('scenario_2')">Scenario 2</button>
            <button type="button" class="btn btn1" @click="loadScenario('scenario_3')">Scenario 3</button>

            <p style="font-weight:bold" v-if="scenario">{{initialData[scenario].desc}}</p>
            <div style="height:130px;margin-bottom:22px;">
                <div v-if="addItemWindow" class="additem">
                    <p>Add item</p>
                    <input ref="newitem" @keyup.enter="addItem" type="text" v-model="newItem.textContent">
                    <button type="button" 
                        :disabled="!newItem.textContent.length" 
                        class="btn btn1" 
                        @click="addItem"
                        
                    >Add</button>
                </div>
            </div>
            <div v-if="scenario" class="text-l font-sm" style="width:200px;">
                <ul>
                    <li>Drag to change order. </li>
                    <li> Add item - does it end up in the right group?</li>
                </ul>
               </div>
            <draggable v-model="form.dataGroups" 
                @start="drag=true" 
                @end="drag=false"  
                ghost-class="ghost" 
                class="row"
            >
                <div v-for="(dataGroup, dataGroupIndex) in form.dataGroups" 
                    :key="`group_${dataGroupIndex}`" 
                    class="drag-element flex-center"
                    style="display:flex"
                >
                    <!-- add item to the current datagroup - but check the config if it is allowed -->
                    <a v-if="true"   
                        @click="openAddItemWindow(dataGroup.properties.name,dataGroupIndex)" 
                        class="btn btn2"  
                        v-cloak>Add new item
                    </a>

                    <div v-for="(item, idx) in dataGroup.dataGroup" 
                        :key="`item_${idx}`"
                        class="datagroup-item"
                    >
                        <p class="font-sm" >{{item.textContent }}</p>
                        <!-- <p>{{item.textContent}}</p> -->
                    </div>
                </div>

            </draggable>
            <div v-if="errors.length">
                <p style="font-size:12px; font-weight:bold;">Resulting errors</p>
                <div v-for="error in errors" style="margin-top:12px">
                
                    <div class="err">{{error}}</div>
                </div>
            </div>
            
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable'
    export default {
        components: {draggable},
        data() {
            return {
                drag:false,
                addItemWindow:false,
                scenario: null,
                errors: [],
                //This array holds the config settings for the available data groups
                datagroupsConfig: [
                    {
                        name: 'firstGroup',
                        numElements: 3
                    },
                    {
                        name: 'secondGroup',
                        numElements: 1
                    },
                ],
                form: {

                    //dataGroups - Needs to be array since draggable requires it
                    dataGroups: []
                },
                newItem: {
                    groupName: null,
                    groupIndex: null,
                    textContent: '',
                },
                initialData: { 
                    
                    scenario_1: {
                        //scenario_1 - no complications
                        desc: 'scenario 1: all groups have initial content - no complications',
                        dataGroups:  [  
                            {
                                groupName: 'firstGroup',
                                dataGroup: [    
                                    {
                                        groupName: 'firstGroup',
                                        textContent: 'Hello this is an item for first group.'
                                    },
                                    {
                                        groupName: 'firstGroup',
                                        textContent: 'Another item for first group'
                                    }
                                ]
                            },
                            {
                                groupName: 'secondGroup',
                                dataGroup: [
                                    {
                                        groupName: 'secondGroup',
                                        textContent: 'This is an item for second group'
                                    } 
                                ]
                            }
                        ]
                    },
                    scenario_2 : {
                        //no initial content
                        desc: 'scenario 2: no initial content',
                        dataGroups:[]
                    },
                    scenario_3 : {
                        //scenario_3 , no content in second group
                        desc: 'scenario 3: second group have no initial content', 
                        dataGroups:  [  
                            {
                                groupName: 'firstGroup',
                                dataGroup: [                   
                                    {
                                        groupName: 'firstGroup',
                                        textContent: 'Hello this is an item for first group'
                                    },
                                    {
                                        groupName: 'firstGroup',
                                        textContent: 'Another item for first group'
                                    }
                                ]
                            },
                            {
                                groupName: 'secondGroup',
                                dataGroup: [], 
                            }
                            
                        ]
                    }
                }
                
            }
           
        },
        methods: {
            loadScenario(scenario){
                
                this.reset();

                this.addItemWindow = false;
                setTimeout(() => {
                     this.scenario = scenario
                    this.loadInitialData(scenario);
                }, 500);
               
                 
            },
            reset(){
                this.form.dataGroups = []
                this.errors = [];
                this.scenario = null;
                this.newItem =  {
                    groupName: null,
                    textContent: '',
                }
            },
            loadInitialData(){
                let currentScenario = this.initialData[this.scenario];

                //Initialize the data group with one object for each dataGroup defined in datagroupsConfig.
                this.datagroupsConfig.forEach( dataGroupProps => {
 
                    this.form.dataGroups.push({
                        properties: {...dataGroupProps}, 
                        dataGroup : []
                    })
                }) 

                //Fill the data groups with any existing datagroups data
                this.form.dataGroups.forEach( formDataGroup => { 
                    //loop the stored data
                    [...currentScenario.dataGroups].forEach( existingDataGroup => {
                        
                        if(formDataGroup.properties.name === existingDataGroup.groupName ){
                            console.log('match:',existingDataGroup.groupName);
                            //Add data
                            formDataGroup.dataGroup = [...existingDataGroup.dataGroup];
                            
                        } 
                    })
                })
   
            },
            openAddItemWindow(groupName,groupIndex) {
 
                //Set group name
                this.newItem.groupName = groupName
                this.newItem.groupIndex = groupIndex
                
                this.addItemWindow = true;

                this.$nextTick( () => {

                    this.$refs.newitem.focus();
                })
                
            },
            //Add "newItem" to the dataGroup index
            addItem(){
                
                //add item
                this.form.dataGroups[this.newItem.groupIndex].dataGroup.push({
                    groupName: this.newItem.groupName,
                    textContent: this.newItem.textContent
                });
                //reset
                this.newItem =  {
                    groupName: null,
                    groupIndex:null,
                    textContent: '',
                }
                this.addItemWindow = false;
            }
        },
    }
</script>

<style lang="scss" scoped>
   .ghost {
        opacity: 0.5;
        background: rgb(163, 163, 163);
    }
    .drag-element{
        cursor: move;
        padding: 10px;
        border: 1px solid rgb(236, 236, 236);

        &:hover{
            background:rgb(236, 236, 236);
        }
    }
    .datagroup-item{
        min-height:68px;
        max-width: 140px;
        padding:4px;
        margin-right:2px;
        background: rgb(174, 255, 174);
        border: 1px solid black;
    }
    .btn{
        
        padding:7px;
        color:rgb(36, 36, 36);
        border:2px solid black;
        margin-right:2px;
        cursor:pointer;
        &:hover{
            background: rgb(245, 245, 245);
            color:rgb(48, 48, 48);
        }
    }
    .btn1{
        background: rgb(114, 227, 255);
    }
    .btn2{
        background:rgb(165, 255, 243);
        margin-right:22px;
        
    }
    .wrap{
        max-width:1100px;
        margin:0 auto;
    }
    .err{
        color:rgb(253, 171, 171);
        padding:2px;

    }
    .additem{
        background: rgb(229, 249, 255);
        display: inline-block;
        padding: 18px;
     
    }
    .text-l{
        text-align:left;
    }
    .font-sm{
        font-size:13px;
    }
    .flex-center {
    align-items: center;
    justify-content: center;
    }
</style>