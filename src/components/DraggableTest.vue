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
                    <input type="text" v-model="newItem.textContent">
                    <button type="button" :disabled="!newItem.textContent.length" class="btn btn1" @click="addItem">Add</button>
                </div>
            </div>
            <div v-if="scenario" class="text-l font-sm" style="width:200px;">
                <ul>
                    <li>Drag to change order. </li>
                    <li> Add item - see that it ends up in the right group</li>
                </ul>
               </div>
            <draggable v-model="form.dataGroups" 
                @start="drag=true" 
                @end="drag=false"  
                ghost-class="ghost" 
                class="row"
            >
                <div v-for="(boxGroup, idx) in form.dataGroups" 
                    :key="`group_${idx}`" 
                    class="drag-element flex-center"
                    style="display:flex"
                >
                    <!-- add item to the current datagroup - but check the config if it is allowed -->
                    <a v-if="true"   
                        @click="openAddItemWindow(idx)" 
                        class="btn btn2"  
                        v-cloak>Add new item
                    </a>

                    <div v-for="(item, idx) in boxGroup" 
                        :key="`item_${idx}`"
                        class="boxgroup-item"
                    >
                        <p class="font-sm" >{{item.textContent }}</p>
                        <p>{{boxGroup.textContent}}</p>
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
                    dataGroups: [
 
                    ]
                },
                newItem: {
                    groupName: null,
                    textContent: '',
                },
                initialData: { 
                    
                    scenario_1: {
                        //scenario_1 - no complications
                        desc: 'scenario 1: all groups have initial content - no complications',
                        dataGroups:  [  
                            [                   
                                {
                                    groupName: 'firstGroup',
                                    textContent: 'Hello this is an item for first group.'
                                },
                                {
                                    groupName: 'firstGroup',
                                    textContent: 'Another item for first group'
                                }
                            ],
                            //All 'secondGroup' items goes into this array 
                            [
                                {
                                    groupName: 'secondGroup',
                                    textContent: 'This is an item for second group'
                                }, 
                            ]
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
                            [                   
                                {
                                    groupName: 'firstGroup',
                                    textContent: 'Hello this is an item for first group'
                                },
                                {
                                    groupName: 'firstGroup',
                                    textContent: 'Another item for first group'
                                }
                            ], 
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

                if(currentScenario.dataGroups.length){

                    //Add the datagroups
                    this.form.dataGroups =[...currentScenario.dataGroups];

                    //If any datagroups are missing, we need to add an empty array for that datagroup
                    let numDataGroupsNeeded = this.datagroupsConfig.length; 
          
                    let numDatagroups = currentScenario.dataGroups.length; 
                    numDataGroupsNeeded = numDataGroupsNeeded - numDatagroups;

                    //add a default empty array for every not existing box group 
                   this.initializeMissingDataGroups(numDataGroupsNeeded)
                }else{
                    //No dataGroups content - initialize empty array for each missing data group
                    this.initializeMissingDataGroups(this.datagroupsConfig.length)
                }
            },
            openAddItemWindow(idx) {

                //-------------- Problem --------------- 
                //We need to set the groupName so that we can add the item to that group
                //We don't know the current groupname unless we can check an existing item by the index
                let groupName = this.getGroupNameByGroupIndex(idx);
                if(groupName){
                    this.newItem.groupName = groupName
                }
                
                this.addItemWindow = true;
            },
            initializeMissingDataGroups(num){
                 for (let index = 0; index < num; index++) {
                        this.form.dataGroups.push([])
                        
                    } 
            },
            getGroupNameByGroupIndex(idx){
                if(this.form.dataGroups[idx].length){
                    //get groupname by the existing item's groupName property
                    return this.form.dataGroups[idx][0].groupName
                }else{ 
                     this.errors.push('Can\'t get current group name')
                    return false;
                }
            },
            //Add "newItem" to the dataGroup array it belongs to (determined by its groupName)
            addItem(){
                
                //-------------- Problem --------------- 
                //Each dataGroup array is for a certain group type.
                //The item should go into the array of the group it belongs to
                // but we can't determine this (unless there is an existing item to inspect)

                //try to find existing item to identify the group name and retrieve the array index
                console.log('groupName is ' + this.newItem.groupName);
                let targetIndex = this.form.dataGroups.findIndex( (group,idx) => {
                    if(group.length){
                        
                        if(group[0].groupName == this.newItem.groupName){
                            console.log('groupName match: ' + this.newItem.groupName + ' index: ' + idx);
                            return true;
                        } 
                    } 
                })
                 
                if(targetIndex == -1){
                    //not found. Put in first array
                    console.log('no existing group found');
                    this.errors.push('Can\'t locate group: ' + this.newItem.groupName)
                    targetIndex = 0;

                }else{
                    console.log('targetIndex was found: ', targetIndex);
                }
                //add item
                this.form.dataGroups[targetIndex].push(this.newItem);
                this.newItem =  {
                    groupName: null,
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
    .boxgroup-item{
        min-height:30px;
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