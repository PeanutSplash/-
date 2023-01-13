<template>
    <el-dialog :title="!dataForm.attrId ? '新增商品' : '修改商品信息'" :close-on-click-modal="false" :visible.sync="visible">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
            label-width="80px">
            <el-form-item label="商品名称" prop="spuName">
                <el-input v-model="dataForm.spuName" placeholder="商品名称"></el-input>
            </el-form-item>
            <el-form-item label="商品描述" prop="spuDescription">
                <el-input v-model="dataForm.spuDescription" placeholder="商品描述"></el-input>
            </el-form-item>
            <el-form-item label="原价" prop="spuPrice">
                <el-input v-model="dataForm.spuPrice" placeholder="原价"></el-input>
            </el-form-item>
            <el-form-item label="折扣价" prop="spuDiscount">
                <el-input v-model="dataForm.spuDiscount" placeholder="折扣价"></el-input>
            </el-form-item>
            <el-form-item label="商品品牌" prop="brandId">
                <el-select v-model="dataForm.brandId" placeholder="请选择品牌">
                    <el-option v-for="item in brandOptions" :key="item.brandId" :label="item.name"
                        :value="item.brandId">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="商品分类" prop="catalogId">
                <el-cascader v-model="dataForm.catalogIdValue" :clearable="true" :props="defaultParams"
                    :options="catalogOptions" @change="getCatId"></el-cascader>
            </el-form-item>

            <el-form-item label="是否上架" prop="publishStatus">
                <el-select v-model="dataForm.publishStatus" placeholder="请选择上架状态">
                    <el-option v-for="item in publishStatusOptions" :key="item.value" :label="item.label"
                        :value="item.value">
                    </el-option>
                </el-select>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            dataForm: {
                spuName: '',
                spuDescription: '',
                spuPrice: '',
                spuDiscount: '',
                brandId: '',
                catalogId: '',
                catalogIdValue: [],
                publishStatus: '',
            },
            /* 选择器的选择参数 */
            publishStatusOptions: [{
                value: '1',
                label: '是'
            }, {
                value: '0',
                label: '否'
            }],
            brandOptions: [],
            catalogOptions: [],
            defaultParams: {//级联选择器的设置参数
                expandTrigger: 'hover',
                label: 'name',
                value: 'catId',
                children: 'children',
            },
            dataRule: {
                spuName: [
                    { required: true, message: '商品名不能未空', trigger: 'blur' }
                ],
                spuDescription: [
                    { required: true, message: '商品描述不能为空', trigger: 'blur' }
                ],
                spuPrice: [
                    { required: true, message: '原价不得为空', trigger: 'blur' }
                ],
                spuDiscount: [
                    { required: true, message: '折扣价不得为空', trigger: 'blur' }
                ],
                catalogId: [
                    { required: true, message: '分类id不得为空', trigger: 'blur' }
                ],
                brandId: [
                    { required: true, message: '请选择品牌', trigger: 'blur' }
                ],
                publishStatus: [
                    { required: true, message: '上架状态', trigger: 'blur' }
                ]
            }
        }
    },
    created() {
        /* 获取选择器中的内容 */
        this.getProductType()
        this.getBrandList()
    },
    methods: {
        init(id) {
            this.dataForm.attrId = id || 0
            this.visible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].resetFields()
                if (this.dataForm.attrId) {
                    this.$http({
                        url: this.$http.adornUrl(`/product/attr/info/${this.dataForm.attrId}`),
                        method: 'get',
                        params: this.$http.adornParams()
                    }).then(({ data }) => {
                        if (data && data.code === 0) {
                            this.dataForm.attrName = data.attr.attrName
                            this.dataForm.enable = data.attr.enable
                            this.dataForm.skuId = data.attr.skuId
                        }
                    })
                }
            })
        },
        getProductType() {// 递归方法(分类)
            // 这是从后台获取数据
            this.$http.get("https://lingqi.schoolhelp.top/LMDApi/lionmountaindistribution/category")
                .then(res => {
                    // 调用递归方法，去除级联数据后将数据赋值给级联选择器
                    this.catalogOptions = this.getTreeData(res.data.data);
                });
        },
        getCatId() {//识别选择的catid
            if (this.dataForm.catalogIdValue.length != 0) {
                var index = this.dataForm.catalogIdValue.length - 1//取最后一位
                this.dataForm.catalogId = this.dataForm.catalogIdValue[index]
            } else {
                this.dataForm.catalogId = ''
            }
        },
        getTreeData(data) {//递归删除空childres
            // 循环遍历json数据
            for (var i = 0; i < data.length; i++) {
                if (data[i].children.length < 1) {
                    // children若为空数组，则将children设为undefined
                    data[i].children = undefined;
                } else {
                    // children若不为空数组，则继续 递归调用 本方法
                    this.getTreeData(data[i].children);
                }
            }
            return data;
        },
        getBrandList() {//品牌列表
            this.$http({
                url: this.$http.adornUrl('product/listBrand?current=' + 1 + '&&pageSize=' + 100),
                method: 'get',
            }).then(({ data }) => {
                if (data && data.code === 200) {
                    this.brandOptions = data.data.records
                } else {
                    this.dataList = []
                }
            })
        },
        dataFormSubmit() {// 表单提交
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`product/addSpu`),
                        method: 'post',
                        data: this.$http.adornData({
                            spuName: this.dataForm.spuName,
                            spuDescription: this.dataForm.spuDescription,
                            spuPrice: parseInt(this.dataForm.spuPrice) * 100,
                            spuDiscount: parseInt(this.dataForm.spuDiscount) * 100,
                            catalogId: parseInt(this.dataForm.catalogId),
                            brandId: parseInt(this.dataForm.brandId),
                            publishStatus: parseInt(this.dataForm.publishStatus),
                        })
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                }
            })
        }
    }
}
</script>
