<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

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
                    v-model="formTodo.priority"
                    :items="['Penting', 'Tidak penting']"
                    label="Urutkan"
                    outlined
                    class="mr-4">
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
                    <v-btn small class="mr-2" @click="editItem(item)">
                        EDIT
                    </v-btn>
                    <v-btn small @click="deleteItem(item)">
                        DELETE
                    </v-btn>
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

        <v-dialog v-model="dialog" persistent max-width="600px">
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
                            :items="['Penting', 'Biasa', 'Tidak penting']"
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
                headers: [
                    {
                        text: "Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    { text: "Priority", value: "priority" },                    
                    { text: "Actions", value: "actions" },
                ],

                todos: [
                    {
                        task: "bernafas",
                        priority: "Penting",
                        note: "huffttt",
                    },
                    {
                        task: "nongkrong",
                        priority: "Tidak penting",
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

            sort:function(s) {
                if(s === this.currentSort) {
                this.currentSortDir = this.currentSortDir==='asc'?'desc':'asc';
                }
                this.currentSort = s;
            }
        },
    };
</script>