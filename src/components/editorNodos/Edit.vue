<template>
    <div class="container-drawflow-canvan">
        <aside class="sidebar-drawflow">
            <h3 class="sub-title"> Elige y arrastra un nodo:</h3>
            <div class="sub-title-line"></div>
            
            <div class="list-nodes-container">
                <div class="list-nodes" v-for="node in nodesList" :key="node" :dataNode="node.item" draggable="true" @dragstart="drag($event)">
                    <div class="list-nodes-title" v-if="node.name==='number'">Basics items: </div>
                    <div class="list-nodes-title" v-if="node.name==='add'">Basics Operations: </div>
                    <div class="list-nodes-title" v-if="node.name==='If-else'">Control Structures: </div>
                    <div class="node-style" :style="`background-color: ${node.color}`"> {{node.name}} </div>
                </div>
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
    import SubtractNode from './nodesList/SubtractNode.vue'
    import MultiplyNode from './nodesList/MultiplyNode.vue'
    import DivideNode from './nodesList/DivideNode.vue'
    import IfElseNode from './nodesList/IfElseNode.vue'
    import ForNode from './nodesList/ForNode.vue'


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
    var dataNumNode = { "number": 0};//
    var dataAddNode = { "added": 0 };//
    var dataSubtractNode = { "subtracted": 0 };//
    var dataMultiplyNode = { "multiplied": '' };//
    var dataDivideNode = { "divided": '' };//
    function addNewNode(name, pos_x, pos_y){
        pos_x = pos_x * (editor.value.precanvas.clientWidth / (editor.value.precanvas.clientWidth * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().x * (editor.value.precanvas.clienteWidth / (editor.value.precanvas.clienteWidth * editor.value.zoom)))
        pos_y = pos_y * (editor.value.precanvas.clientHeight / (editor.value.precanvas.clientHeight * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().y * (editor.value.precanvas.clienteHeight / (editor.value.precanvas.clienteHeight * editor.value.zoom)))
        const nodeSelected = nodesList.find(element => element.item == name);
        if(name === "NumNode")
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, dataNumNode, name, 'vue');
        else if(name === "AddNode")
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, dataAddNode, name, 'vue');
        else if(name === "SubtractNode")
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, dataSubtractNode, name, 'vue');
        else if(name === "MultiplyNode")
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, dataMultiplyNode, name, 'vue');
        else if(name === "DivideNode")
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, dataDivideNode, name, 'vue');
        else
            editor.value.addNode(name, nodeSelected.inputs, nodeSelected.outputs, pos_x, pos_y, name, {}, name, 'vue');
        dataNumNode = { "number": 0};//
        dataAddNode = { "added": 0 };//
        dataSubtractNode = { "subtracted": 0 };//
        dataMultiplyNode = { "multiplied": '' };//
        dataDivideNode = { "divided": '' };//
        console.log("itemselected: ",nodeSelected)
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
        editor.value.registerNode("SubtractNode", SubtractNode, {}, {});
        editor.value.registerNode("MultiplyNode", MultiplyNode, {}, {});
        editor.value.registerNode("DivideNode", DivideNode, {}, {});
        editor.value.registerNode("AssignNode", AssignNode, {}, {});
        editor.value.registerNode("IfElseNode", IfElseNode, {}, {});
        editor.value.registerNode("ForNode", ForNode, {}, {});

        //cuando se selecciona un nodo creado
        editor.value.on('nodeSelected', function(id) {
        console.log("Node Selected " + id);
        let e = editor.value.getNodeFromId(id);
        console.log("this is it!: ",e)
        })

        //suma, resta, multiplicacion, division --> de nodos
        let valueAddNode1 = 0
        let valueAddNode2 = 0
        let valueSubNode1 = 0
        let valueSubNode2 = 0
        let valueMulNode1 = 0
        let valueMulNode2 = 0
        let valueDivNode1 = 0
        let valueDivNode2 = 0
        editor.value.on('connectionCreated', function(data_inner) {
            console.log("output_id: ",data_inner.output_id)
            console.log("input_id: ",data_inner.input_id)
            let node = editor.value.getNodeFromId(data_inner.output_id)
            let nodeInput = editor.value.getNodeFromId(data_inner.input_id)
            if (nodeInput.name === "AddNode"){
                if (valueAddNode1 === 0) {
                    if (typeof node.data.number !== 'undefined')
                        valueAddNode1 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueAddNode1 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueAddNode1 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueAddNode1 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueAddNode1 = node.data.divided
                }else{
                    if (typeof node.data.number !== 'undefined')
                        valueAddNode2 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueAddNode2 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueAddNode2 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueAddNode2 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueAddNode2 = node.data.divided
                }
                let result = parseInt(valueAddNode1) + parseInt(valueAddNode2)
                editor.value.updateNodeDataFromId(data_inner.input_id, {"added": result })
                result = 0
                if (valueAddNode2 != 0){
                    valueAddNode1 = valueAddNode2 = 0
                }
            }else if (nodeInput.name === "SubtractNode"){
                if (valueSubNode1 === 0) {
                    if (typeof node.data.number !== 'undefined')
                        valueSubNode1 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueSubNode1 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueSubNode1 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueSubNode1 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueSubNode1 = node.data.divided
                }else{
                    if (typeof node.data.number !== 'undefined')
                        valueSubNode2 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueSubNode2 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueSubNode2 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueSubNode2 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueSubNode2 = node.data.divided
                }
                let result = parseInt(valueSubNode1) - parseInt(valueSubNode2)
                editor.value.updateNodeDataFromId(data_inner.input_id, {"subtracted": result })
                result = 0
                if (valueSubNode2 != 0){
                    valueSubNode1 = valueSubNode2 = 0
                }
            }else if (nodeInput.name === "MultiplyNode"){
                if (valueMulNode1 === 0) {
                    if (typeof node.data.number !== 'undefined')
                        valueMulNode1 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueMulNode1 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueMulNode1 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueMulNode1 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueMulNode1 = node.data.divided
                }else{
                    if (typeof node.data.number !== 'undefined')
                        valueMulNode2 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueMulNode2 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueMulNode2 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueMulNode2 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueMulNode2 = node.data.divided
                }
                if (valueMulNode2 != 0){
                    let result = parseInt(valueMulNode1) * parseInt(valueMulNode2)
                    editor.value.updateNodeDataFromId(data_inner.input_id, {"multiplied": result })
                    result = 0
                    valueMulNode1 = valueMulNode2 = 0
                }
            }else if (nodeInput.name === "DivideNode"){
                if (valueDivNode1 === 0) {
                    if (typeof node.data.number !== 'undefined')
                        valueDivNode1 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueDivNode1 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueDivNode1 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueDivNode1 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueDivNode1 = node.data.divided
                }else{
                    if (typeof node.data.number !== 'undefined')
                        valueDivNode2 = node.data.number
                    else if (typeof node.data.added !== 'undefined')
                        valueDivNode2 = node.data.added
                    else if (typeof node.data.subtracted !== 'undefined')
                        valueDivNode2 = node.data.subtracted
                    else if (typeof node.data.multiplied !== 'undefined')
                        valueDivNode2 = node.data.multiplied
                    else if (typeof node.data.divided !== 'undefined')
                        valueDivNode2 = node.data.divided
                }
                if (valueDivNode2 != 0){
                    let result = parseInt(valueDivNode1) / parseInt(valueDivNode2)
                    editor.value.updateNodeDataFromId(data_inner.input_id, {"divided": result })
                    result = 0
                    valueDivNode1 = valueDivNode2 = 0
                }
            }
        })

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
            name: 'assign',
            color: 'rgb(201, 241, 252)',
            item: 'AssignNode',
            inputs: 1,
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
            name: 'subtract',
            color: 'rgb(216, 240, 152)',
            item: 'SubtractNode',
            inputs: 2,
            outputs: 1
        },
        {
            name: 'multiply',
            color: 'rgb(152, 175, 240)',
            item: 'MultiplyNode',
            inputs: 2,
            outputs: 1
        },
        {
            name: 'divide',
            color: 'rgb(240, 192, 152)',
            item: 'DivideNode',
            inputs: 2,
            outputs: 1
        },
        {
            name: 'If-else',
            color: 'rgb(240, 152, 152)',
            item: 'IfElseNode',
            inputs: 2,
            outputs: 2
        },
        {
            name: 'For',
            color: 'rgb(152, 168, 240)',
            item: 'ForNode',
            inputs: 2,
            outputs: 2
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

    .sub-title-line {
        border-bottom: 2px solid gray;
    }

    .list-nodes-container {
        height: 40rem;
        overflow-y: auto;
    }

    .list-nodes-container::-webkit-scrollbar {
        -webkit-appearance: none!important;
    }

    .list-nodes-container::-webkit-scrollbar:vertical {
        width: 10px!important;
    }

    .list-nodes-container::-webkit-scrollbar-thumb {
        background-color: #797979!important;
        border-radius: 30px!important;
        border: 2px solid #f1f2f3!important;
    }

    .list-nodes-container::-webkit-scrollbar-track {
        border-radius: 30px!important;
    }

    .list-nodes-title {
        font-family: Montserrat;
        font-size: 14px;
        font-weight: 600;
        margin-top: 25px;
        margin-bottom: 8px;
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