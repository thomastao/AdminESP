<template>
  <div id="org" class="app-container calendar-list-container">
    <el-row  v-loading.body="listLoading">
      <el-col :span="6">
        <el-input placeholder="输入关键字进行过滤" v-model="filterText">
        </el-input>
        <el-tree class="filter-tree" style="margin-top:10px;"  :data="data2" :props="defaultProps" :default-expand-all="true"  :filter-node-method="filterNode" :default-expanded-keys="[0,1]" ref="tree2" @node-click="clickTree" @node-expand="expandTree">
        </el-tree>
      </el-col>
      <el-col :span="18">
        <div class="right-layout-from">
          <div v-if="this.state == 'see'" class="top-button">
            <el-button-group>
              <el-button type="primary" native-type="submit"  @click="toCreate()" icon="plus">新增下级机构</el-button>
              <el-button type="primary"  @click="toUpdate()" icon="edit">编辑当前机构</el-button>
              <el-button type="danger"  @click="toDeleted()" icon="delete">删除</el-button>
            </el-button-group>
          </div>
          <div  class="top-button">
            <el-button v-if="this.state == 'add'" type="primary" native-type="submit"  @click="createOrg('orgFrom')">保存</el-button>
            <el-button v-if="this.state == 'edit'" type="primary" native-type="submit"  @click="updateOrg('orgFrom')">保存</el-button>
          </div>
          <el-card class="box-card" style="margin-bottom: 10px;">
            <el-form :model="orgFrom" :rules="rules" ref="orgFrom" label-width="120px" class="demo-ruleForm" >
              <el-row aria-disabled="">
                <el-col :span="11">
                  <el-form-item  label="组织机构名称" prop="name">
                    <el-input v-model="orgFrom.name" v-if="this.state == 'see'" disabled="disabled" ></el-input>
                    <el-input v-model="orgFrom.name" v-else ></el-input>
                  </el-form-item>
                </el-col>
                <el-col :span="2">
                  <div class="grid-content"></div>
                </el-col>
                <el-col :span="11">
                  <el-form-item label="组织机构编码" prop="code">
                    <el-input v-model="orgFrom.code" v-if="this.state == 'see'" disabled="disabled" ></el-input>
                    <el-input v-model="orgFrom.code" v-else></el-input>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row>
                <el-col :span="11">
                  <el-form-item label="上级组织机构" prop="parentname">
                    <el-input v-model="orgFrom.parentname" disabled="disabled"></el-input>
                  </el-form-item>
                </el-col>
                <el-col :span="2">
                  <div class="grid-content"></div>
                </el-col>
                <el-col :span="11">
                  <el-form-item label="机构类别" prop="orgType">
                  <el-select v-model="orgFrom.orgType" placeholder="请选择机构类别" v-if="this.state == 'see'" disabled="disabled" >
                    <el-option label="类别一" value="key_one"></el-option>
                    <el-option label="类别二" value="key_two"></el-option>
                    <el-option label="类别三" value="key_three"></el-option>
                    <el-option label="类别四" value="key_four"></el-option>
                  </el-select>
                  <el-select v-model="orgFrom.orgType" placeholder="请选择机构类别" v-else >
                    <el-option label="类别一" value="key_one"></el-option>
                    <el-option label="类别二" value="key_two"></el-option>
                    <el-option label="类别三" value="key_three"></el-option>
                    <el-option label="类别四" value="key_four"></el-option>
                  </el-select>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-form-item label="层级编码" prop="levelCode">
                <el-input v-model="orgFrom.levelCode" disabled="disabled"></el-input>
              </el-form-item>
              <el-form-item label="机构说明" prop="note">
                <el-input type="textarea" v-model="orgFrom.note" v-if="this.state == 'see'" disabled="disabled"></el-input>
                <el-input type="textarea" v-model="orgFrom.note" v-else></el-input>
              </el-form-item>
            </el-form>
          </el-card>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
  import { orgTree, getObj, addObj, putObj, getNextCode, checkCode, delObj } from 'api/admin/org/index'
  export default {
    watch: {
      filterText(val) {
        this.$refs.tree2.filter(val);
      }
    },
    created() {
      // 初始化左侧树
      this.getOrgTree()
    },
    methods: {
      initObj() {
        return {
          name: '',
          code: '',
          levelCode: '',
          orgType: '', // 机构类别
          deleted: '',
          note: '',
          parentid: ''
        }
      },
      getOrgTree() {
        this.listLoading = true
        var that = this
        orgTree(this.treeForm).then(response => {
          console.log(response.data)
          that.$set(that, 'data2', JSON.parse(response.data))
           // 获取左侧树第一个节点，初始化页面
          // this.getObj(JSON.parse(response.data)[0].id)
          this.dataTOForm(JSON.parse(response.data)[0])
          that.listLoading = false
        })
      },
      getObj(id) {
        var that = this
        getObj(id).then(response => {
          console.log(response.data)
          that.$set(that, 'orgFrom', response.data)
          that.state = 'see'
        })
      },
      toCreate() {
        var yid = this.orgFrom.id
        var lCode = this.orgFrom.levelCode
        var pName = this.orgFrom.name
        this.resetTemp()
        this.orgFrom.parentid = yid
        this.orgFrom.parentname = pName
        this.nextFrom.parentId = yid
        this.nextFrom.levelCode = lCode
        this.getNextCode()
        this.state = 'add'
      },
      getNextCode() {
        var that = this
        getNextCode(this.nextFrom).then(response => {
          console.log(response.data)
          that.orgFrom.levelCode = response.data
        })
      },
      toUpdate() {
        this.state = 'edit'
      },
      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1 || data.lspell.indexOf(value) !== -1 || data.uspell.indexOf(value) !== -1 || data.initials.indexOf(value) !== -1;
      },
      clickTree(data) {
        console.log(data)
        this.dataTOForm(data)
        // this.getObj(data.id)
      },
      dataTOForm(data) {
        this.orgFrom.parentname = data.parentName
        this.orgFrom.parentid = data.parentid
        this.orgFrom.name = data.label
        this.orgFrom.deleted = data.deleted
        this.orgFrom.levelCode = data.level_code
        this.orgFrom.id = data.id
        this.orgFrom.code = data.code
        this.orgFrom.note = data.note
        this.orgFrom.orgType = data.org_type
        this.state = 'see'
      },
      expandTree(data) {
        console.log(data)
      },
      updateOrg(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            putObj(this.orgFrom.id, this.orgFrom).then(() => {
              this.getOrgTree()
              this.state = 'see'
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            })
          } else {
            return false
          }
        })
      },
      createOrg(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            this.orgFrom.deleted = 'N'
            addObj(this.orgFrom).then(() => {
              this.getOrgTree()
              this.state = 'see'
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            })
          } else {
            return false
          }
        })
      },
      resetOrg(formName) {
        this.$refs[formName].resetFields();
      },
      resetTemp() {
        this.orgFrom = this.initObj();
      },
      toDeleted() {
        this.$confirm('此操作将永久删除, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.codeFom.id = this.orgFrom.id
          delObj(this.codeFom).then(data => {
            if (data) {
              this.getOrgTree();
              this.$notify({
                title: '成功',
                message: '删除成功',
                type: 'success',
                duration: 2000
              });
            } else {
              this.$notify({
                title: '失败',
                message: '请先删除下级组织机构',
                type: 'error',
                duration: 2000
              });
            }
          });
        });
      }
    },
    data() {
      const validateCode = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请填写编码'))
        }
        this.codeFom.code = value
        this.codeFom.id = this.orgFrom.id
        checkCode(this.codeFom).then(result => {
          if (!result) {
            callback(new Error('该编码已存在'))
          } else {
            callback()
          }
        });
      };
      return {
        filterText: '',
        state: 'see',
        data2: [{
          id: 1,
          label: '一级 1'
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1'
          }, {
            id: 6,
            label: '二级 2-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        orgFrom: {
          id: '',
          name: '',
          code: '',
          levelCode: '',
          orgType: '', // 机构类别
          deleted: 'N',
          note: '',
          parentid: '',
          parentname: ''
        },
        treeForm: {
          parentid: 0
        },
        nextFrom: {
          parentId: 0,
          levelCode: ''
        },
        codeFom: {
          id: '',
          code: ''
        },
        rules: {
          name: [
            { required: true, message: '请输入组织机构名称', trigger: 'blur' }
          ],
          code: [
            { required: true, validator: validateCode, trigger: 'blur' }
          ],
          orgType: [
            { required: true, message: '请选择机构类别', trigger: 'blur' }
          ]
        },
        listLoading: false
      };
    }
  }
</script>
<style>
.right-layout-from{
  margin-left: 20px;
}
.top-button{
  margin-bottom: 20px;
}
.tree-right-from{
  width: 70%;
  margin: 3%;
}
.grid-content{
    border-radius: 4px;
    min-height: 36px;
}
.on-border{
  border: none
}
</style>
