<script setup>
import { api } from '@/modules/api';
import { computed, onBeforeMount, ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const files = ref([])

const filesDisplay = computed(()=> {
    const filesMap =  []
    for(const file of files.value){
        if(showHidden.value) {
            filesMap.push(file)
        } else {
            // not show hidden need filter
            if(!file.name.startsWith(".")) {
                filesMap.push(file)
            }
        }    
    }
    
    console.log("files Map", filesMap)
    return filesMap
})

const route = useRoute()

const path = computed(()=>{
    let p = route.query.path
    if(!p) {
        // default path is /
        return "/"
    }
    return p
})

watch(()=> path.value, async ()=>{
    await getFiles()
})

onBeforeMount(async ()=> {
    await getFiles()
})

const getFiles = async () => {
    try {
        await api("GET", `/files?path=${path.value}`).then(res => {
            if(!res["data"]) {
                res["data"] = []
            }
            files.value = res["data"]
        })
    } catch (error) {
        console.log(error)
    }
}

const deleteFile = async (pathDelete, nameDelete) => {
    const c = confirm("Want delete " + nameDelete)
    if(!c){
        return;
    }
    try {
        await api("POST", `/files/delete${pathDelete}`).then(res => {
            const idx = files.value.findIndex(item => item.name == nameDelete)
            if(idx >= 0) {
                files.value.splice(idx, 1)
            }
        })
    } catch (error) {
        console.log(error)
    }
}

const createDir = async () => {
    const nameCreate = prompt("Enter Dir name")
    if(!nameCreate){
        return;
    }

    const pathCreate = path.value + "/" + nameCreate

    try {
        await api("POST", `/files/create?path=${pathCreate}`).then(res => {
            files.value.unshift({
                name: nameCreate,
                is_dir: true
            })
        })
    } catch (error) {
        console.log(error)
    }
}


const pathTo = (name) => {
    const basePath = "?path="

    if(name == "/") {
        // to root
        return basePath + "/"
    }

    if(path.value == "/"){
        return basePath + path.value + name
    } else {
        return basePath + path.value + "/" + name
    }
}

const pathParts = computed(()=> {
    return path.value.split("/").slice(1); 
})

const pathPartsTo = (index) => {
    const basePath = "?path="
    let partTo = ""
    for(let i = 0 ; i <= index; i++){
        partTo = partTo + "/" + pathParts.value[i]
    }

    return basePath + partTo
}

const showHidden = ref(false)

</script>

<template>
    <div>
        <h1>Godriver</h1>
        <hr>
        <div>
            <span v-for="(part, index) of pathParts" :key="index">
                / <RouterLink :to="pathPartsTo(index)">{{ part }}</RouterLink>
            </span>
        </div>
        <hr>
        <div>
            <span v-if="path != '/'">
                <RouterLink :to="pathTo('/')">Root</RouterLink> -
                <RouterLink :to="pathPartsTo(pathParts.length - 2)">Back</RouterLink>
            </span> -
            <span>
                <input type="checkbox" v-model="showHidden">
                <span>Show hidden</span>
            </span> - 
            <button @click="createDir">Create Dir</button>
        </div>
        <hr>
        <table>
            <tr>
                <th>Name</th>
                <th>Action</th>
            </tr>
            <tr v-for="file of filesDisplay" :key="file.name">
                <td>
                    <div v-if="!file.is_dir">{{ file.name }}</div>
                    <div v-else>
                        <b>
                            <RouterLink :to="pathTo(file.name)">{{ file.name }}</RouterLink>
                        </b>
                    </div>
                </td>
                <td>
                    <button @click="deleteFile(pathTo(file.name), file.name)">Delete</button>
                </td>
            </tr>
        </table>
    </div>
</template>

<style scoped>
table {
    width: 100%;
}

td, th {
  border: 1px solid #ddd;
  padding: 8px;
}


tr:hover {background-color: #ddd;}

th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #04AA6D;
  color: white;
}
</style>