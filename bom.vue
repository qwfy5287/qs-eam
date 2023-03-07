<!--
 * @Author: 梦回清尘 1164391645@qq.com
 * @Date: 2022-10-19 16:27:19
 * @LastEditors: huanglong
 * @LastEditTime: 2023-01-11 11:42:37
 * @FilePath: \ems-vue\main\src\views\eam\asset\assetInfo\bom.vue
 * @Description:设备信息管理-基础信息-设备BOM
-->

<template>
  <div class="bom-back">
    <div class="classify-sty">
      <div class="grid-content bg-purple">
        <span style="margin-right: 10px">设备编号</span>
        <el-input
          v-model="assetData.assetCode"
          :disabled="true"
          placeholder="请输入"
          style="width: 276px !important"
        />
        <span style="margin-left: 30px; margin-right: 10px">设备名称</span>
        <el-input
          v-model="assetData.assetName"
          :disabled="true"
          placeholder="请输入"
          style="width: 276px !important"
        />
      </div>
      <el-row :gutter="20">
        <!-- 左侧bom树形分类 -->
        <el-col :span="5">
          <div class="grid-content bg-purple">
            <div class="classify-left">
              <div class="classify-title">
                <h2>设备部位</h2>
                <div class="eam-tree-btn-box">
                  <IconAdd @click="addBomPart" />
                  <IconSub @click="remove" />
                </div>
              </div>
              <div id="bomTreeId" style="display: block">
                <div style="margin-bottom: 10px">
                  <el-input
                    v-model.trim="filterText"
                    placeholder="请输入部位名称"
                  />
                </div>
                <el-scrollbar style="height: 100%; width: 100%">
                  <el-tree
                    ref="tree"
                    v-loading="bomLoading"
                    class="filter-tree"
                    :data="dataTree"
                    :props="defaultProps"
                    :default-expand-all="true"
                    :filter-node-method="filterNode"
                    :expand-on-click-node="false"
                    highlight-current
                    @node-click="handleNodeClick"
                  >
                    <span class="custom-tree-node" slot-scope="{ node, data }">
                      <span>{{ node.label }}</span>
                      <span>
                        <el-button
                          v-if="data.parentId !== '-1'"
                          type="text"
                          size="mini"
                          @click="() => edit(data)"
                          >修改</el-button
                        >
                      </span>
                    </span>
                  </el-tree>
                </el-scrollbar>
              </div>
            </div>
          </div>
        </el-col>
        <!-- 操作 树形 分组 -->
        <el-dialog
          v-if="treeVisible"
          :title="treeTitle"
          :visible.sync="treeVisible"
          width="30%"
          append-to-body
          :before-close="handleClose"
        >
          <el-form
            ref="formChoose"
            :model="formTree"
            class="demo-form-inline"
            :rules="chooseRules"
          >
            <el-form-item v-if="treeDia === 'edit'" label="父级分类">
              <Treeselect
                v-model="formTree.parentId"
                placeholder="请选择"
                :props="{ checkStrictly: true, value: 'id' }"
                :options="partTree"
                :normalizer="normalizer"
              />
            </el-form-item>

            <el-form-item label="部位编号" prop="boneCode">
              <el-input v-model="formTree.boneCode" />
            </el-form-item>
            <el-form-item label="部位名称" prop="boneName">
              <el-input v-model="formTree.boneName" />
            </el-form-item>
            <!-- <el-form-item label="描述" prop="description">
              <el-input v-model="formTree.description" />
            </el-form-item>-->
            <el-form-item label="顺序号" prop="seq">
              <!-- <el-input v-model="formTree.seq" /> -->
              <el-input-number
                v-model="formTree.seq"
                :min="1"
                label="描述文字"
              />
            </el-form-item>
          </el-form>

          <span slot="footer" class="dialog-footer">
            <el-button @click="handleClose">取 消</el-button>
            <el-button type="primary" @click="handlecConfirm">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 右侧设备表格 -->
        <el-col v-loading="loading" :span="19">
          <div class="grid-content bg-purple">
            <div class="classify-right">
              <div class="table-box">
                <div class="table-content">
                  <div class="grouping-title">
                    <span>上级节点</span>
                    <el-input
                      v-model="bomData.parentName"
                      disabled
                      style="width: 276px !important"
                    />
                    <span style="margin-left: 30px">部位编号</span>
                    <el-input
                      v-model="bomData.boneCode"
                      disabled
                      style="width: 276px !important"
                    />
                    <span style="margin-left: 30px">部位名称</span>
                    <el-input
                      v-model="bomData.boneName"
                      disabled
                      style="width: 276px !important"
                    />
                  </div>
                  <!-- 表格 -->
                  <div
                    style="
                      display: flex;
                      justify-content: space-between;
                      align-items: center;
                      margin: 10px 0;
                    "
                  >
                    <h3 style="margin: 0 0 10px 0">关联备件</h3>
                    <div
                      :class="check ? 'disabled-btn' : 'dia-btn'"
                      style="margin-right: 10px"
                      saveTag="true"
                      @click="saveSparePartsList"
                      class="eam-icon-text-btn"
                    >
                      <IconSave />
                      保存
                    </div>
                  </div>
                  <el-table border :data="spareTable" style="width: 100%">
                    <el-table-column type="index" label="序号" />
                    <el-table-column
                      label="备件分类"
                      prop="partsCategoryName"
                      align="left"
                      header-align="center"
                    />
                    <el-table-column
                      label="备件编号"
                      prop="partsCode"
                      align="left"
                      header-align="center"
                    />
                    <el-table-column
                      label="备件名称"
                      prop="partsName"
                      align="left"
                      header-align="center"
                    />
                    <el-table-column
                      label="规格型号"
                      prop="specification"
                      align="left"
                      header-align="center"
                    />
                    <el-table-column
                      label="数量"
                      prop="partsQty"
                      align="left"
                      header-align="center"
                    >
                      <template slot-scope="scope">
                        <el-input
                          v-model="scope.row.partsQty"
                          autosize
                          autofocus
                          :disabled="check"
                          @input="
                            scope.row.partsQty = Number(
                              scope.row.partsQty.replace(/^(0+)|[^\d]+/g, '')
                            )
                          "
                        />
                      </template>
                    </el-table-column>
                    <el-table-column
                      label="单位"
                      prop="unitCode"
                      align="left"
                      header-align="center"
                    />
                    <el-table-column
                      v-if="!check"
                      label="操作"
                      align="center"
                      width="100"
                    >
                      <template slot-scope="scope">
                        <div class="eam-table-btn-box">
                          <IconAdd @click="spareAdd(scope.row, scope.$index)" />
                          <IconDelete
                            @click="handleDelete(scope.row, scope.$index)"
                          />
                        </div>
                      </template>
                    </el-table-column>
                  </el-table>
                  <!-- 分页器 -->
                  <pagination
                    v-show="total > 0"
                    background
                    :total="total"
                    :page.sync="form.pageNum"
                    :limit.sync="form.pageSize"
                    @pagination="getList2"
                  />
                  <!-- 备件选择 -->
                  <el-dialog
                    :visible.sync="selectSpare"
                    append-to-body
                    title="备件选择"
                    width="40%"
                  >
                    <div class="dia-bg">
                      <div class="system-back">
                        <el-row>
                          <el-col :span="12">
                            <div
                              class="select-spare-box"
                              style="border-right: 2px solid #edeff0"
                            >
                              <div style="margin-bottom: 10px">
                                <el-input
                                  v-model.trim="filterText"
                                  placeholder="请输入分类名称"
                                />
                              </div>
                              <div>
                                <el-tree
                                  ref="tree"
                                  v-loading="selectSpareLoading"
                                  class="filter-tree"
                                  :data="selectSpareData"
                                  :props="defaultProps"
                                  :filter-node-method="filterNode"
                                  :default-expand-all="false"
                                  :expand-on-click-node="false"
                                  highlight-current
                                  @node-click="handlSparePartseNodeClick"
                                />
                              </div>
                            </div>
                          </el-col>
                          <el-col :span="12">
                            <div class="select-spare-box">
                              <div style="margin-bottom: 10px">
                                <el-input
                                  v-model.trim="filterText2"
                                  placeholder="请输入备件名称"
                                  @input="spareSearchInput"
                                />
                              </div>
                              <div class="select-spare-box-top">
                                <div>已选：{{ checkedSpareNum }}个备件</div>
                                <div style="color: #59acff" @click="clearSpare">
                                  清空
                                </div>
                              </div>
                              <el-checkbox-group
                                v-model="checkedSpare"
                                v-loading="checkedSpareLoading"
                                @change="handleCheckedSpareChange"
                              >
                                <el-checkbox
                                  v-for="item in spareList"
                                  :key="item.id"
                                  :label="item.id"
                                  :disabled="item.disabled"
                                  >{{
                                    item.partsName + "（" + item.id + "）"
                                  }}</el-checkbox
                                >
                              </el-checkbox-group>
                            </div>
                          </el-col>
                        </el-row>
                      </div>
                    </div>

                    <div slot="footer">
                      <el-button @click="selectCancle">取 消</el-button>
                      <el-button type="primary" @click="selectSpareSubmit"
                        >保 存</el-button
                      >
                    </div>
                  </el-dialog>
                </div>
              </div>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>

    <!-- 备件选择对话框 -->
    <SpareBomDialog
      ref="spareBomDialogRef"
      :check="check"
      :infoData="infoData"
      @save="handleSpareSave"
      :max="100"
    />
  </div>
</template>

<script>
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import Treeselect from "@riophae/vue-treeselect";
import { getFactory } from "@api/eam/manage/emMstLocation";
import {
  ListAssetBomTree,
  addAssetBomPosition,
  assetBomInfoEdit,
  assetBomDelete,
  assetBomInfo,
  getTreeCancelId,
  getAssetPartsByBomId,
  getAssetPartsByBomIdDelete,
  addAssetPartsList,
} from "@api/eam/assets/AssetBom";
import { parePartsBasicGetList2 } from "@api/eam/spareParts/sparePartsBasic";
import { sparePartsGetTree } from "@api/eam/spareParts/sparePartsType";

import SpareBomDialog from "./components/SpareBomDialog.vue";

export default {
  name: "DeviceCategory",
  components: {
    Treeselect,
    SpareBomDialog,
  },
  props: {
    // 基础信息数据
    infoData: {
      type: Object,
      default: () => {},
    },
    // 当前页面为查看还是编辑
    check: {
      type: Boolean,
      default: true,
    },
    deviceNum: {
      type: String,
      default: "",
    },
    devicename: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      // bom数据
      assetData: {},
      // 部位详情
      bomData: {
        id: "",
        parentId: "",
        boneCode: "",
        boneName: "",
      },
      bomLoading: true,
      // 树形分类列表
      dataTree: [],
      // 树形结构关键字
      defaultProps: {
        children: "children",
        label: "label",
      },
      // 部位树
      partTree: [],
      // 是否点击了分类
      clickType: false,
      treeVisible: false,
      treeTitle: "",
      filterText: "",
      // 当前弹窗为新增还是
      treeDia: "",
      // 表单下拉选项
      factoryList: [],
      locationTypeList: [],
      // 表格数据
      tableData: [],
      userVisible: false,
      // 存储 当前设备信息
      assetInfo: { factoryId: "", assetId: "" },
      // 提交表单查询用户组 用户详情
      formTree: {
        factoryId: "",
        assetId: "",
        parentId: "",
        boneCode: "",
        boneName: "",
        description: "",
        seq: 1,
      },
      // 表格请求参数
      form: {
        pageNum: 1,
        pageSize: 10,
        assetBomId: "",
      },
      spareList: [],
      // 控制备件弹窗展示
      selectSpare: false,
      // 所有的备件信息列表
      allSpareList: [],
      checkedSpare: [],
      checkedSpareNum: 0,
      // 备件弹窗树形结构
      selectSpareData: [],
      // 添加表格时默认数据
      defaultData: {
        assetBomId: "",
        partsId: "",
        applyStatus: "",
        partsCategoryName: "",
        createTime: "",
        partsCode: "",
        partsName: "",
        specification: "",
        partsPlanQty: "",
        unitPrice: "",
        cost: "",
      },
      // 当前表格备件数据
      spareTable: [],
      // 备件名称搜索
      filterText2: "",
      // 备件选择备件列表loading
      checkedSpareLoading: false,
      // 原始备件数据
      originSpareList: [],
      // 备件选择分类loading
      selectSpareLoading: false,
      // 备件表格loading
      spareTableLoading: false,
      // 查询用户列表参数
      formChoose: {
        pageNum: 1,
        pageSize: 10,
        userCode: undefined,
        userName: undefined,
        nickName: undefined,
        phone: undefined,
        status: 0,
        deptId: undefined,
        postId: undefined,
        dateRange: undefined,
      },
      userTotal: 0,
      // 用户列表
      userList: [],
      total: 0,
      loading: true,
      // 表单验证规则
      chooseRules: {
        factoryId: [{ required: true, message: "请选择工厂", trigger: "blur" }],
        boneCode: [
          { required: true, message: "请填写部位名称", trigger: "blur" },
        ],
        boneName: [
          { required: true, message: "填写部位名称", trigger: "blur" },
        ],
      },
    };
  },
  watch: {
    filterText(val) {
      this.$refs.tree.filter(val);
    },
  },
  created() {
    this.assetData = this.infoData;
    this.assetInfo.factoryId = this.assetData.factoryId;
    this.assetInfo.assetId = this.assetData.id;
    // this.getFactory()
    this.getAllSpareList();
    this.getTree();
  },

  methods: {
    // 获取工厂列表
    getFactory() {
      getFactory().then((res) => {
        this.factoryList = res.data;
      });
    },
    // 点击 设备部位 树  获取数据
    handleNodeClick(data, node) {
      this.clickType = true;
      // 添加部位使用此作为设备部位的parentId
      this.formTree.addParentId = data.id;
      this.formTree.parentId = data.parentId;
      const Obj = {
        ...this.formTree,
        boneName: data.label,
        assetBomId: data.id,
        parentId: data.parentId,
      };
      this.formTree = Obj;
      // TODO
      this.form.assetBomId = data.id;
      this.getList2();
      // // 点击如果为设备本身 则展示设备下第一条部位的详情
      // if (data['children'] !== undefined) {
      //   //   if (data.parentId === '0') {
      //   //     this.form.assetBomId = data.children[0].id
      //   //     this.getList2()
      //   //   } else {
      //   //     this.form.assetBomId = data.id
      //   //     this.getList2()
      //   //   }
      //   this.form.assetBomId = data.id
      //   this.getList2()
      //   // } else if (data['children'] === undefined && data.parentId === '0') {
      //   //   this.spareTable = []
      //   //   this.bomData = {}
      //   // } else {
      // }
    },
    // 获取树形结构
    getPartTreeSelect(assetBomId) {
      getTreeCancelId({
        assetId: this.assetData.id,
        assetBomId,
      }).then((res) => {
        this.partTree = res.data;
      });
    },
    // 新增设备部位
    addBomPart() {
      if (this.check) return;
      this.formTree.parentId = this.formTree.addParentId;
      if (this.clickType === false && this.formTree.parentId === undefined) {
        this.$message.error("请点击一个部位后再尝试吧");
        return false;
      }
      this.treeTitle = "新增设备部位";
      this.treeDia = "add";
      this.formTree.boneName = "";
      this.formTree.boneCode = "";
      this.treeVisible = true;
    },
    // 获取设备部位树列表
    getTree() {
      this.bomLoading = true;
      ListAssetBomTree({ assetId: this.assetData.id }).then((res) => {
        this.dataTree = res.data;
        this.bomLoading = false;
        // 此设备有部位则获取部位详情数据
        if (
          res.data[0] !== undefined &&
          res.data[0]["children"] !== undefined
        ) {
          this.form.assetBomId = res.data[0].children[0].id;
          this.getList2();
        } else if (
          res.data[0] !== undefined &&
          res.data[0]["children"] === undefined
        ) {
          this.spareTable = [];
          this.bomData = {};
        } else {
          this.getList2();
        }
        this.loading = false;
      });
    },
    // 修改 设备部位  树
    edit(data) {
      if (this.check) return;
      this.treeTitle = "修改设备部位";
      this.treeDia = "edit";

      const Obj = {
        ...this.formTree,
        ...data,
        boneName: data.label,
      };
      // 获取设备部位详情
      assetBomInfo({ ...this.form }).then((res) => {
        Obj.boneCode = res.data.boneCode;
        this.formTree = Obj;
        this.treeVisible = true;
      });
      this.getPartTreeSelect(data.id);
    },
    // 删除 指定设备部位
    remove() {
      if (this.check) return;
      const _this = this;
      if (_this.clickType === false) {
        _this.$message.error("请点击所需要删除的部位后再尝试吧");
        return;
      }
      if (_this.formTree.parentId === "0") {
        _this.$message.error("顶级设备不可删除，请不要这样操作");
        return;
      }
      _this.$modal
        .confirm(`是否确认删除部位 ${_this.formTree.boneName} 吗？`)
        .then(
          () => {
            assetBomDelete({ assetBomId: _this.formTree.assetBomId }).then(
              (res) => {
                if (res.code === 200) {
                  _this.clickType = false;
                  _this.formTree.addParentId = undefined;
                  _this.getTree();
                  _this.$modal.msgSuccess("删除成功");
                }
              }
            );
          },
          () => {}
        );
    },
    // 添加修改树形 分组表单重置
    formTreeReset() {
      this.formTree = {
        parentId: "",
        boneCode: undefined,
        boneName: undefined,
        description: undefined,
        seq: 1,
      };
      this.clickType = false;
    },
    // 操作用户归属弹窗 关闭回调
    handleClose() {
      this.$refs["formChoose"].resetFields();
      this.treeVisible = false;
      this.formTreeReset();
    },
    // 操作用户归属弹窗 确定按钮
    handlecConfirm() {
      const _this = this;
      this.$refs["formChoose"].validate((valid) => {
        if (valid) {
          const BomInfo = {
            ..._this.formTree,
            ..._this.assetInfo,
          };
          if (_this.treeDia === "add") {
            addAssetBomPosition(BomInfo).then((res) => {
              _this.treeVisible = false;
              if (res.code === 200) {
                _this.msgSuccess("添加成功");
                _this.clickType = false;
                _this.getTree();
              }
            });
          } else if (_this.treeDia === "edit") {
            assetBomInfoEdit(BomInfo).then((res) => {
              _this.treeVisible = false;
              if (res.code === 200) {
                _this.msgSuccess("修改成功");
                _this.clickType = false;
                _this.getTree();
              }
            });
          }
          _this.formTreeReset();
        } else {
          return false;
        }
      });
    },
    // 筛选 数据
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    // 获取 部位详情列表
    getList2() {
      this.loading = true;
      // 获取设备部位详情
      assetBomInfo({ ...this.form }).then((res) => {
        this.formTree.boneCode = res.data.boneCode;
        this.bomData = res.data;
        this.loading = false;
      });
      // 获取设备部位关联备件列表
      this.getAssetPartsByBomIdList();
    },
    //  获取设备部位关联备件列表
    getAssetPartsByBomIdList() {
      getAssetPartsByBomId({ ...this.form }).then((res) => {
        if (res.data.length !== 0) {
          this.spareTable = res.data;
          this.total = res.total;
          this.loading = false;
        } else {
          !this.check
            ? (this.spareTable = [{ ...this.defaultData }])
            : (this.spareTable = []);

          this.loading = false;
        }
      });
    },
    // 转换位置信息数据结构
    normalizer(node) {
      if (node.children && !node.children.length) {
        delete node.children;
      }
      return {
        id: node.id,
        label: node.label,
        children: node.children,
      };
    },
    // 保存添加的备件
    saveSparePartsList() {
      if (this.check) return;
      if (this.spareTable[0].assetBomId === "") {
        this.msgWarning("请添加备件后再点击保存按钮");
        return false;
      }
      addAssetPartsList({
        factoryId: this.assetInfo.factoryId,
        assetId: this.assetInfo.assetId,
        assetBomId: this.bomData.id,
        partsList: this.spareTable,
      }).then((res) => {
        res.code === 200 && this.msgSuccess("添加备件成功");
        this.getAssetPartsByBomIdList();
      });
    },
    // 备件表格新增
    spareAdd(row, $index) {
      if (this.check) return;
      let ss = this.infoData;
      row.emMaintenanceStandardPartsLogList = this.spareTable?.map((d) => {
        return {
          factoryId: this.infoData.factoryId,
          assetId: this.infoData.id,
          partsId: d.id,
          // applyStatus: 'Add',
          partsCategoryName: d.partsCategoryName,
          partsCode: d.partsCode,
          partsName: d.partsName,
          specification: d.specification,
          unitPrice: d.price,
          partsQty: 1,
          cost: d.price,
          //
          unitCode: d.unitCode,
          price: d.price,
        };
      });

      this.$refs.spareBomDialogRef.edit(row, $index);
      // this.currentTableIndex = $index

      // this.spareList = []
      // sparePartsGetTree().then((res) => {
      //   this.spareTableLoading = false
      //   this.selectSpareData = res.data
      //   this.spareShow = true
      //   this.selectSpare = true
      // })
    },

    // 表格按钮删除
    handleDelete(row, $index) {
      if (this.check) return;
      const _this = this;
      if (row.id) {
        this.$modal
          .confirm(`是否将 ${row.partsName}  从关联备件中删除吗？`)
          .then(() => {
            return getAssetPartsByBomIdDelete({
              assetPartsId: row.id,
              assetBomId: this.bomData.id,
            });
          })
          .then((res) => {
            if (res.code === 200) {
              // 处理element分页错误
              const totalPage = Math.ceil(
                (_this.total - 1) / _this.form.pageSize
              );
              const pageNum =
                _this.form.pageNum > totalPage ? totalPage : _this.form.pageNum;
              _this.form.pageNum = pageNum < 1 ? 1 : pageNum;

              _this.dataTree.children &&
                (_this.form.assetBomId = _this.dataTree.children[0].assetBomId);
              _this.getList2();
              _this.$modal.msgSuccess("删除成功");
              if (!this.spareTable.length) {
                this.spareTable = [{ ...this.defaultData }];
              }
            }
          });
      } else {
        this.spareTable = this.spareTable.filter((item, index) => {
          return $index !== index;
        });
        this.spareTable.length === 0 &&
          (this.spareTable = [{ ...this.defaultData }]);
      }
    },
    // 备件名称搜索
    spareSearchInput() {
      this.spareList = this.originSpareList.filter((item) =>
        item.partsName.includes(this.filterText2)
      );
    },
    // 清空所选备件
    clearSpare() {
      this.checkedSpare = [];
      this.checkedSpareNum = 0;
    },
    // 备件信息选择
    handleCheckedSpareChange() {
      this.checkedSpareNum = this.checkedSpare.length;
    },
    // 获取所有备件列表
    getAllSpareList() {
      parePartsBasicGetList2().then((res) => {
        this.allSpareList = res.data;
      });
    },
    // 备件选择分类点击
    handlSparePartseNodeClick(data) {
      const query = {
        partsCategoryId: data.id,
      };
      this.checkedSpareLoading = true;
      parePartsBasicGetList2(query).then((res) => {
        this.checkedSpareLoading = false;
        this.originSpareList = res.data;
        this.spareList = res.data?.slice(0, 300);

        // this.spareList = res.data
        this.spareList.forEach((item) => {
          if (this.spareTable[0].id !== "") {
            this.spareTable.map((spareParts) => {
              spareParts.id === item.id && (item["disabled"] = true);
            });
          }
          // item['disabled'] = false
        });
        this.originSpareList.forEach((item) => {
          if (this.spareTable[0].id !== "") {
            this.spareTable.map((spareParts) => {
              spareParts.id === item.id && (item["disabled"] = true);
            });
          }
          // item['disabled'] = false
        });
        const choseList = [];
        this.spareTable.forEach((item) => {
          if (item.applyStatus === "Add") {
            choseList.push(item.partsId);
          }
        });
        this.originSpareList.forEach((item) => {
          choseList.forEach((item2) => {
            if (item.id === item2) {
              item.disabled = true;
            }
          });
        });
      });
    },
    // 备件选择保存
    selectSpareSubmit() {
      const newTable = [];
      this.allSpareList.forEach((item) => {
        this.checkedSpare.forEach((item2) => {
          if (item.id === item2) {
            newTable.push({
              assetBomId: this.bomData.id,
              factoryId: this.assetInfo.factoryId,
              assetId: this.formTree.assetId,
              partsId: item.id,
              applyStatus: "Add",
              partsCategoryName: item.partsCategoryName,
              partsCode: item.partsCode,
              partsQty: item.partsQty,
              unitCode: item.unitCode,
              partsName: item.partsName,
              specification: item.specification,
              unitPrice: item.price,
              partsPlanQty: 1,
              cost: item.price,
            });
          }
        });
      });

      if (this.spareTable[0].partsCode !== "") {
        this.spareTable = this.spareTable.concat(newTable);
      } else {
        this.spareTable = newTable;
      }
      console.log(newTable);
      this.filterText = "";
      this.filterText2 = "";
      this.checkedSpare = [];
      this.checkedSpareNum = 0;
      this.selectSpare = false;
    },
    // 备件选择取消
    selectCancle() {
      this.filterText = "";
      this.filterText2 = "";
      this.checkedSpareNum = 0;
      this.originSpareList = [];
      this.spareList = [];
      this.checkedSpare = [];
      this.selectSpare = false;
    },
    // 工厂翻译
    // factoryIdFormat(row) {
    //   if (row.factoryId && this.factoryList.length) {
    //     const obj = this.factoryList.find((item) => {
    //       return item.id === row.factoryId
    //     })

    //     return obj.factoryName
    //   }
    // }
    // 备件选择 保存
    handleSpareSave(spareList) {
      // 清空站位行
      if (this.spareTable[0].partsCode === "") {
        this.spareTable = [];
      }

      spareList?.forEach((d) => {
        let exist = this.spareTable?.find((sd) => sd.id === d.partsId);
        if (!exist && d.partsCode) {
          let temp = {
            assetBomId: this.bomData.id,
            // assetBomId: d.assetBomId,
            factoryId: d.factoryId,
            // assetId: d.assetId,
            assetId: "",
            partsId: d.partsId,
            // id: d.partsId, //
            applyStatus: d.applyStatus,
            partsCategoryName: d.partsCategoryName,
            partsCode: d.partsCode,
            partsQty: d.partsQty,
            unitCode: d.unitCode,
            partsName: d.partsName,
            specification: d.specification,
            unitPrice: d.unitPrice,
            partsPlanQty: d.partsPlanQty,
            cost: d.cost,
          };
          this.spareTable.push(temp);
          console.log(
            "🚀 ~ file: bom.vue:942 ~ spareList?.forEach ~ temp:",
            temp
          );
        }
      });
    },
  },
};
</script>

<style scoped lang="scss">
.dia-btn2 {
  display: flex;
  justify-content: center;
  .dia-btns {
    width: 18px;
    height: 18px;
  }
}

::v-deep .user-table .el-dialog__body {
  padding-top: 0 !important;
  padding-bottom: 2px !important;
}
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}

.grouping-title {
  margin: 10px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  div > span {
    margin-right: 20px;
    font-size: 20px;
    i {
      font-weight: 800;
    }
  }
}
// row组件的样式
.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
::v-deep .el-col {
  border-radius: 4px;
  padding: 0;
}

.dia-btn {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #007fff;
  border-radius: 5px;
  color: #ffffff;
  padding: 8px 18px 10px 20px;
  img {
    width: 20px;
    height: 20px;
    margin-right: 4px;
  }
}
.dia-btn:hover {
  cursor: pointer;
}
.bg-purple {
  margin: 14px;
  background: #f0f2f5;
  margin-top: 0;
}

.grid-content {
  padding: 14px;
  border-radius: 4px;
  min-height: 36px;
  background-color: #fff;
}
.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}
.bom-back {
  padding: 14px;
  background-color: #f0f2f5;
}
.classify-sty {
  border-radius: 4px;
  background-color: #fff;
  box-shadow: 0px 4px 10px 0px rgba(163, 164, 179, 0.04);
  caret-color: transparent;
}
.classify-left {
  background-color: #fff;
  .classify-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 0 22px 0;
    > h2 {
      margin: 0;
      padding: 0;
      font-size: 20px;
      font-weight: 400;
      height: 24px;
    }
    > div {
      i {
        margin-right: 19px;
        border-radius: 50%;
        color: #fff;
        background-color: #a8a8a8;
        text-align: center;
      }
      i:hover {
        cursor: pointer;
      }
    }
  }
}

.dia-bg {
  background-color: #f5f7fa;
  padding: 14px;
}
.select-spare-box {
  padding: 20px 14px;
  .select-spare-box-top {
    display: flex;
    justify-content: space-between;
    cursor: pointer;
    margin-bottom: 10px;
  }
}
.classify-right {
  padding: 14px;
  background-color: #f0f2f5;
  .table-box {
    margin: 0 0 14px 0;
    border-radius: 4px;
    .table-content {
      border-radius: 4px;
      background-color: #fff;
      padding: 24px;
      .el-form {
        .el-input {
          width: 280px;
        }
        .el-select {
          width: 280px;
        }
      }
      .block {
        padding: 40px 10px 0 10px;
      }
      .btn-color {
        color: #a8a8a8;
      }
    }
  }
  .table-box:last-child {
    margin: 0;
  }
}
</style>
