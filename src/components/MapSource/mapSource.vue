<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 探头数据
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <!-- 操作栏 -->
            <div class="handle-box">
                <el-button type="success" icon="el-icon-document-add" class="handle-del mr10" @click="addProbe">新增探头</el-button>
                <el-input v-model="query.name" placeholder="用户名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            </div>
            <!-- 表格正文 -->
            <el-table :data="tableData" border class="table" ref="multipleTable" header-cell-class-name="table-header" @selection-change="handleSelectionChange">
                <!-- <el-table-column type="selection" width="55" align="center"></el-table-column> -->
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="mId" label="markId" width="75" align="center"></el-table-column>
                <el-table-column prop="address" width="200" label="地址" align="center"></el-table-column>
                <el-table-column prop="latitude" label="维度" align="center"></el-table-column>
                <el-table-column prop="longitude" label="经度" align="center"></el-table-column>
                <el-table-column label="具体位置" align="center">
                    <template slot-scope="scope">
                        <el-tag class="cruse" @click="seeBig(scope.$index, scope.row)">点击查看</el-tag>
                        <!-- <el-button type="text" icon="el-icon-edit" @click="seeBig(scope.$index, scope.row)">点击查看</el-button> -->
                    </template>
                </el-table-column>
                <el-table-column label="精确度" align="center">
                    <template slot-scope="scope">
                        <el-tag v-if="scope.row.state == 1">精确</el-tag>
                        <el-tag v-else>大体位置</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="date" label="注册时间" align="center"></el-table-column>
                <el-table-column label="操作" width="160" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination background layout="total, prev, pager, next,jumper" :current-page="query.pageIndex" :page-size="query.pageSize" :total="pageTotal"
                    @current-change="handlePageChange"></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="地址">
                    <el-input v-model="form.address"></el-input>
                </el-form-item>
                <el-form-item label="维度">
                    <el-input :value="form.latitude"></el-input>
                </el-form-item>
                <el-form-item label="经度">
                    <el-input :value="form.longitude"></el-input>
                </el-form-item>
                <el-form-item label="精确度">
                    <!-- <el-input v-model="form.state"></el-input> -->
                    <el-input :value="form.state"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 新增弹出框 -->
        <el-dialog title="新增探头" :visible.sync="addVisible" width="30%">
            <el-form :model="form">
                <el-form-item label="地址">
                    <el-input v-model="form.address" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="维度">
                    <el-input v-model="form.latitude"></el-input>
                </el-form-item>
                <el-form-item label="经度">
                    <el-input v-model="form.longitude"></el-input>
                </el-form-item>
                <el-form-item label="精确度">
                    <el-input v-model="form.state"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAdd">确 定</el-button>
            </div>
        </el-dialog>

    </div>
</template>

<script>
    import Schart from 'vue-schart';
    import bus from '../common/bus';
    import { fetchData } from '@/api/index';

    export default {
        name: 'mapSource',
        data() {
            return {
                query: {
                    address: '',
                    name: '',
                    pageIndex: 1,
                    pageSize: 10
                },
                tableData: [],
                multipleSelection: [],
                delList: [],
                editVisible: false,
                addVisible: false,
                pageTotal: 100,
                form: {},
                idx: -1,
                id: -1
            };
        },
        components: {
            Schart
        },
        computed: {
        },
        created() {
            console.log('created')
            this.getData();

        },
        methods: {
            // 获取 easy-mock 的模拟数据
            getData() {
                fetchData(this.query).then(res => {
                    console.log('getData')
                    console.log(res);
                    console.log('getData')
                    this.tableData = res.list;
                    this.pageTotal = res.list.length || 50;
                });
            },
            // 触发搜索按钮
            handleSearch() {
                // this.$set(this.query, 'pageIndex', 1);
                // this.getData();
            },
            // 删除操作
            handleDelete(index, row) {
                // 二次确认删除
                this.$confirm('确定要删除吗？', '提示', {
                    type: 'warning'
                })
                    .then(() => {
                        this.$message.success('删除成功');
                        this.tableData.splice(index, 1);
                    })
                    .catch(() => { });
            },
            // 多选操作
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            delAllSelection() {
            },
            /**
             * 点击查看
            */
            seeBig(index, row) {
                console.log(index)
                console.log(row)
            },
            // 编辑操作
            handleEdit(index, row) {
                this.idx = index;
                this.form = row;
                this.editVisible = true;

                console.log(index)
                console.log(row)
            },
            // 保存编辑
            saveEdit() {
                this.editVisible = false;
                this.$message.success(`修改第 ${this.idx + 1} 行成功`);
                // this.$set(this.tableData, this.idx, this.form);
            },
            //新增探头
            addProbe() {
                this.addVisible = true
            },
            saveAdd() {
                this.addVisible = false
            },
            // 分页导航
            handlePageChange(val) {
                this.$set(this.query, 'pageIndex', val);
                this.getData();
            }
        },

    };
</script>


<style scoped>
    .handle-box {
        margin-bottom: 20px;
    }

    .handle-select {
        width: 120px;
    }

    .handle-input {
        width: 300px;
        display: inline-block;
    }

    .table {
        width: 100%;
        font-size: 14px;
    }

    .red {
        color: #ff0000;
    }

    .mr10 {
        margin-right: 10px;
    }

    .table-td-thumb {
        display: block;
        margin: auto;
        width: 40px;
        height: 40px;
    }

    .cruse {
        cursor: pointer;
    }
</style>