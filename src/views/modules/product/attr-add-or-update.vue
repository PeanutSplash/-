<template>
    <el-dialog :title="!dataForm.attrId ? '新增规格组' : '修改规格组信息'" :close-on-click-modal="false" :visible.sync="visible">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
            label-width="80px">
            <el-form-item label="名称" prop="attrName">
                <el-input v-model="dataForm.attrName" placeholder="规格组名称"></el-input>
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
                attrName: '',
            },
            dataRule: {
                attrName: [
                    { required: true, message: '规格组名称不能为空', trigger: 'blur' }
                ],
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
                    var data = []
                    data[0] = this.dataForm.attrName
                    this.$http({
                        url: this.$http.adornUrl(`product/addAttrGroup`),
                        method: 'post',
                        data: this.$http.adornData(data)
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
