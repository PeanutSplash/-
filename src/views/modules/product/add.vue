<template>
    <div>
        <el-form :inline="true" :model="dataForm">
            <el-form-item>
                <el-button v-if="isAuth('sys:product:save')" type="primary">新增</el-button>
                1
            </el-form-item>
        </el-form>
    </div>
</template>
import AddOrUpdate from './menu-add-or-update'
import { treeDataTranslate } from '@/utils'
export default {
    data() {
        return {
            dataForm: {},
            dataList: [],
            dataListLoading: false,
            addOrUpdateVisible: false
        }
    },
    components: {
        AddOrUpdate
    },
    activated() {
        this.getDataList()
    },
    methods: {
        // 获取数据列表
        getDataList() {
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl('/sys/menu/list'),
                method: 'get',
                params: this.$http.adornParams()
            }).then(({ data }) => {
                this.dataList = treeDataTranslate(data, 'menuId')
                this.dataListLoading = false
            })
        },
        // 新增 / 修改
        addOrUpdateHandle(id) {
            this.addOrUpdateVisible = true
            this.$nextTick(() => {
                this.$refs.addOrUpdate.init(id)
            })
        },
        // 删除
        deleteHandle(id) {
            this.$confirm(`确定对[id=${id}]进行[删除]操作?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl(`/sys/menu/delete/${id}`),
                    method: 'post',
                    data: this.$http.adornData()
                }).then(({ data }) => {
                    if (data && data.code === 0) {
                        this.$message({
                            message: '操作成功',
                            type: 'success',
                            duration: 1500,
                            onClose: () => {
                                this.getDataList()
                            }
                        })
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }).catch(() => { })
        }
    }
}
</script> -->
<script>
</script>