<template>
    <div style="padding:60px;">
        <el-form label-width="100px">
            <!-- 规格添加 -->
            <el-form-item label="商品规格">
                <div>
                    <!-- 规格列表 -->
                    <div v-for="(specItem,specItemIndex) in tableColumnList.goodsSpecs"
                        :key="specItemIndex">
                        <div>
                            <div class="spe-name">
                                规格名：
                            </div>
                            <div>
                                <el-input v-model.trim="specItem.attr" ref="specValueInput"
                                    @focus="attrFocus(specItem.attr)" @blur="attrBlur(specItem.attr)"></el-input>
                            </div>
                        </div>
                        <div>
                            <div class="spe-value">
                                规格值：
                            </div>
                            <div>
                                <div 
                                    v-for="(valueItem,valueItemIndex) in specItem.valueList"
                                    :key="valueItemIndex">
                                    <el-input v-model.trim="specItem.valueList[valueItemIndex].title"
                                        ref="attrValueInput"
                                        width="50%"
                                        @focus="attrValueFocus(specItem.valueList[valueItemIndex].title)"
                                        @blur="newAttrValueBlur(specItem.attr, specItem.valueList[valueItemIndex].title,specItemIndex,valueItemIndex)">
                                    </el-input>
                                    <div class="el-icon-delete"
                                        v-if="specItem.valueList.length > 0"
                                        @click="removeSpecValue(specItemIndex,valueItemIndex, specItem.attr, specItem.valueList[valueItemIndex].title)">
                                    </div>
                                </div>
                                <div>
                                    <el-button @click="addSpecValue(specItemIndex)"
                                        icon="el-icon-plus"
                                        :disabled="specItem.valueList.filter(value => value.title === '').length > 0">
                                        添加规格值</el-button>
                                </div>
                            </div>
                        </div>
                        <div class="el-icon-delete" @click="removeSpec(specItemIndex)"></div>
                        <br>
                        删除规格
                    </div>
                    <!-- 添加区 -->
                    <div style="margin-bottom: 0;">                      
                        <el-popover placement="top" width="240" v-model="add_popover_bool"
                            @after-enter="$refs.addValueInput.focus()">
                            <div style="display: flex; grid-gap: 10px;">
                                <el-input ref="addValueInput" v-model.trim="add_value"
                                    @keyup.enter.native="addSpec()" />
                                <el-button type="primary" @click="addSpec()">确定</el-button>
                            </div>
                            <el-button slot="reference" type="primary" style="margin-right: 75px;">添加规格项
                            </el-button>
                        </el-popover>
                        <el-button type="danger" :disabled="!tableColumnList.table_data.length" @click="clearALL">清空全部规格项</el-button>
                    </div>
                </div>
            </el-form-item>
            <hr>
            <el-form-item label="规格明细">
                <el-table :data="tableColumnList.table_data" border>
                    <el-table-column :label="specItem.attr" 
                        v-for="(specItem,itemIndex) in tableColumnList.goodsSpecs" :key="itemIndex"
                        :prop="specItem.attr">
                    </el-table-column>
                </el-table>
                <p>数量：{{ specCounts }}条</p>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
export default {
    name:"skuCom",
    data(){
        return{
            //表单
            tableHeaderList: [], //表格列
            tableColumnList: {
                goodsSpecs: [],
                table_data: [], //表格中的数据
            },
            add_value: "", //添加属性的input
            add_popover_bool: false, //添加属性的小弹窗

            showSelectSpecImageDialog: false,
            selectedingImageSpecIndex: null, //获取上传图片的表格索引
            sp_id: -1, // 新增table_data的id
            sc_id: -1, // 新增规格名的id
            specCounts: 0
        }
    },
    computed: {
        // 已添加的规格项
        selectedAttr() {
            return this.tableColumnList.goodsSpecs.map(x => x.attr)
        }
    },
    methods:{
        /**
         * 添加规格项
         */
        addSpec() {
            //判断是否有值
            if (!this.add_value) return this.$message({message:"请输入规格名", type:'warning'})
            //判断是否有相同名称规格
            if (this.selectedAttr.includes(this.add_value)) {
                this.$message({
                    message: '不能添加相同规格名',
                    type: 'warning'
                });
                return
            }
            //id递增
            this.sc_id++;
            this.tableColumnList.goodsSpecs.push({
                id: 'SC' + this.sc_id,
                attr: this.add_value,
                valueList: [{
                    "id": 'SV0&' + 'SC' + this.sc_id,
                    "specid": 'SC' + this.sc_id,
                    "title": ""
                }],
            });

            // 生成处理表头数据和表格数据
            this.generateTableColumn()
            this.traverseSku()

            this.add_popover_bool = false
            this.add_value = ''
        },
        /***
         * 清除所有的规格项
         */
        clearALL() {
            this.$confirm('此操作将清除所有的规格项, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.tableColumnList.goodsSpecs = [];
                // 生成处理表头数据和表格数据
                this.generateTableColumn()
                this.traverseSku()
                this.$message({
                    type: 'success',
                    message: '删除成功!'
                });
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                })        
            })
        },

        /**
         * 添加对应规格值
         */
        async addSpecValue(specItemIndex) {
            this.tableColumnList.goodsSpecs[specItemIndex].valueList.push({
                "id": 'SV' + this.tableColumnList.goodsSpecs[specItemIndex].valueList.length + '&' + this.tableColumnList.goodsSpecs[specItemIndex].id,
                "specid": this.tableColumnList.goodsSpecs[specItemIndex].id,
                "title": ""
            });
            // 让新增的输入框获得焦点
            await this.$nextTick()
            this.$refs.attrValueInput[this.$refs.attrValueInput.length - 1].focus()
        },
        
        /**
         * 属性获得焦点时 得到旧的值 存一下
         */
        attrFocus(oldVal) {
            this.old_attr = oldVal
        },

        /**
         * 属性失去焦点时
         */
        attrBlur(newVal) {
            // 如果 '新值等于旧值' 或者 '空' 什么也不做
            if (newVal === this.old_attr || newVal === '') return

            // 生成处理表头数据和表格数据
            this.generateTableColumn()
            this.traverseSku()
        },
        /**
         * 属性值获得焦点时 得到旧的值 在输入框失去焦点的时候, 如果值没有变化, 则什么也不做
         * @oldVal 属性值
         */
        attrValueFocus(oldVal) {
           this.old_attr_value = oldVal
        },

        /**
         * 属性值失去焦点时, 操作表格数据 (新版本 可以实现无限个规格)
         * @curr_attr 属性
         * @newVal 属性值
         * @specItemIndex   规格项下标
         * @valueItemIndex 下标
         */ 
        newAttrValueBlur(curr_attr, newVal, specItemIndex, valueItemIndex) {
            if (curr_attr === '') return
            if (newVal === this.old_attr_value) return
            let value_num = this.tableColumnList.goodsSpecs[specItemIndex].valueList.filter(item => item.title == newVal)
            if (value_num.length > 1) {
                this.$message({
                    message: `规格值不能重复`,
                    type: 'warning'
                })
                this.tableColumnList.goodsSpecs[specItemIndex].valueList[valueItemIndex].title = "";
                return
            }

            //  这里根据规格生成的笛卡尔积计算出table中需要改变的行索引 ( 包含新增和修改 )
            let cartesian_arr = this.tableColumnList.goodsSpecs
            // console.log(cartesian_arr)
            let change_idx_arr = [] // 需要被改变的行的索引
            for (let i in cartesian_arr) {
                if (cartesian_arr[i][curr_attr] === newVal) change_idx_arr.push(Number(i))
            }

            // 新的表格应该有的长度与现有的表格长度比较, 区分新增还是修改
            let length_arr = this.tableColumnList.goodsSpecs.map(x => x.valueList.length)
            let new_table_length = length_arr.reduce((acc, curr_item) => acc * curr_item) // 新的表格数据长度 求乘积
            let old_table_length = this.tableColumnList.table_data.length // 旧的表格数据长度

            this.specCounts = new_table_length
            // 如果是修改
            if (new_table_length === old_table_length) {
                this.tableColumnList.table_data.forEach((item, index) => {
                    if (change_idx_arr.includes(index)) this.tableColumnList.table_data[index][curr_attr] = newVal
                })
                this.traverseSku()
                return
            }
            // 如果是新增
            if (new_table_length > old_table_length) {
                // 得到当前属性的当前值和其他规格的 goodsSpecs, 构造新的表格数据
                let other_sku_arr = this.tableColumnList.goodsSpecs.map(item => {
                    if (item.attr !== curr_attr){
                       this.traverseSku()
                       return item
                    } 
                    else {
                        this.traverseSku()
                        return {
                            id: this.tableColumnList.goodsSpecs[specItemIndex].id, attr: item.attr, valueList: [{
                                "id": 'SV' + this.tableColumnList.goodsSpecs[specItemIndex].valueList.length + '&' + this.tableColumnList.goodsSpecs[specItemIndex].id,
                                "specid": this.tableColumnList.goodsSpecs[specItemIndex].id,
                                "title": newVal
                            }]
                        }
                    }
                })
                // 得到新增的表格数据
                let ready_map = this.generateBaseData(other_sku_arr)
                let new_table_data = this.mergeTableData(ready_map)

                change_idx_arr.forEach((item_idx, index) => {
                    this.tableColumnList.table_data.splice(item_idx, 0, new_table_data[index])
                })

                this.tableColumnList.table_data.reverse().reverse()
                this.traverseSku()
                console.log(this.tableColumnList.table_data);
            }
        },

        /**
         * 删除规格
         */
        removeSpec(specItemIndex) {
            this.$confirm('确定删除该规格吗?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.tableColumnList.goodsSpecs.splice(specItemIndex, 1);
                // 生成处理表头数据和表格数据
                this.generateTableColumn()
                this.traverseSku()
                this.$message({
                    type: 'success',
                    message: '删除成功!'
                });
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
        },

        /**
         * @specItemIndex
         * @valueItemIndex
         * @attr_name
         * @attr_val
         * 删除规格值
         */
        removeSpecValue(specItemIndex, valueItemIndex, attr_name, attr_val) {
            this.tableColumnList.goodsSpecs[specItemIndex]["valueList"].splice(valueItemIndex, 1);

            // 删除table行
            let data_length = this.tableColumnList.table_data.length
            for (let i = 0; i < data_length; i++) {
                if (this.tableColumnList.table_data[i][attr_name] == attr_val) {
                    this.tableColumnList.table_data.splice(i, 1)
                    //长度 索引对应减小
                    i = i - 1
                    data_length = data_length - 1
                }
            }
            this.specCounts = this.tableColumnList.table_data.length
        },
        /**
         * 根据 this.tableColumnList.goodsSpecs 生成表格列, tableHeaderList 用于 el-table-column 的 v-for
         */
        generateTableColumn() {
            this.tableHeaderList = this.tableColumnList.goodsSpecs
        },

        /**
         * 合并 goodsSpecs  返回整个表格数据数组
         */
        mergeTableData(arr) {
            return arr.map((item) => {
                this.sp_id++
                return { ...item }
            })
        },

        /**
         * 遍历 goodsSpecs 生成表格数据
         */ 
        traverseSku() {
            let ready_map = this.generateBaseData(this.tableColumnList.goodsSpecs)
            this.tableColumnList.table_data = this.mergeTableData(ready_map)
            //总数
            this.specCounts = this.tableColumnList.table_data.length
        },

        /**
         * 实现笛卡尔积  
         * 入参是: this.tableColumnList.goodsSpecs 
         * 传入的数组 '为空', '长度为1', '长度大于1' 三种情况 分别处理
         */

        generateBaseData(arr) {
            //为空
            if (arr.length === 0) return []
            //长度为1
            if (arr.length === 1) {
                let [item_spec] = arr
                return item_spec.valueList.map(x => {
                    return {
                        [item_spec.attr]: x.title
                    }
                })
            }
            //长度大于1
            if (arr.length >= 1) {
                return arr.reduce((accumulator, spec_item) => {
                    // accumulator判断之前的数据是  整合的规格数组还是单个规格项
                    let acc_value_list = Array.isArray(accumulator.valueList) ? accumulator.valueList : accumulator
                    // 获取当前项的valueList
                    let item_value_list = spec_item.valueList
                    let result = []

                    acc_value_list.forEach( (item,index) => {
                        item_value_list.forEach((v,i) => {
                            let temp_data = {}
                            if(!acc_value_list[index].title){
                                temp_data = {
                                    ...acc_value_list[index],
                                    [spec_item.attr]: item_value_list[i].title
                                }
                            }else {
                                temp_data[accumulator.attr] = acc_value_list[index].title
                                temp_data[spec_item.attr] = item_value_list[i].title
                            }
                            result.push(temp_data)
                        })
                    })
                    return result
                })
            }
        },
    }

}
</script>

<style scoped>
.spe-name {
    font-size: 20px;
    color: #e91010;
}
.spe-value {
    font-size: 16px;
    color: #0caaf9;
}
</style>