   <template>
    <el-dialog
      title=""
      v-if="dialogVisible"
      :visible.sync="dialogVisible"
      width="35%"
      :before-close="onCloseBtn"
      :close-on-click-modal="false"
      destroyOnClose
      @close="onCloseBtn"
      >

     <el-form ref="form" :model="form" label-width="80px" :rules="rules">
      <el-form-item label="活動標題" prop="title">
        <el-input v-model="form.title"></el-input>
      </el-form-item>
      <el-form-item label="全天" prop="allDay">
        <el-switch v-model="form.allDay"></el-switch>
      </el-form-item>
      <el-form-item label="活動時間" prop="dateRange">
        <el-row type="flex">
          <el-col>
            <el-form-item prop="dateRange">
            <el-date-picker
              type="dates"
              v-model="form.dateRange"
              placeholder="選擇一個或多個日期">
            </el-date-picker>
            </el-form-item>
          </el-col>
          <el-col :offset="2" v-if="!form.allDay">
            <el-form-item prop="dateTime" >
              <el-time-picker placeholder="選擇時間" v-model="form.dateTime" style="width: 100%;"  arrow-control></el-time-picker>
              </el-form-item>
          </el-col>
          
        </el-row>
      </el-form-item>
      <!-- <el-form-item label="備註" prop="remark">
        <el-input type="textarea" v-model="form.remark"></el-input>
      </el-form-item> -->
    </el-form>
      <span slot="footer" class="dialog-footer">
        <el-row type="flex">
          <el-col>
          <el-button @click="onCloseBtn">取 消</el-button>
            <el-button @click="onDeleteBtn" v-if="dialogStatus === 'edit'">刪除</el-button>
            <el-button type="primary" @click="onAddBtn">
              <span v-if="dialogStatus === 'add'">新增</span>
              <span v-else>保存</span>
            </el-button>

          </el-col>
        </el-row>
      </span>
    </el-dialog>
</template>

<script>
import isString from 'lodash/isString'

export default {
    data() {
      return {
        rules: {
          title: [
            { required: true, message: '請輸入活動標題', trigger: 'blur' },
          ],
        }
    }},
    props: {
        form: {
            type: Object,
            required: true
        },
        dialogStatus: {
            type: [String , Boolean],
            required: true
        },
        selectInfo: {
            type: Object,
        }
    },
    computed: {
      dialogVisible(){
        return isString(this.dialogStatus)
      }
    },
    methods: {
      onCloseBtn(){
        this.$emit('handle-close')
      },
      onAddBtn(){
        this.$refs['form'].validate((valid) => {
          if (valid) {           
            this.$emit('handle-add-submit')
          }
        })
      },
      onDeleteBtn(){
        this.$emit('handle-delete')
      }
    }
}

</script>

<style>

</style>