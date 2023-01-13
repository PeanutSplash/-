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
            <el-form-item label="分类id" prop="catalogId">
                <el-input v-model="dataForm.catalogId" placeholder="分类id"></el-input>
            </el-form-item>
            <el-form-item label="品牌id" prop="brandId">
                <el-input v-model="dataForm.brandId" placeholder="品牌id"></el-input>
            </el-form-item>
            <el-form-item label="是否上架" prop="publishStatus">
                <el-input v-model="dataForm.publishStatus" placeholder="1上架,0不上架"></el-input>
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
                catalogId: '',
                brandId: '',
                publishStatus: ''
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
                    { required: true, message: '品牌id不得为空', trigger: 'blur' }
                ],
                publishStatus: [
                    { required: true, message: '上架状态', trigger: 'blur' }
                ]
            }
        }
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
        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`product/addSpu`),
                        method: 'post',
                        data: this.$http.adornData({
                            spuName: this.dataForm.spuName,
                            spuDescription: this.dataForm.spuDescription,
                            spuPrice: parseInt(this.dataForm.spuPrice),
                            spuDiscount: parseInt(this.dataForm.spuDiscount),
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
  