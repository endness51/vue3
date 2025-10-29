  <template>

 <div>123</div>
</template>

<script setup lang="ts">
import download from '@/utils/download'
import { ResumeApi, Resume } from '@/api/resume'


/** 简历主 列表 */
defineOptions({ name: 'Resume' })

const message = useMessage() // 消息弹窗
const { t } = useI18n() // 国际化

const loading = ref(true) // 列表的加载中
const list = ref<Resume[]>([]) // 列表的数据
const total = ref(0) // 列表的总页数
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  userId: undefined,
  title: undefined,
  name: undefined,
  gender: undefined,
  birthdate: [],
  phone: undefined,
  email: undefined,
  avatar: undefined,
  currentResidence: undefined,
  education: undefined,
  workYears: undefined,
  intentionJob: undefined,
  intentionCity: undefined,
  intentionSalary: undefined,
  selfEvaluation: undefined,
  status: undefined,
  isDefault: undefined,
  isDeleted: undefined,
  createTime: [],
})
const queryFormRef = ref() // 搜索的表单
const exportLoading = ref(false) // 导出的加载中

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await ResumeApi.getResumePage(queryParams)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}

/** 搜索按钮操作 */
const handleQuery = () => {
  queryParams.pageNo = 1
  getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value.resetFields()
  handleQuery()
}

/** 添加/修改操作 */
const formRef = ref()
const openForm = (type: string, id?: number) => {
  formRef.value.open(type, id)
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    // 发起删除
    await ResumeApi.deleteResume(id)
    message.success(t('common.delSuccess'))
    let currentRow;
    currentRow.value = {}
    // 刷新列表
    await getList()
  } catch {}
}

/** 批量删除简历主 */
const handleDeleteBatch = async () => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    await ResumeApi.deleteResumeList(checkedIds.value);
    checkedIds.value = [];
    message.success(t('common.delSuccess'))
    await getList();
  } catch {}
}

const checkedIds = ref<number[]>([])
const handleRowCheckboxChange = (records: Resume[]) => {
  checkedIds.value = records.map((item) => item.id!);
}

/** 导出按钮操作 */
const handleExport = async () => {
  try {
    // 导出的二次确认
    await message.exportConfirm()
    // 发起导出
    exportLoading.value = true
    const data = await ResumeApi.exportResume(queryParams)
    download.excel(data, '简历主.xls')
  } catch {
  } finally {
    exportLoading.value = false
  }
}

/** 初始化 **/
onMounted(() => {
  getList()
})
</script>
