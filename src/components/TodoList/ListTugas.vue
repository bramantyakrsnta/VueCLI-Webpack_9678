<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List Tugas</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>

                <v-spacer></v-spacer>

                <v-select
                    v-model="sorting"
                    @change="sortingPriority(sorting)"
                    :items="['Penting', 'Tidak Penting']"
                    label="Urutkan"
                    outlined
                    hide-details
                    class="mr-4 mt-4">
                </v-select>

                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>

            <v-data-table 
                :headers="headers" 
                :items="todos" 
                :search="search"
                :single-expand="singleExpand"
                :expanded.sync="expanded"
                item-key="note"
                show-expand
                class="elevation-1">
                    <template v-slot:[`item.priority`]="{ item }">
                        <td>
                            <v-chip v-if="item.priority == 'Penting'" color="red" outlined class="rounded-lg">
                                {{ item.priority }}
                            </v-chip>
                            <v-chip v-else-if="item.priority == 'Biasa'" color="green" outlined class="rounded-lg">
                                {{ item.priority }}
                            </v-chip>
                            <v-chip v-else color="blue" outlined class="rounded-lg">
                                {{ item.priority }}
                            </v-chip>
                        </td>
                    </template>
                
                <template v-slot:[`item.actions`]="{ item }">                   
                    <v-icon color="blue" @click="editItem(item)">mdi-pencil</v-icon>
                    <v-icon color="red" @click="deleteItem(item)">mdi-delete</v-icon>
                </template>

                <template v-slot:[`item.checklist`]="{ item }">                                   
                    <v-checkbox multiple :key="item" @click.capture.stop="itemChecked(item)"/>  
                </template>

                <template v-slot:top>
                    <v-toolbar flat>
                        <v-toolbar-title>Expandable Table</v-toolbar-title>
                            <v-spacer></v-spacer>
                            <v-switch
                                v-model="singleExpand"
                                label="Single expand"
                                class="mt-auto"
                            ></v-switch>
                    </v-toolbar>
                </template>
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length" class="text-justify">
                        <br>
                        <h2> Note : </h2>
                        <br>
                        {{ item.note }}
                        <br>
                        <br>
                    </td>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="550px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required>
                        </v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak Penting']"
                            label="Priority"
                            required>
                        </v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required>
                        </v-textarea>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="red darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn v-if="cekEdit == true" color="blue darken-1" text @click="save">
                        Tambah
                    </v-btn>
                    <v-btn v-else color="blue darken-1" text @click="edited(formTodo)">
                        Update
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteDialog" persistent max-width="400px">
            <v-card>
                <v-card-title > 
                    <span class="headline font-weight-bold">Yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="cancel">
                        Tidak
                    </v-btn>
                    <v-btn color="red darken-1" text @click="deleted">
                        Ya
                    </v-btn>                    
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-card v-if="tempCheck.length" class="mt-3 text-left">
            <v-card-title>Delete Multiple</v-card-title>
            <v-card-text>
                <v-list-item v-for="(item, index) in tempCheck" :key="index">                    
                        <v-list-item-title>-- {{item.task}}</v-list-item-title>                    
                </v-list-item>
            </v-card-text>
            <v-card-action class="ml-4">
                <v-btn color="error" class="mb-4" dark @click="deleteChecked">
                    Hapus Semua
                </v-btn>     
            </v-card-action>                                  
        </v-card>

    </v-main>
</template>


<script>
    export default {
        name: "List",
        data() {
            return {
                search: null,
                dialog: false,
                deleteDialog: false,
                detailDialog: false,
                itemTemp: null,
                cekEdit: true,
                expanded: [],
                singleExpand: false,
                sorting: null,
                tempCheck: [],
                headers: [
                    {
                        text: "Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    { text: "Priority", value: "priority" },                    
                    { text: "Actions", value: "actions" },
                    { text: "", value: "checklist" },
                ],

                todos: [
                    {
                        task: "bernafas",
                        priority: "Penting",
                        note: "huffttt",
                    },
                    {
                        task: "nongkrong",
                        priority: "Tidak Penting",
                        note: "bersama tman2",
                    },
                    {
                        task: "masak",
                        priority: "Biasa",
                        note: "masak air 500ml",
                    },
                ],

                formTodo: {
                    task: null,
                    priority: null,
                    note: null,
                },

                detail: {
                    task: null,
                    priority: null,
                    note: null,
                },
            };
        },

        methods: {
            save() {
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog = false;
            },
            cancel() {
                this.resetForm();
                this.dialog = false;
                this.deleteDialog = false;
                this.detailDialog = false;
                this.cekEdit = true;
                this.itemTemp = null;
            },
            resetForm() {
                this.formTodo = {
                    task: null,
                    priority: null,
                    note: null,
                };
            },

            editItem(item) {
                this.cekEdit = false;
                this.dialog = true;
                this.itemTemp = item;
                this.formTodo = {
                    task : item.task,
                    priority : item.priority,
                    note : item.note,
                };
            },

            edited(formTodo) {
                this.itemTemp.task = formTodo.task;
                this.itemTemp.priority = formTodo.priority;
                this.itemTemp.note = formTodo.note;
                this.cancel();            
            },

            deleteItem(item) {
                this.deleteDialog = true;
                this.itemTemp = item;
            },

            deleted() {
                this.todos.splice(this.todos.indexOf(this.itemTemp), 1);
                this.dialogdel = false;
                this.cancel();
            },

            sortingPriority: function(sorting) {
                function compare(a,b) {                          
                      if(sorting == "Tidak Penting"){           
                        if (a.priority == "Tidak Penting")
                            return -1;                     
                        if (a.priority == "Penting")
                            return 1;                                           
                        if (a.priority == "Biasa" && b.priority=="Tidak Penting")
                            return 1;                       
                        if (a.priority == "Biasa" && b.priority=="Penting")
                            return 0;                                                                                                    
                      }
                      if(sorting == "Penting"){
                        if (a.priority == "Penting")
                            return -1;
                        if (a.priority == "Tidak Penting")
                            return 1;                                       
                        if (a.priority == "Biasa" && b.priority=="Penting")
                            return 1;                       
                        if (a.priority == "Biasa" && b.priority=="Tidak Penting")
                            return 0;
                      }                                            
                }
                return this.todos.sort(compare);
            },

            itemChecked(item){
                if(this.tempCheck.includes(item)) {
                    this.tempCheck.splice(this.tempCheck.indexOf(item), 1);
                } else {
                    this.tempCheck.push(item);                
                }
            },
            
            deleteChecked: function(){             
                for(var i = 0; i<this.tempCheck.length; i++){            
                    this.todos.splice( this.todos.indexOf(this.tempCheck[i]), 1);
                }
                this.tempCheck=[];                            
            },
        },
    };
</script>