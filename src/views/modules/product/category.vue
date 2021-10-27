<!--  -->
<template>
    <div>
        <el-tree :data="data" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
            :default-expanded-keys="expandedKey">
            <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)">
                        添加
                    </el-button>

                    <el-button type="text" size="mini" @click="() => edit(data)">
                        编辑
                    </el-button>

                    <el-button v-if="node.childNodes.length==0" type="text" size="mini"
                        @click="() => remove(node, data)">
                        删除
                    </el-button>
                </span>
            </span>
        </el-tree>

        <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>


            <el-form :model="category">
                <el-form-item label="图标">
                    <el-input v-model="category.icon" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>


            <el-form :model="category">
                <el-form-item label="计量单位">
                    <el-input v-model="category.productUnit" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="submitData">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
    //这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
    //例如：import 《组件名称》 from '《组件路径》';
    export default {
        //import引入的组件需要注入到对象中才能使用
        components: {},
        props: {},
        data() {
            return {
                title: "",
                dialogType: "",
                category: {
                    name: "",
                    parentCid: 0,
                    catLevel: 0,
                    showStatus: 1,
                    sort: 0,
                    catId: null,
                    productUnit: "",
                    icon: ""
                },
                dialogVisible: false,
                data: [],
                expandedKey: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            };
        },
        //方法集合
        methods: {
            getMenus() {
                //this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/product/category/list/tree'),
                    method: 'get'
                }).then(({ data }) => {
                    console.log("成功", data.data)
                    this.data = data.data;
                })
            },
            submitData() {
                if (this.dialogType == "add") {
                    this.addCategory();
                }
                if (this.dialogType == "edit") {
                    this.editCategory();
                }
            },
            //修改
            edit(data) {
                //表示确定的是编辑
                this.dialogType = "edit";
                //对话框显示
                this.title = "编辑分类";
                //控制台打印
                console.log("要修改数据", data);
                //打开对话框
                this.dialogVisible = true;
                //数据回显

                this.$http({
                    url: this.$http.adornUrl(`/product/category//info/${data.catId}`),
                    method: 'get',
                }).then(({ data }) => {
                    console.log("回显数据", data);
                    this.category.name = data.data.name;
                    this.category.catId = data.data.catId;
                    this.category.icon = data.data.icon;
                    this.category.productUnit = data.data.productUnit;
                    this.category.parentCid = data.data.parentCid;
                })
            },
            append(data) {
                this.dialogType = "add";
                this.title = "添加分类";
                console.log("append", data);
                this.dialogVisible = true;
                this.category.parentCid = data.catId;
                this.category.catLevel = data.catLevel * 1 + 1;
                this.category.name = "";
                this.category.catId = null;
                this.category.icon = "";
                this.category.productUnit = "";
            },

            //编辑提交
            editCategory() {
                //只发送其中四个数据
                var { catId, name, icon, productUnit } = this.category;
                this.$http({
                    url: this.$http.adornUrl('/product/category/update'),
                    method: 'post',
                    data: this.$http.adornData({ catId, name, icon, productUnit }, false)
                }).then(({ data }) => {
                    this.$message({
                        type: 'success',
                        message: '修改成功!'
                    });
                    //关闭对话框
                    this.dialogVisible = false;
                    //刷新数据
                    this.getMenus();
                    //回显  this.category和console.log("提交数据", this.category);输出的一致
                    this.expandedKey = [this.category.parentCid];
                });
            },

            //添加分类确定
            addCategory() {
                console.log("提交数据", this.category);
                this.$http({
                    url: this.$http.adornUrl('/product/category/save'),
                    method: 'post',
                    data: this.$http.adornData(this.category, false)
                }).then(({ data }) => {
                    this.$message({
                        type: 'success',
                        message: '添加成功!'
                    });
                    //关闭对话框
                    this.dialogVisible = false;
                    //刷新数据
                    this.getMenus();
                    //回显  this.category和console.log("提交数据", this.category);输出的一致
                    this.expandedKey = [this.category.parentCid];
                });
            },
            remove(node, data) {
                var ids = [data.catId]
                //删除前弹框提示
                this.$confirm(`此操作将删除【${data.name}】菜单, 是否继续?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    //删除
                    this.$http({
                        url: this.$http.adornUrl('/product/category/delete'),
                        method: 'post',
                        data: this.$http.adornData(ids, false)
                    }).then(({ data }) => {
                        console.log("删除成功");
                        //刷新页面
                        this.getMenus();
                        //重新展开
                        this.expandedKey = [node.parent.data.catId]
                    });
                    consloe.log("remove", node, data);
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                }).catch(() => {

                });

            }
        },
        //监听属性 类似于data概念
        computed: {},
        //监控data中的数据变化
        watch: {},
        //生命周期 - 创建完成（可以访问当前this实例）
        created() {
            this.getMenus();
        },
        //生命周期 - 挂载完成（可以访问DOM元素）
        mounted() {

        },
        beforeCreate() { }, //生命周期 - 创建之前
        beforeMount() { }, //生命周期 - 挂载之前
        beforeUpdate() { }, //生命周期 - 更新之前
        updated() { }, //生命周期 - 更新之后
        beforeDestroy() { }, //生命周期 - 销毁之前
        destroyed() { }, //生命周期 - 销毁完成
        activated() { }, //如果页面有keep-alive缓存功能，这个函数会触发
    }
</script>

//@import url(); 引入公共css类
<style scoped>

</style>