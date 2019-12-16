<template>
    <div class="app">
        <div>
            <input type="text" v-model="newTask" @keyup.enter="addTask">
            <button @click="addTask">Add Task</button>
        </div>
        <div>
            <ul>
                <li v-for="item in items" :key="item.id">

                    <a href="#" v-if="item.done" @click="tombolSelesai(item)">✅</a>
                    <a href="#" v-if="!item.done" @click="tombolSelesai(item)">🔲</a>

                    <input type="text" v-model="editTask" v-if="editId === item.id">
                    <span v-else>{{ item.name }}</span>

                    <span v-if="!editId">
                    <button @click="remove(item)">&times;</button>
                    <button @click="edit(item)">Edit</button>
                    </span>
                    <span v-if="editId === item.id">
                    <button @click="update(item)">Update</button>
                    <button @click="cancel">Cancel</button>
                    </span>
                </li>
                </ul>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
export default {

    data() {
        return {
          newTask: '',

          editTask: '',
          editId: null,

          items: []
        }
    },

    async mounted (){
        this.getList()
        /* const response = await axios.get('http://localhost:4001/tasks/')
        this.tasks = response.data */
    },

    methods: {
        async getList() {
            const backend='http://localhost:4001/tasks/'
            const response = await axios.get(backend)
            this.items = response.data
        },

        async addTask() {
            const backend='http://localhost:4001/tasks/'
            const payload = {
                name: this.newTask
            }

            const response = await axios.post(backend+'addTask', payload)

            this.items.push({
                name: this.newTask
            })

            this.newTask = ''
            this.getList()

        },

        async remove(item) {
            const backend='http://localhost:4001/tasks/'
            const response = await axios.delete(backend+`delTask/${item.id}`)

            const index = this.items.indexOf(item)
           this.items.splice(index, 1)
        },

        edit(item) {
          this.editId = item.id
          this.editTask = item.name
        },

        async update(item) {
            const backend='http://localhost:4001/tasks/'
            const payload = {
                id: item.id,
                done: item.done,
                name: this.editTask
            }

            const response = await axios.put(backend+`setTask/${item.id}`, payload)
            this.getList()
            this.cancel()
        },

        cancel() {
          this.editId = ''
          this.editTask = ''
        },

        async tombolSelesai(item) {
            const backend='http://localhost:4001/tasks/'
            const payload = {
                name: item.name,
            }

            payload.done = item.done ? false : true

            const response = await axios.put(backend+`setTask/${item.id}`, payload)
            this.getList()
        },
  }

}
</script>
