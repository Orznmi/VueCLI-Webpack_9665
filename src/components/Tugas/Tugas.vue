<template>
<v-main class="list">
    <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

    <v-card>
        <v-card-title>
            <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details>
            </v-text-field>
            <v-spacer></v-spacer>
            <v-col cols="12" sm="5">
                <v-select v-model="sort" :items="['Penting', 'Tidak penting']" label="Urutkan" @change="sorting(sort)" outlined hide-details dense></v-select>
            </v-col>
            <v-spacer></v-spacer>

            <v-btn color="success" dark @click="dialog =true"> Tambah </v-btn>
        </v-card-title>

        <v-data-table :headers="headers" :items="todos" :search="search" v-model="selected" :single-expand="singleExpand" item-key="task" :expanded.sync="expanded" show-expand class="elevation-1">
            <template v-slot:[`item.priority`]="{ item }">
                <td>
                    <v-chip small label outlined :color="color(item.priority)">{{ item.priority }}</v-chip>
                </td>
            </template>

            <template v-slot:expanded-item="{ headers, item }">
                <td :colspan="headers.length">
                    {{ item.note }}
                </td>
            </template>

            <template v-slot:[`item.actions`]="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)" icon color="blue">mdi-pencil</v-icon>
                <v-icon small class="mr-2" @click="deleteItem(item)" icon color="red" slot="activator">mdi-delete</v-icon>
            </template>

            <template v-slot:[`item.select`]="{ item }">
                <v-checkbox multiple :key="item" @click.capture.stop="checked(item)">
                </v-checkbox>
            </template>
        </v-data-table>

    </v-card>

    <v-spacer></v-spacer>
    <v-spacer></v-spacer>
    <br>

    <v-card v-if="check.length>0">
        <v-card-title>
            <span>Delete Multiple</span>
        </v-card-title>
        <v-spacer></v-spacer>
        <v-list-item v-for="(item,i) in check" :key="i">
            <v-list-item-content>
                <p> {{item.task}}</p>
            </v-list-item-content>
        </v-list-item>

        <v-card-actions>
            <v-btn small @click="deleteAll" color="red">Hapus Semua</v-btn>
        </v-card-actions>

    </v-card>

    <v-dialog v-model=" dialog" persistent max-width="600px">
        <v-card>
            <v-card-title>
                <span class="headline">Form Todo</span>
            </v-card-title>
            <v-card-text>
                <v-container>
                    <v-text-field v-model="formTodo.task" label="Taks" required>
                    </v-text-field>

                    <v-select v-model="formTodo.priority" :items="['Penting', 'Biasa', 'Tidak Penting']" label="Priority" required>
                    </v-select>

                    <v-textarea v-model="formTodo.note" label="Note" required>
                    </v-textarea>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="cancel"> Cancel </v-btn>
                <v-btn v-if="cek == true" color="blue darken-1" text @click="save">Save</v-btn>
                <v-btn v-else color="blue darken-1" text @click="update(formTodo)">Update</v-btn>

            </v-card-actions>
        </v-card>
    </v-dialog>

    <template>
        <v-row justify="center">
            <v-dialog v-model="dialogHapus" persistent max-width="290">
                <v-card>
                    <v-card-title class="heading 3">
                        Yakin mau menghapus?
                    </v-card-title>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="green darken-1" text @click="dialogHapus = false">
                            Tidak
                        </v-btn>
                        <v-btn color="red darken-1" text @click="hapus">
                            Ya
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-row>
    </template>

    <v-dialog v-model="dialogView" width="600px">
        <v-card>
            <v-card-title>
                <span class="headline font-weight-medium">{{ detail.task }}</span>
            </v-card-title>
            <v-card-text>
                <v-chip small label outlined :color="color(detail.priority)">{{ detail.priority }}</v-chip>
            </v-card-text>
            <v-card-text> {{detail.note}} </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="dialogView = false">
                    close
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
            sort: "",
            search: null,
            cek: true,
            itemTemp: null,
            dialogEdit: false,
            dialogHapus: false,
            dialogView: false,
            dialog: false,
            expanded: [],
            check: [],
            updateSubmit: false,
            headers: [{
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                    select: "task"

                },
                {
                    text: "Priority",
                    value: "priority"
                },
                {
                    text: "Actions",
                    value: "actions"
                },
                {
                    text: "",
                    value: "select"
                }

            ],
            todos: [{
                    task: " bernafas ",
                    priority: "Penting",
                    note: " huffttt ",
                },
                {
                    task: " nongkrong ",
                    priority: "Tidak penting",
                    note: " bersama tman2 ",
                },
                {
                    task: " masak ",
                    priority: "Biasa",
                    note: " masak air 500ml ",
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
            this.dialogUpdate = false;
            this.cek = true;
            this.itemTemp = null;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        deleteItem(item) {
            this.dialogHapus = true;
            this.itemTemp = item;
        },
        hapus() {
            var index = this.todos.indexOf(this.itemTemp);
            this.todos.splice(index, 1);

            this.dialogHapus = false;
        },
        editItem(item) {
            this.dialog = true;
            this.cek = false;
            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
            this.itemTemp = item;
        },
        update(formTodo) {
            this.itemTemp.task = formTodo.task;
            this.itemTemp.priority = formTodo.priority;
            this.itemTemp.note = formTodo.note;
            this.dialog = false;
            this.cek = false;
            this.cancel();
        },

        detailItem(item) {
            this.dialogView = true;
            this.detail = {
                task: item.task,
                priority: item.priority,
                note: item.note,

            }
        },

        color(priority) {
            if (priority == "Penting") {
                return 'red';
            } else if (priority == 'Biasa') {
                return 'blue';
            } else {
                return 'green';
            }
        },
        //selected item
        checked(item) {
            if (this.check.includes(item)) {
                this.check.splice(this.check.indexOf(item), 1);
                this.hapus = false;
            } else {
                this.check.push(item);
            }
        },
        //delete all
        deleteAll() {
            var checked = this.check.slice(0);
            for (var i = 0; i < checked.length; i++) {
                // const index = this.todos.indexOf(this.checked[i]);
                // this.todos.splice(index, 1);
                this.todos.splice(this.todos.indexOf(this.check[i]), 1)
            }
            for (var i = 0; i < checked.length; i++) {
                this.check.splice(this.check.indexOf(this.check), 1)
            }

            this.check = [];

        },
        //sorting
        sorting(sort) {
            function compare(a, b) {
                if (sort == "Penting") {
                    if (a.priority == "Penting")
                        return -1;
                    if (a.priority == "Tidak penting")
                        return 1;
                    if (a.priority == "Biasa" && b.priority == "Penting")
                        return 1;
                    if (a.priority == "Biasa" && b.priority == "Tidak penting")
                        return 0;
                } else if (sort == "Tidak penting") {
                    if (a.priority == "Tidak penting")
                        return -1;
                    if (a.priority == "Penting")
                        return 1;
                    if (a.priority == "Biasa" && b.priority == "Tidak Penting")
                        return 1;
                    if (a.priority == "Biasa" && b.priority == "Penting")
                        return 0;
                }
            }
            return this.todos.sort(compare);
        },
    },

};
</script>
