<template>

    <div class="ppcNode" :class="[{focused},node.type]"
         @click.stop="focus">
         <!--@click.stop="select({i:index, selected: node})">-->
        <div class="node-container" v-if="node.type==='randList' || node.type==='loopList'">
            <label class="node-meta">
                <div class="key">Тип:</div>
                <div class="value">
                    <!--{{types.obj[node.type].title}}-->
                    <select class="head-select"
                            name="type"
                            id="ntype"
                            v-model="node.type"
                    >
                            <!--@change="changeNodeType"-->
                        <option :value="item.value"
                                v-for="item in types.arr"
                                :disabled="owner.type==='process' && item.value==='quest'"
                        >{{item.title}}
                        </option>
                    </select>
                </div>
                <div class="key">Название:</div>
                <div class="value">{{node.attrs.nodeName.value}}</div>
            </label>

            <div class="node-list" :class="node.type">

                <div class="node-item" v-for="(child, idx) in node.list" :key="child.forKey">
                    <div class="insert-bar">
                        <div class="insert-button">
                            <i class="ico ico-plus"></i>
                        </div>
                        <div class="add-list">
                            <div class="add-item" v-for="mi in types.arr"
                                 @click="addNode(mi.value, idx)">
                                {{mi.title}}
                            </div>
                        </div>
                    </div>
                    <div class="pay-field" v-if="node.type==='randList'"
                         @click.stop="$children[idx].focus">
                        <input class="pay-input"
                               type="text"
                               v-model="child.attrs.pay.value"
                               v-number-only
                        />
                        <div class="pay-percent">
                            {{(child.attrs.pay.value/paySum*100).toFixed(2)}}%
                        </div>
                    </div>

                    <div class="delete-button" @click="removeNode(idx)">
                        <i class="ico ico-minus"></i>
                    </div>

                    <ppcNode
                        :class="{selected: selectedChild===idx}"
                        :node="child"
                        :owner="node"
                        :createNodeFunc="createNodeFunc"
                        @select="select"
                        @changed="processChanged"
                        :index="idx"
                    />

                </div>

                <div class="node-item pos-rel">
                    <div class="add-button">
                        <i class="ico ico-plus"></i>
                    </div>
                    <div class="add-list">
                        <div class="add-item" v-for="mi in types.arr"
                             @click="addNode(mi.value)">
                            {{mi.title}}
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="quest-container" v-else-if="node.type==='quest'">
            <!--<input class="pay-input" v-if="owner.type==='randList'"-->
                   <!--type="text"-->
                   <!--v-model="node.attrs.pay"/>-->
            <textarea class="quest-input" type="text"
                  v-model="node.attrs.quest.value"
                  @input="processChanged"
            />
        </div>
        <div class="node-container" v-else>
            <div class="type-error-node">
                Неизвестный тип узла
            </div>
        </div>
    </div>
</template>

<script>
    import {nodeTypes} from "@/assets/js/const.js"

    export default {
        name: "ppcNode",
        components: {},
        props: ['node', 'owner', 'createNodeFunc', 'index'],
        data() {
            return {
                types: nodeTypes,
                focused: false,
                selectedChild: -1,
            }
        },
        computed: {
            paySum(){
                return this.node.list.reduce((s,v)=>{
                    return s + (v.attrs.pay.value * 1);
                },0);
            }
        },
        methods: {
            processChanged(){
                this.$emit('changed');
            },
            addNode(type, idx){
                if(idx===undefined) {
                    this.node.list.push(this.createNodeFunc(type));
                } else {
                    this.node.list.splice(idx, 0, this.createNodeFunc(type));
                }
                this.$emit('changed');
            },
            removeNode(idx){
                if(this.node.list[idx].type==='quest') {
                    this.node.list.splice(idx, 1);
                } else {
                    this.node.list.splice(idx, 1, ...this.node.list[idx].list);
                }
                this.$emit('changed');
            },
//            select({i, selected}){
//                console.log(arguments);
//                console.log(selected);
//                this.$emit('select', {i:this.index, selected});
//                if (selected !== this.node) {
//                    this.selectedChild = i;
//                    this.focused = false;
//                } else {
//                    this.selectedChild = -1;
//                    this.focused = true;
//                }
//            },
            select({i, selected}){
                this.$emit('select', {i:this.index, selected});
                    this.selectedChild = i;
                    this.focused = false;
            },
            focus(){
                this.unselectAllNodes();
                this.$emit('select', {i:this.index, selected:this.node});
                this.selectedChild = -1;
                this.focused = true;
            },
            unselect(){
                console.log('unselect !', this.index);
                this.focused = false;
                this.selectedChild = -1;
                this.$children.forEach(v => v.unselect());
            },
            unselectAllNodes(){
                console.log('!! unselectAllNodes  fired !!');
                let e = new Event('unselectAllNodes', {
                    bubbles: true,
                });
                this.$el.dispatchEvent(e);
            },
        },
        watch: {
            "owner.selectedChild"(v){
                if (v !== this.index) {
                    this.selectedChild = -1;
                    this.focused = false;
                }
            },
            "node.type"(v, old) {
                console.log(`Node Type Changed !! (${old} ==> ${v})`);
                if(old==='quest') {
                    this.node.list.push(this.createNodeFunc('quest'));
                    this.node.list[0].attrs.quest.value = this.node.attrs.quest.value;
                    this.node.attrs.quest.value = '';
                } else if(v==='quest') {
//                    this.node.type = old;
//                    this.owner.removeNode(this.index);
                    this.owner.list.splice(this.index+1, 0, ...this.node.list);
                    this.node.list.length = 0;
                }
                this.processChanged();
            }
        },
        mounted(){

        },
    }
</script>

<style lang="scss">
    .ppcNode {
        position: relative;
        width: 100%;
        height: auto;
        /*padding: 2px 2px 2px 2px;*/
        margin: 4px 4px;
        background-color: hsl(50, 30%, 90%);
        border: 1px solid hsl(50, 20%, 70%);
        border-radius: 4px;
        box-shadow: 0 0 5px 0 hsla(50, 20%, 20%, 0.1);
        color: hsl(50, 20%, 20%);
        &.quest {
             /*padding: 2px 2px 2px 2px;*/
            padding: 0;
            /*background-color:hsl(50, 20%, 50%);*/
        }
        .node-container {
            width: 100%;
            display: flex;
            flex-flow: column nowrap;
            /*padding-right: 5px;*/
        }
        .node-meta {
            display: flex;
            align-items: center;
            background-color: hsl(50, 20%, 95%);
            border-bottom: 1px solid hsl(50, 20%, 80%);
            padding: 0px 3px 0px 8px;
            border-radius: 5px 5px 0 0;
            margin: 0;
            .head-select {
                flex: 1 1 auto;
                height: 28px;
                margin-left: 1px;
                padding-left: 1px;
                background-color: transparent;
                border: none;

            &:focus {
                 background-color: white;
                 border-color: hsl(50, 30%, 75%);
             }
            }
        }

        .node-list {
            /*width: 100%;*/
            margin: 0 0 0 5px;
            padding: 0 2px 2px 0;
            .node-item {
                position: relative;
                display: flex;
                .ppcNode {
                    flex: 1 1 auto;
                }
                .pay-field {
                    flex: 0 0 auto;
                    width: 70px;
                    height: 30px;
                    border-radius: 4px;
                    background-color: hsl(50, 20%, 95%);
                    border: 1px solid hsl(50, 20%, 70%);
                    /*border: 1px solid hsl(0, 0%, 90%);*/
                    display: flex;
                    align-items: center;
                    margin: 4px 0px 4px 4px;
                    padding: 0px;
                    /*bc*/
                    input.pay-input {
                        flex: 0 0 auto;
                        height: 28px;
                        width: 22px;
                        min-width: 22px;
                        line-height: 28px;
                        text-align: center;
                        font-size: 15px;
                        /*font-weight: bold;*/
                        background-color: transparent;
                        border-radius: 5px;
                        border: none;
                        &:focus {
                             width: 26px;
                             background-color: white;
                             border-color: hsl(50, 30%, 75%);
                         }
                    }
                    .pay-percent {
                        flex: 1 1 auto;
                        font-size: 12px;
                        /*font-weight: bold;*/
                        color: hsl(50, 30%, 55%);
                        text-align: right;
                        padding-right: 2px;
                    }
                }
            }
        }

        .insert-bar {
            height: 10px;
            width: calc(100% - 22px);
            position: absolute;
            left: 0;
            top: -5px;
            &:hover {
                background-color: hsla(120, 10%, 70%, 0.3);
                .insert-button {
                    display: flex;
                }
            }
            .insert-button {
                display: none;
                position: absolute;
                height: 20px;
                width: 20px;
                left: -4px;
                top: -7px;
                margin: 2px auto 0 0;
                background-color: hsl(120, 10%, 70%);
                align-items: center;
                justify-content: center;
                font-size: 11px;
                /*line-height: 5px;*/
                color: white;
                border: 1px solid currentColor;
                border-radius: 50%;
                cursor: pointer;
                /*transform: scale3d(0.9, 0.9, .1);*/
                transition: all ease 0.8s;

                &:hover {
                     background-color: hsl(120, 80%, 50%);
                     /*transform: scale3d(1, 1, .1);*/
                     transition: all ease 0.2s;
                & + .add-list {
                        display: flex;
                    }
                }

            }
            .add-list {
                top: -5px;
                left: -3px;
            }
        }

        .key {
            font-size: 11px;
            font-weight: bold;
            margin-right: 6px;
        }

        .value {
            font-size: 13px;
            margin-right: 12px;
        }

        .quest-container {
            display: flex;
            height: auto;
            align-items: flex-start;
            padding: 2px;
            input, textarea {
                padding: 0px 5px 0px 5px;
                line-height: 20px;
            }
            .pay-input {
                display: block;
                flex: 0 0 auto;
                width: 35px;
                height: 28px;
                font-size: 14px;
                line-height: 28px;
                text-align: right;
                margin-right: 3px;
                border-radius: 4px;
                border: 1px solid hsl(0, 0%, 90%);
            }
            .quest-input {
                display: block;
                flex: 1 1 auto;
                height: 28px;
                min-height: 28px;
                font-size: 16px;
                line-height: 24px;
                /*border-radius: 0 4px 4px 0;*/
                border-radius: 4px;
                border: 1px solid hsl(0, 0%, 90%);
            }
        }
        .add-list {
            display: none;
            position: absolute;
            z-index: 5;
            top: 0;
            left: 0;
            width: auto;
            height: auto;
            padding: 5px 0;
            flex-flow: column nowrap;
            background-color: white;
            border: 1px solid hsl(50, 20%, 80%);
            .add-item {
                padding: 2px 10px;
                background-color: white;
                cursor: pointer;
                &:hover {
                    background-color: hsl(150, 50%, 80%);
                    color: black;

                 }
            }
            &:hover {
                display: flex;
             }
        }
        &.selected {
             /*z-index: 5;*/
             box-shadow: 0 0 0 1px hsl(50, 30%, 80%), 0 0 0 2px hsl(50, 20%, 50%);
            &.focused {
                 background-color: hsl(150, 30%, 90%);
                 border: 1px solid hsl(150, 20%, 50%);
                 box-shadow: 0 0 0 1px hsl(150, 30%, 80%), 0 0 0 3px hsl(150, 20%, 50%);
             }
        }
    }

</style>
