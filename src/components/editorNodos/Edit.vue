<template>
    <div class="container-drawflow-canvan">
        <aside class="sidebar-drawflow">
            <h3 class="sub-title"> Elige y arrastra un nodo:</h3>
            
            <div v-for="node in nodesList" :key="node" draggable="true">
                <div class="node-style" :style="`background-color: ${node.color}`"> {{node.name}} </div>
            </div>
            
        </aside>
        <main class="main-drawflow">
            <h3 class="title">Aquí puedes arrastrar y soltar los nodos para empezar a crear tu nuevo programa.</h3>
            <div id="drawflow" @dragover="$event.preventDefault()"></div>
        </main>
    </div>
</template>
<script setup>
import Drawflow from 'drawflow'
import styleDrawflow from 'drawflow/dist/drawflow.min.css'
import {onMounted, shallowRef, h, getCurrentInstance, render, readonly, ref} from 'vue'


        const editor = shallowRef({})
        const Vue = {version: 3, h, render};
        const internalInstance = getCurrentInstance()
        internalInstance.appContext.app._context.config.globalProperties.$df = editor;

        onMounted(() => {
            const id = document.getElementById("drawflow");
            console.log(id)
            editor.value = new Drawflow(id, Vue, internalInstance.appContext.app._context);
            editor.value.start();
        })

        const nodesList = readonly([
            {
                name: 'number',
                color: 'rgb(232, 233, 256)',
                item: 'sumNode',
                inputs: 1,
                outputs: 1
            },
            {
                name: 'add',
                color: 'rgb(152, 240, 171)',
                item: 'addNode',
                inputs: 2,
                outputs: 1
            },
            {
                name: 'assign',
                color: 'rgb(201, 241, 252)',
                item: 'asignNode',
                inputs: 1,
                outputs: 0
            },
        ])
        
    
</script>
<style scoped>
    .container-drawflow-canvan {
        display: grid;
        grid-template-columns: 20rem auto;
        column-gap: 25px;
        width: 100%;
        margin-top: 2rem;
    }

    .sidebar-drawflow {
        height: 45rem;
        border-radius: 10px;
        border: 1px solid black;
        padding-left: 2rem;
        padding-right: 2rem;
        background-color: rgb(250, 246, 239);
    }

    .sub-title {
        font-family: Montserrat;
        font-size: 1rem;
    }

    .node-style {
        height: 4rem;
        margin-bottom: 0.8rem;
        display: flex;
        align-items: center;
        justify-content: center;
        font-family: Montserrat;
        font-size: 18px;
        font-weight: 600;
        border-radius: 12px;
        border: 1px solid black;
        /*box-shadow: 5px 2px 6px 0px black;*/
    }

    .main-drawflow {
        height: 45rem;
        border-radius: 10px;
        border: 1px solid black;
        padding-left: 1rem;
        padding-right: 1rem;
    }

    .title {
        font-family: Montserrat;
        font-size: 1rem;
    }

    #drawflow {
        width: auto;
        height: 40.5rem;
        background: #e4f3f8;
        background-size: 20px 20px;
        background-image: radial-gradient(#494949 1px, transparent 1px);
        border-radius: 10px;
        border: 1px solid black;
    }
</style>