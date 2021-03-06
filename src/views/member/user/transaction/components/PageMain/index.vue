<template>
  <div class="cs-p">
    <el-table
      :data="tableData"
      :highlight-current-row="true"
      @sort-change="sortChange">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="table-expand">
            <el-form-item label="交易编号">
              <span>{{props.row.transaction_id}}</span>
            </el-form-item>

            <el-form-item label="来源订单号">
              <span
                @click="handleOrder(props.row.source_no)"
                class="link">{{props.row.source_no}}</span>
            </el-form-item>

            <el-form-item label="交易账号">
              <span>{{props.row.get_user.username}}</span>
            </el-form-item>

            <el-form-item label="交易昵称">
              <span>{{props.row.get_user.nickname}}</span>
            </el-form-item>

            <el-form-item label="收支类型">
              <el-tag
                :type="typeMap[props.row.type].type"
                effect="plain"
                size="mini">
                {{typeMap[props.row.type].text}}
              </el-tag>
            </el-form-item>

            <el-form-item label="收支金额">
              <el-tag
                :type="typeMap[props.row.type].type"
                effect="plain"
                size="mini">
                {{typeMap[props.row.type].symbol}} {{props.row.amount | getNumber}}
              </el-tag>
            </el-form-item>

            <el-form-item label="剩余余额">
              <span>{{props.row.balance | getNumber}}</span>
            </el-form-item>

            <el-form-item label="交易来源">
              <span>{{getToPayment(props.row.to_payment)}}</span>
            </el-form-item>

            <el-form-item label="收支模型">
              <span>{{moduleMap[props.row.module]}}</span>
            </el-form-item>

            <el-form-item label="购物卡卡号">
              <span>{{props.row.card_number}}</span>
            </el-form-item>

            <el-form-item label="交易备注">
              <span>{{props.row.remark}}</span>
            </el-form-item>

            <el-form-item label="操作原因">
              <span>{{props.row.cause}}</span>
            </el-form-item>

            <el-form-item label="操作人">
              <span>{{props.row.action}}</span>
            </el-form-item>

            <el-form-item label="创建日期">
              <span>{{props.row.create_time}}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>

      <el-table-column
        label="来源订单号"
        prop="source_no"
        width="240">
        <template slot-scope="scope">
          <span
            @click="handleOrder(scope.row.source_no)"
            class="link">{{scope.row.source_no}}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="交易账号"
        prop="get_user.username">
      </el-table-column>

      <el-table-column
        label="收支金额"
        prop="type"
        sortable="custom"
        min-width="100">
        <template slot-scope="scope">
          <el-tag
            :type="typeMap[scope.row.type].type"
            effect="plain"
            size="mini">
            {{typeMap[scope.row.type].symbol}} {{scope.row.amount | getNumber}}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column
        label="剩余余额"
        prop="balance">
        <template slot-scope="scope">
          {{scope.row.balance | getNumber}}
        </template>
      </el-table-column>

      <el-table-column
        label="交易来源"
        prop="to_payment">
        <template slot-scope="scope">
          {{getToPayment(scope.row.to_payment)}}
        </template>
      </el-table-column>

      <el-table-column
        label="收支模型"
        prop="module">
        <template slot-scope="scope">
          {{moduleMap[scope.row.module]}}
        </template>
      </el-table-column>

      <el-table-column
        label="交易备注"
        prop="remark"
        :show-overflow-tooltip="true">
      </el-table-column>

      <el-table-column
        label="操作人"
        prop="action">
      </el-table-column>

      <el-table-column
        label="创建日期"
        prop="create_time"
        align="center"
        width="160"
        sortable="custom">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import util from '@/utils/util'

export default {
  props: {
    toPayment: {
      default: () => {}
    },
    tableData: {
      default: () => []
    }
  },
  data() {
    return {
      typeMap: {
        0: {
          text: '收入',
          type: 'success',
          symbol: '+'
        },
        1: {
          text: '支出',
          type: 'danger',
          symbol: '-'
        }
      },
      moduleMap: {
        points: '积分',
        money: '余额',
        card: '购物卡'
      }
    }
  },
  filters: {
    getNumber(val) {
      return util.getNumber(val)
    }
  },
  methods: {
    getToPayment(val) {
      return val !== '' ? this.toPayment[val]['name'] : ''
    },
    sortChange({ column, prop, order }) {
      let sort = {
        order_type: undefined,
        order_field: undefined
      }

      if (column && order) {
        sort.order_type = order === 'ascending' ? 'asc' : 'desc'
        sort.order_field = prop
      }

      this.$emit('sort', sort)
    },
    handleOrder(order_no) {
      this.$router.push({
        name: 'order-admin-info',
        params: { order_no }
      })
    }
  }
}
</script>

<style scoped>
  .link:hover {
    cursor: pointer;
    color: #409EFF;
    text-decoration: underline;
  }

  .table-expand {
    font-size: 0;
    padding: 0 10px;
  }

  .table-expand >>> label {
    float: left;
    width: 90px;
    color: #99A9BF;
  }

  .table-expand >>> .el-form-item__content {
    display: block;
    padding-left: 90px;
  }

  .table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>
