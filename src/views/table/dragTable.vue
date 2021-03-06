<template>
  <div class="app-container">
    <!-- Note that row-key is necessary to get a correct row order. -->
    <el-table v-loading="listLoading" :data="list" row-key="id" border fit highlight-current-row style="width: 100%">

      <el-table-column align="center" label="ID" width="65">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column width="180px" align="center" label="Date">
        <template slot-scope="scope">
          <span>{{ scope.row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>

      <el-table-column min-width="300px" label="Title">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>

      <el-table-column width="110px" align="center" label="Author">
        <template slot-scope="scope">
          <span>{{ scope.row.author }}</span>
        </template>
      </el-table-column>

      <el-table-column width="100px" label="Importance">
        <template slot-scope="scope">
          <svg-icon v-for="n in +scope.row.importance" :key="n" icon-class="star" class="icon-star"/>
        </template>
      </el-table-column>

      <el-table-column align="center" label="Readings" width="95">
        <template slot-scope="scope">
          <span>{{ scope.row.pageviews }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="Status" width="110">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status | articleStatusFilter">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>

      <el-table-column align="center" label="Drag" width="80">
        <template slot-scope="{}">
          <svg-icon class="drag-handler" icon-class="drag"/>
        </template>
      </el-table-column>

    </el-table>
    <!-- $t is vue-i18n global function to translate lang (lang in @/lang)  -->
    <div class="show-d">{{ $t('table.dragTips1') }} : &nbsp; {{ oldList }}</div>
    <div class="show-d">{{ $t('table.dragTips2') }} : {{ newList }}</div>

  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { IListQuery } from '@/interface';
import Sortable from 'sortablejs';
import {SUCCESS_STATUS} from '@/constant';

@Component({
  name: 'DragTable'
})
export default class DragTable extends Vue {
  list: any[] = [];
  total: number = 0;
  listLoading: boolean = true;
  listQuery: IListQuery = {
    page: 1,
    limit: 10
  };
  sortable: any = null;
  oldList: any[] = [];
  newList: any[] = [];

  created() {
    this.getList();
  }

  getList() {
    this.listLoading = true;
    this.$services.articleList({method: 'get', data: this.listQuery}).then((response) => {
      const {code, data} = response;
      if (code === SUCCESS_STATUS) {
        this.list = data.items;
        this.total = data.total;
      } else {
        this.list = [];
        this.total = 0;
      }
      this.listLoading = false;
      this.oldList = this.list.map((v) => v.id);
      this.newList = this.oldList.slice();
      this.$nextTick(() => {
        this.setSort();
      });
    });
  }

  setSort() {
    const el = document.querySelectorAll('.el-table__body-wrapper > table > tbody')[0] as HTMLElement;
    this.sortable = Sortable.create((el), {
      ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
      setData(dataTransfer) {
        dataTransfer.setData('Text', '');
        // to avoid Firefox bug
        // Detail see : https://github.com/RubaXa/Sortable/issues/1012
      },
      onEnd: (evt: any) => {
        const targetRow = this.list.splice(evt.oldIndex, 1)[0];
        this.list.splice(evt.newIndex, 0, targetRow);

        // for show the changes, you can delete in you code
        const tempIndex = this.newList.splice(evt.oldIndex, 1)[0];
        this.newList.splice(evt.newIndex, 0, tempIndex);
      }
    });
  }
}
</script>

<style>
  .sortable-ghost {
    opacity: .8;
    color: #fff !important;
    background: #42b983 !important;
  }
</style>

<style scoped>
  .icon-star {
    margin-right: 2px;
  }

  .drag-handler {
    width: 20px;
    height: 20px;
    cursor: pointer;
  }

  .show-d {
    margin-top: 15px;
  }
</style>
