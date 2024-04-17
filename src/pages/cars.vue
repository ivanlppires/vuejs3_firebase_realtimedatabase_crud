<template>
    <v-data-table :headers="headers" :items="cars">
        <template v-slot:top>
            <v-toolbar flat color="primary">
                <v-toolbar-title>Cars Management</v-toolbar-title>
                <v-divider class="mx-4" inset vertical></v-divider>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ props }">
                        <v-btn variant="outlined" append-icon="mdi-plus" v-bind="props">
                            Add
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="text-h5">{{ formTitle }}</span>
                        </v-card-title>

                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12">
                                        <v-text-field v-model="editedItem.maker" label="Fabricante"></v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field v-model="editedItem.model" label="Modelo"></v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field v-model="editedItem.year" label="Ano"></v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue-darken-1" variant="text" @click="close">
                                Cancelar
                            </v-btn>
                            <v-btn color="blue-darken-1" variant="text" @click="save">
                                Salvar
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="500px">
                    <v-card>
                        <v-card-title class="text-h5">Tem certeza que deseja remover?</v-card-title>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancelar</v-btn>
                            <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm">OK</v-btn>
                            <v-spacer></v-spacer>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
            <v-icon class="me-2" size="small" @click="editItem(item)">
                mdi-pencil
            </v-icon>
            <v-icon size="small" @click="deleteItem(item)">
                mdi-delete
            </v-icon>
        </template>
        <template v-slot:no-data>
            Sem dados disponiveis
        </template>
    </v-data-table>
</template>

<script setup>
import { computed, nextTick, ref, watch } from 'vue'

const dialog = ref(false)
const dialogDelete = ref(false)
const headers = ref([
    { title: 'Fabricante', key: 'maker' },
    { title: 'Modelo', key: 'model' },
    { title: 'Ano', key: 'year' },
    { title: 'Ações', key: 'actions', sortable: false },
])
const cars = ref([])
const editedIndex = ref(-1)
const editedItem = ref({
    id: '',
    maker: '',
    model: '',
    year: '',
})
const defaultItem = ref({
    id: '',
    maker: '',
    model: '',
    year: '',
})
const formTitle = computed(() => {
    return editedIndex.value === -1 ? 'Novo carro' : 'Editar item'
})
function editItem(item) {
    editedItem.value = Object.assign({}, item)
    dialog.value = true
}
function deleteItem(item) {
    editedIndex.value = item.id
    editedItem.value = Object.assign({}, item)
    dialogDelete.value = true
}
function deleteItemConfirm() {
    fetch('https://fmds-crud-default-rtdb.firebaseio.com/cars/' + editedIndex.value + '.json',
        {
            method: 'delete'
        })
        .then()
        .then(() => load())
        .catch(err => console.log(err))
    closeDelete()
}
function close() {
    dialog.value = false
    nextTick(() => {
        editedItem.value = Object.assign({}, defaultItem.value)
        editedIndex.value = -1
    })
}
function closeDelete() {
    dialogDelete.value = false
    nextTick(() => {
        editedItem.value = Object.assign({}, defaultItem.value)
        editedIndex.value = -1
    })
}
function save() {
    if (editedItem.value.id) {
        console.log('entrou')
        fetch('https://fmds-crud-default-rtdb.firebaseio.com/cars/' + editedItem.value.id + '.json',
        {
            method: 'PUT',
            body: JSON.stringify(editedItem.value)
        })
        .then().then(
            () => load()
        ).catch(err => console.log(err))
    } else {

        fetch('https://fmds-crud-default-rtdb.firebaseio.com/cars.json',
            {
                method: 'POST',
                body: JSON.stringify(editedItem.value)
            })
            .then()
            .then(
                () => load()
            )
            .catch(
                () => {
                    console.log('erro ao cadastrar');
                }
            )
    }
    close()
}
const load = () => {
    cars.value = []
    fetch('https://fmds-crud-default-rtdb.firebaseio.com/cars.json')
        .then(data => data.json())
        .then(
            data => {
                for (const key in data) {
                    console.log(key)
                    const car = {
                        id: key,
                        maker: data[key].maker,
                        model: data[key].model,
                        year: data[key].year
                    }
                    cars.value.push(car)
                }
            }
        )
        .catch(err => console.log(err))
}
onBeforeMount(() => {
    load();
}),

    watch(dialog, val => {
        val || close()
    })
watch(dialogDelete, val => {
    val || closeDelete()
})
</script>