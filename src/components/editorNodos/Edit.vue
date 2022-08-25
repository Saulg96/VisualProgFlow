<template>
    <div class="container-drawflow-canvan">
        <aside class="sidebar-drawflow">
            <h3 class="sub-title"> Elige y arrastra un nodo:</h3>
            
            <div class="list-nodes" v-for="node in nodesList" :key="node" :dataNode="node.item" draggable="true" @dragstart="drag($event)">
                <div class="node-style" :style="`background-color: ${node.color}`"> {{node.name}} </div>
            </div>
            
        </aside>
        <main class="main-drawflow">
            <h3 class="title">Aquí puedes arrastrar y soltar los nodos para empezar a crear tu nuevo programa.</h3>
            <div id="drawflow" @drop="drop($event)" @dragover="$event.preventDefault()"></div>
        </main>
    </div>
</template>
<script setup>
    import Drawflow from 'drawflow'
    //import styleDrawflow from 'drawflow/dist/drawflow.min.css'
    import styleDrawflow from '../../assets/drawflow.min.css'
    import {onMounted, shallowRef, h, getCurrentInstance, render, readonly, ref} from 'vue'
    import AddNode from './nodesList/AddNode.vue'
    import AssignNode from './nodesList/AssignNode.vue'
    import NumNode from './nodesList/NumNode.vue'


    const editor = shallowRef({})
    const Vue = {version: 3, h, render};
    const internalInstance = getCurrentInstance()
    internalInstance.appContext.app._context.config.globalProperties.$df = editor;

    let itemSelect = "";
    let lastMove = null;

    function positionMobile(ev){
        lastMove = ev;
    }

    const drag = (ev) => {
        if (ev.type === "touchStart") {
            itemSelect = ev.target.closest("list-nodes").getAttribute("dataNode");
        } else {
            ev.dataTransfer.setData("node-style", ev.target.getAttribute("dataNode"))
        }
    }

    const drop = (ev) => {
        if (ev.type === "touchEnd") {
            var parentDrawFlow = document.elementFromPoint(lastMove.touches[0].cientX, lastMove.touches[0].cientY).closest("#drawflow");
            if (parentDrawFlow != null) {
                addNewNode(itemSelect, lastMove.touches[0].cientX, lastMove.touches[0].cientY)
            }
            itemSelect = "";
        } else {
            ev.preventDefault();
            var data = ev.dataTransfer.getData("node-style");
            addNewNode(data, ev.clientX, ev.ClientY)
        }
    }

    function addNewNode(name, pos_x, pos_y){
        pos_x = pos_x * (editor.value.precanvas.clientWidth / (editor.value.precanvas.clientWidth * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().x * (editor.value.precanvas.clienteWidth / (editor.value.precanvas.clienteWidth * editor.value.zoom)))
        pos_y = pos_y * (editor.value.precanvas.clientHeight / (editor.value.precanvas.clientHeight * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().y * (editor.value.precanvas.clienteHeight / (editor.value.precanvas.clienteHeight * editor.value.zoom)))
        const nodeSelected = nodesList.find(element => element.item == name);
        editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, {}, name, 'vue');
    }

    onMounted(() => {
        var listNodes = document.getElementsByClassName("list-nodes");
        for (let i = 0; i < listNodes.length; i++){
            listNodes[i].addEventListener("touchStart", drag, false);
            listNodes[i].addEventListener("touchMove", positionMobile, false);
            listNodes[i].addEventListener("touchEnd", drop, false);
        }
        const id = document.getElementById("drawflow");
        console.log(id)
        editor.value = new Drawflow(id, Vue, internalInstance.appContext.app._context);
        editor.value.start();

        editor.value.registerNode("NumNode", NumNode, {}, {});
        editor.value.registerNode("AddNode", AddNode, {}, {});
        editor.value.registerNode("AssignNode", AssignNode, {}, {});
    })

    const nodesList = readonly([
        {
            name: 'number',
            color: 'rgb(232, 233, 256)',
            item: 'NumNode',
            inputs: 0,
            outputs: 1
        },
        {
            name: 'add',
            color: 'rgb(152, 240, 171)',
            item: 'AddNode',
            inputs: 2,
            outputs: 1
        },
        {
            name: 'assign',
            color: 'rgb(201, 241, 252)',
            item: 'AssignNode',
            inputs: 1,
            outputs: 1
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
        cursor: move;
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