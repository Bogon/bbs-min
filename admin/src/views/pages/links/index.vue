<template>
  <section class="page-container">
    <div ref="toolbar" class="toolbar">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.title" placeholder="标题" />
        </el-form-item>
        <el-form-item>
          <el-input v-model="filters.url" placeholder="链接" />
        </el-form-item>
        <el-form-item>
          <el-select v-model="filters.status" clearable placeholder="请选择状态" @change="list">
            <el-option label="正常" value="0" />
            <el-option label="删除" value="1" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="list"> 查询 </el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd"> 新增 </el-button>
        </el-form-item>
      </el-form>
    </div>

    <div ref="mainContent" :style="{ height: mainHeight }">
      <el-table
        v-loading="listLoading"
        height="100%"
        :data="results"
        highlight-current-row
        stripe
        border
      >
        <el-table-column prop="id" label="编号" width="100" />
        <el-table-column prop="title" label="标题" />
        <el-table-column prop="url" label="链接">
          <template slot-scope="scope">
            <a :href="scope.row.url" target="_blank">{{ scope.row.url }}</a>
          </template>
        </el-table-column>
        <el-table-column prop="logo" label="Logo" width="60">
          <template slot-scope="scope">
            <el-image
              v-if="scope.row.logo"
              :src="scope.row.logo"
              class="link-logo"
              fit="cover"
              :preview-src-list="[scope.row.logo]"
            />
          </template>
        </el-table-column>
        <el-table-column prop="summary" label="描述" />
        <el-table-column prop="status" label="状态" width="50">
          <template slot-scope="scope">
            {{ scope.row.status === 0 ? "正常" : "删除" }}
          </template>
        </el-table-column>
        <el-table-column prop="createTime" label="创建时间">
          <template slot-scope="scope">
            {{ scope.row.createTime | formatDate }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template slot-scope="scope">
            <el-button size="small" @click="handleEdit(scope.$index, scope.row)"> 编辑 </el-button>
          </template>
        </el-table-column>
        <template #empty>
          <el-empty />
        </template>
      </el-table>
    </div>

    <div ref="pagebar" class="pagebar">
      <el-pagination
        :page-sizes="[20, 50, 100, 300]"
        :current-page="page.page"
        :page-size="page.limit"
        :total="page.total"
        layout="total, sizes, prev, pager, next, jumper"
        @current-change="handlePageChange"
        @size-change="handleLimitChange"
      />
    </div>

    <el-dialog :visible.sync="addFormVisible" :close-on-click-modal="false" title="新增">
      <el-form ref="addForm" :model="addForm" label-width="80px">
        <el-form-item label="标题">
          <el-input v-model="addForm.title" />
        </el-form-item>
        <el-form-item label="链接">
          <el-input v-model="addForm.url" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="addForm.summary" :autosize="{ minRows: 2, maxRows: 4 }" />
        </el-form-item>
        <el-form-item label="Logo">
          <upload v-model="addForm.logo" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="addFormVisible = false"> 取消 </el-button>
        <el-button :loading="addLoading" type="primary" @click.native="addSubmit"> 提交 </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="editFormVisible" :close-on-click-modal="false" title="编辑">
      <el-form ref="editForm" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-input v-model="editForm.id" type="hidden" />
        <el-form-item label="标题">
          <el-input v-model="editForm.title" />
        </el-form-item>
        <el-form-item label="链接">
          <el-input v-model="editForm.url" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="editForm.summary" :autosize="{ minRows: 2, maxRows: 4 }" />
        </el-form-item>
        <el-form-item label="Logo">
          <upload v-model="editForm.logo" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="editForm.status" placeholder="请选择">
            <el-option :key="0" :value="0" label="正常" />
            <el-option :key="1" :value="1" label="删除" />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false"> 取消 </el-button>
        <el-button :loading="editLoading" type="primary" @click.native="editSubmit">
          提交
        </el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
import Upload from "@/components/Upload";
import mainHeight from "@/utils/mainHeight";

export default {
  name: "Links",
  components: { Upload },
  data() {
    return {
      mainHeight: "300px",
      results: [],
      listLoading: false,
      page: {},
      filters: {},

      addForm: {},
      addFormVisible: false,
      addFormRules: {},
      addLoading: false,

      editForm: {},
      editFormVisible: false,
      editFormRules: {},
      editLoading: false,
    };
  },
  mounted() {
    mainHeight(this);
    this.list();
  },
  methods: {
    list() {
      const me = this;
      me.listLoading = true;
      const params = Object.assign(me.filters, {
        page: me.page.page,
        limit: me.page.limit,
      });
      this.axios
        .form("/api/admin/link/list", params)
        .then((data) => {
          me.results = data.results;
          me.page = data.page;
        })
        .finally(() => {
          me.listLoading = false;
        });
    },
    handlePageChange(val) {
      this.page.page = val;
      this.list();
    },
    handleLimitChange(val) {
      this.page.limit = val;
      this.list();
    },
    handleAdd() {
      this.addForm = {};
      this.addFormVisible = true;
    },
    addSubmit() {
      const me = this;
      this.axios
        .form("/api/admin/link/create", this.addForm)
        .then((data) => {
          me.$message({ message: "提交成功", type: "success" });
          me.addFormVisible = false;
          me.list();
        })
        .catch((rsp) => {
          me.$notify.error({ title: "错误", message: rsp.message });
        });
    },
    handleEdit(index, row) {
      const me = this;
      this.axios
        .get(`/api/admin/link/${row.id}`)
        .then((data) => {
          me.editForm = Object.assign({}, data);
          me.editFormVisible = true;
        })
        .catch((rsp) => {
          me.$notify.error({ title: "错误", message: rsp.message });
        });
    },
    editSubmit() {
      const me = this;
      this.axios
        .form("/api/admin/link/update", me.editForm)
        .then((data) => {
          me.list();
          me.editFormVisible = false;
        })
        .catch((rsp) => {
          me.$notify.error({ title: "错误", message: rsp.message });
        });
    },
  },
};
</script>

<style scoped>
.link-logo {
  max-width: 50px;
  max-height: 50px;
}
</style>
