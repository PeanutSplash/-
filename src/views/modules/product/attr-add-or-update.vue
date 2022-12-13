<template>
        <el-dialog :title="!dataForm.attrId ? '新增商品' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
            <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
                label-width="80px">
                <el-form-item label="属性名" prop="attrName">
                    <el-input v-model="dataForm.attrName" placeholder="属性名"></el-input>
                </el-form-item>
                <el-form-item label="启用状态" prop="enable">
                    <el-input v-model="dataForm.enable" placeholder="[0 - 禁用，1 - 启用]"></el-input>
                </el-form-item>
                <el-form-item label="商品id" prop="skuId">
                    <el-input v-model="dataForm.skuId" placeholder="sku商品id"></el-input>
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
                attrId: 0,
                attrName: '',
                enable: '',
                skuId: ''
            },
            dataRule: {
                attrName: [
                    { required: true, message: '属性名不能为空', trigger: 'blur' }
                ],
                enable: [
                    { required: true, message: '启用状态', trigger: 'blur' }
                ],
                skuId: [
                    { required: true, message: 'sku商品id不能为空', trigger: 'blur' }
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
                        url: this.$http.adornUrl(`/product/attr/${!this.dataForm.attrId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData({
                            'attrId': this.dataForm.attrId || undefined,
                            'attrName': this.dataForm.attrName,
                            'enable': this.dataForm.enable,
                            'skuId': this.dataForm.skuId
                        })
                    }).then(({ data }) => {
                        if (data && data.code === 0) {
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
  