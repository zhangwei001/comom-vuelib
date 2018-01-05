<template>
    <div id="hot-preview">
        <HotTable :root="root" ref="hot" :settings="hotSettings"></HotTable>

        <button @click="getData">获取数据</button>
    </div>
</template>

<script>
    import HotTable from 'vue-handsontable-official';
    import Vue from 'vue';
    import tableColumns from '../datas/tableColumns1.json';
    import tableData from '../datas/tableData1.json';

    export default {
        data: function() {
            return {
                root: 'test-hot',
                hotSettings: {
                    data: [['sample', 'data','aaa']],
                    colHeaders: [],
                    columns:[] //列的name配置
                    ,rowHeaders:true
                }
            };
        },
        components: {
            HotTable
        },
        created(){
            var t = this;
            console.log(tableColumns);

            //设置表头
            tableColumns.forEach(function (item) {
                t.hotSettings.colHeaders.push(item.title);
                t.hotSettings.columns.push({data:item.name});
            });

            //设置数据
            var aData = [];
            tableData.maps.forEach(function (item) {
                aData.push(item);
            });
            t.hotSettings.data = aData;

        }
        ,methods:{
            getData(){
                console.log(this.$refs.hot.getData());
            }
        }
    }
</script>

<style lang="scss">
    #test-hot {
        //width: 600px;
        height: 400px;
        overflow: hidden;
    }
</style>