---
title: olp中的列表实现
categories:
  - null
description: olp中的列表实现
date: 2025-08-28 10:21:52
tags:
  - olp
  - krjx
---

## Element UI + Vue3 实现大数据列表

### 任务1：JD 列表优化（添加字段）

过程：  

1. 调用新的 api 来获取新的数据

发现了问题，原本的 api 只需要传入一个 compId 就能获取到所有的参数（并且页面展示也是直接一个可滚动表格展示所有内容），而新的 api 需要传入 compId + pageNo + pageSize。  

那就改为列表来显示数据：  
没事了，新的 api 不管传入什么 pageNo、pageSize，仍然是返回所有的数据

{
    "id": 2819,
    "name": "Senior Wells Manager",
    "code": "AOSIWELLSSECPOS001",
    "sort": 0,
    "status": 0,
    "compId": 123,
    "companyName": null,
    "comShortname": null,
    "deptId": 646,
    "deptName": null,
    "depShortname": null,
    "sectId": null,
    "sectName": null,
    "shortName": null,
    "parentId": 2746,
    "ancestors": null,
    "dataSource": null,
    "jdPublished": null
}

返回的数据结构如上，请你帮我增加 Unique Code 列（值为"code": "AOSIHRXXPOS058" 的值），还有 Confirm 列（jdPublished 为 null 或 false 时，值为No；为 True 时，值为 Yes）

![alt text](AAA_PictureDir_olp中的列表实现/image.png)
![alt text](AAA_PictureDir_olp中的列表实现/image-1.png)

氯雷他定

tai nuo er

抗过敏

## 临时文件

<script setup lang="ts">
import { useRouter } from 'vue-router'
import 'splitpanes/dist/splitpanes.css'
import { Pane, Splitpanes } from 'splitpanes'
import { ElMessage, ElTree } from 'element-plus'
import { OrgType } from '@/enums/OrgType'
import { getSection, listSection, sectTreeSelect } from '@/api/system/section'
import { JobDescApi, type PostListReqVO } from '@/api/edp/jobDesc'
import { useI18n } from 'vue-i18n'

defineOptions({
  name: 'ProMatrix'
})

const { t } = useI18n()
const router = useRouter()
const message = useMessage() // 消息弹窗
const positionList = ref<any[]>([])
const allPositionList = ref<any[]>([]) // 存储所有数据
const sectionList = ref<any[]>([])
const positionOptions = ref<any[]>([])
const open = ref(false)
const loading = ref(true)
const ids = ref<number[]>([])
const title = ref('')
const postTotal = ref(0)
const sectionRef = ref()
const data = reactive<{
  form: any
  queryParams: any
  rules: any
}>({
  form: {},
  queryParams: {
    compId: undefined,
    id: undefined,
    name: undefined,
    code: undefined,
    sort: undefined,
    status: undefined,
    remark: undefined,
    createTime: undefined,
    companyName: undefined,
    comShortname: undefined,
    deptId: undefined,
    deptName: undefined,
    depShortname: undefined,
    sectId: undefined,
    sectName: undefined,
    shortName: undefined,
    parentId: undefined,
    ancestors: undefined,
    dataSource: undefined
  },
  rules: {
    name: [{ required: true, message: t('sys.post.positionNameRule'), trigger: 'blur' }],
    postCode: [{ required: false, message: t('sys.post.positionCodeRule'), trigger: 'blur' }],
    orderNum: [{ required: true, message: t('sys.post.orderNumRule'), trigger: 'blur' }]
  }
})

const { queryParams, form, rules } = toRefs(data)
const deptName = ref('')
const deptOptions = ref<any[]>([])
const postTreeRef = ref<InstanceType<typeof ElTree>>()
/** 当前选中的部门信息 */
const selDept = ref()
const defaultExpand = ref()
const isExpandAll = ref(false)
const loadingForm = ref(false)

/** 左侧-查询公司部门下拉树结构 */
const getDeptTree = async () => {
  const data = await sectTreeSelect({ type: 0 })
  console.log('部门树数据:', data)
  deptOptions.value = data
  selDept.value = deptOptions.value[0]
  queryParams.value.compId = deptOptions.value[0].id
  // 展开默认选中节点
  const firstOption = deptOptions.value[0]
  defaultExpand.value = [
    deptOptions.value[0].virtualId,
    firstOption.level === OrgType.Company && firstOption.children
      ? deptOptions.value[0].children[0].virtualId
      : undefined
  ]
  await nextTick(() => {
    postTreeRef.value?.setCurrentNode(deptOptions.value[0])
  })
  // 调用handleNodeClick传递第一个节点，模拟点击部门节点
  await handleNodeClick(deptOptions.value[0])
}
/** 通过条件过滤节点  */
const filterNode = (value: string, data: any) => {
  if (!value) return true
  return data.label.toLocaleLowerCase().includes(value.toLocaleLowerCase())
}
/** 根据名称筛选部门树 */
watch(deptName, (val) => {
  postTreeRef.value!.filter(val)
})
/** 查询Position列表 */
const getList = async () => {
  loading.value = true
  try {
    console.log('查询Position列表:', queryParams.value)
    // 使用接口获取所有数据
    const data = await JobDescApi.getPostList(queryParams.value)
    console.log('原始接口返回数据:', data)

    // 检查数据结构
    if (data && Array.isArray(data)) {
      // 如果直接返回数组
      allPositionList.value = data
      console.log('数据是数组，直接使用:', data.slice(0, 3)) // 打印前3条看结构
    } else if (data && data.list && Array.isArray(data.list)) {
      // 如果是包装在对象中的数组
      allPositionList.value = data.list
      console.log('数据在.list中，使用data.list:', data.list.slice(0, 3)) // 打印前3条看结构
    } else {
      allPositionList.value = []
      console.log('数据格式不正确:', data)
    }

    // 应用前端过滤
    filterPositionList()
    console.log('过滤后的positionList:', positionList.value.slice(0, 3)) // 打印前3条
  } finally {
    loading.value = false
  }
}

/** 前端过滤岗位列表 */
const filterPositionList = () => {
  console.log('开始过滤，allPositionList长度:', allPositionList.value.length)
  console.log('当前过滤条件:', {
    name: queryParams.value.name,
    code: queryParams.value.code,
    status: queryParams.value.status
  })

  let filteredList = [...allPositionList.value]
  console.log('初始filteredList长度:', filteredList.length)

  // 按名称过滤
  if (queryParams.value.name) {
    filteredList = filteredList.filter(
      (item) => item.name && item.name.toLowerCase().includes(queryParams.value.name.toLowerCase())
    )
    console.log('按名称过滤后长度:', filteredList.length)
  }

  // 按编码过滤
  if (queryParams.value.code) {
    filteredList = filteredList.filter(
      (item) => item.code && item.code.toLowerCase().includes(queryParams.value.code.toLowerCase())
    )
    console.log('按编码过滤后长度:', filteredList.length)
  }

  // 按状态过滤
  if (queryParams.value.status !== undefined) {
    filteredList = filteredList.filter((item) => item.status === queryParams.value.status)
    console.log('按状态过滤后长度:', filteredList.length)
  }

  positionList.value = filteredList
  postTotal.value = filteredList.length
  console.log('最终positionList长度:', positionList.value.length)
  console.log('最终positionList前3条数据:', positionList.value.slice(0, 3))
}

// 节点点击事件
const handleNodeClick = async (node: any) => {
  selDept.value = node

  if (node.level === OrgType.Company) {
    queryParams.value.compId = node.id
    queryParams.value.deptId = undefined
    queryParams.value.sectId = undefined
    queryParams.value.sectName = undefined
  }
  if (node.level === OrgType.Department) {
    queryParams.value.name = node.label
    queryParams.value.deptId = node.id
    queryParams.value.compId = undefined
    queryParams.value.sectId = undefined
    queryParams.value.sectName = undefined
  }
  if (node.level === OrgType.Section) {
    queryParams.value.sectName = node.label
    queryParams.value.sectId = node.id
    queryParams.value.deptId = undefined
    queryParams.value.name = undefined
    queryParams.value.compId = undefined
    const data = await getSection(node.id as number)
    queryParams.value.deptId = data.department.deptId
    queryParams.value.name = data.department.positionName
  }
  await getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryParams.value.name = undefined
  queryParams.value.code = undefined
  queryParams.value.status = undefined
  // 重置后重新过滤
  if (allPositionList.value.length > 0) {
    filterPositionList()
  } else {
    getList()
  }
}

// 监听搜索条件变化，实时过滤
watch(
  () => [queryParams.value.name, queryParams.value.code, queryParams.value.status],
  () => {
    if (allPositionList.value.length > 0) {
      filterPositionList()
    }
  },
  { deep: true }
)

// JD生成
const handleJDGenerate = (postId: number, positionName: string) => {
  console.log('Generating JD for position:', positionName)

  // 检查岗位名称是否为空
  if (!positionName) {
    console.warn('Position name is empty!')
    return
  }

  router.push({
    path: '/edp/chatbot',
    query: {
      postId,
      positionName,
      deptName: selDept.value?.label || '',
      createChat: 'true'
    }
  })
}

// 技能生成
const handleSkillGenerate = async (positionId: number, positionName: string) => {
  try {
    console.log('点击的岗位', positionId)
    if (!positionId) return

    const params = {
      pageNo: 1,
      pageSize: 10,
      positionId: positionId,
      arContent: '' // 添加必需的参数
    }

    const res = await JobDescApi.getJDVersionList(params)
    console.log('该岗位JD版本列表:', res)

    // 判断是否有JD版本
    const hasJDVersion = res.list.some((item) => item.id)
    if (!hasJDVersion) {
      ElMessage.warning('Please generate the job description for this position first.')
      return
    }

    // 判断岗位是否有已发布JD版本
    const hasPublishedVersion = res.list.some((item) => item.status === 2)
    if (!hasPublishedVersion) {
      ElMessage.warning('Please generate the job description for this position first.')
      return
    }

    await router.push({
      path: '/edp/skill-generate',
      query: {
        jdId: res.list[0].id,
        positionId,
        positionName,
        deptName: selDept.value?.label || ''
      }
    })
  } catch (error) {
    console.error('Skill Generate跳转失败', error)
  }
}

/** 初始化 */
onMounted(() => {
  getDeptTree()
})
</script>

<template>
  <div class="app-main-height">
    <Splitpanes class="default-theme">
      <Pane :size="26" class="!bg-white">
        <el-scrollbar class="!app-main-height p-2.5" height="calc(100vh)">
          <div class="head-container">
            <el-input
              v-model="deptName"
              :placeholder="t('sys.user.deptNamePH')"
              clearable
              class="mb-[20px]"
            >
              <template #prefix>
                <Icon class="mr-5px" icon="ep:search" />
              </template>
            </el-input>
          </div>
          <div class="head-container">
            <el-tree
              ref="postTreeRef"
              :data="deptOptions"
              :props="{ label: 'label', children: 'children' }"
              :expand-on-click-node="false"
              :default-expanded-keys="defaultExpand"
              :filter-node-method="filterNode"
              node-key="virtualId"
              highlight-current
              @node-click="handleNodeClick"
            >
              <template #default="{ node }">
                <div class="flex justify-between">
                  <div class="me-2.5">
                    <el-tag
                      v-if="node.data.level === OrgType.Company"
                      :title="node.data.shortName"
                      :style="{
                        '--el-tag-text-color': '#630EB8',
                        '--el-tag-bg-color': '#F3F1FF',
                        '--el-tag-border-color': '#D3CEF0'
                      }"
                    >
                      {{ node.data.shortName ? node.data.shortName : 'Company' }}
                    </el-tag>
                    <el-tag
                      v-else-if="node.data.level === OrgType.Department"
                      :title="node.data.shortName"
                    >
                      {{ node.data.shortName ? node.data.shortName : 'Dept' }}
                    </el-tag>
                    <el-tag v-else-if="node.data.level === OrgType.Section" type="info">
                      {{ 'Section' }}
                    </el-tag>
                  </div>
                  <span :title="node.label" class="whitespace-normal line-clamp-1 break-all">
                    {{ node.label }}</span
                  >
                </div>
              </template>
            </el-tree>
          </div>
        </el-scrollbar>
      </Pane>
      <Pane class="!app-main-height !bg-white">
        <el-scrollbar class="app-main-height p-2.5" height="calc(100vh)">
          <div class="pt-3 flex items-center">
            <div class="p-2">
              <el-tag
                v-if="selDept?.level === OrgType.Company"
                :style="{
                  '--el-tag-text-color': '#630EB8',
                  '--el-tag-bg-color': '#F3F1FF',
                  '--el-tag-border-color': '#D3CEF0'
                }"
              >
                {{ t('global.company') }}
              </el-tag>
              <el-tag v-if="selDept?.level === OrgType.Department">
                {{ t('sys.user.department') }}
              </el-tag>
              <el-tag v-else-if="selDept?.level === OrgType.Section" type="info">
                {{ t('sys.user.section') }}
              </el-tag>
              <span class="ms-2.5">{{ selDept?.label }}</span>
            </div>
          </div>
          <OrgTotalBar :number="postTotal" :text="t('sys.post.totalTip')" />

          <!-- 搜索工作栏 -->
          <ContentWrap>
            <el-form class="-mb-15px" :model="queryParams" :inline="true">
              <el-form-item label="岗位名称" prop="name">
                <el-input
                  v-model="queryParams.name"
                  placeholder="请输入岗位名称"
                  clearable
                  class="!w-240px"
                />
              </el-form-item>
              <el-form-item label="岗位编码" prop="code">
                <el-input
                  v-model="queryParams.code"
                  placeholder="请输入岗位编码"
                  clearable
                  class="!w-240px"
                />
              </el-form-item>
              <el-form-item>
                <el-button @click="resetQuery">
                  <Icon icon="ep:refresh" class="mr-5px" /> Reset
                </el-button>
              </el-form-item>
            </el-form>
          </ContentWrap>

          <!--JD列表-->
          <ContentWrap>
            <el-table v-loading="loading" :data="positionList" row-key="id">
              <!--岗位名称-->
              <el-table-column :label="t('sys.post.postName')" prop="name" min-width="360" />

              <!--岗位编码-->
              <el-table-column prop="code" align="center" label="Unique Code" width="150" />

              <!--数据来源-->
              <el-table-column
                prop="dataSource"
                align="center"
                :label="t('sys.company.dataSource')"
                width="110"
              />

              <!-- 状态 -->
              <el-table-column align="center" label="Status" width="110">
                <template #default="scope">
                  <span v-if="scope.row.status === 0">enable</span>
                  <span v-else>disable</span>
                </template>
              </el-table-column>

              <!-- 是否已经创建JD -->
              <el-table-column align="center" label="Confirm" width="110">
                <template #default="scope">
                  <span v-if="scope.row.jdPublished === true">Yes</span>
                  <span v-else>No</span>
                </template>
              </el-table-column>

              <!--创建者-->
              <el-table-column
                prop="createBy"
                align="center"
                :label="t('sys.company.creator')"
                width="110"
              />

              <!--操作-->
              <el-table-column
                :label="t('global.action')"
                min-width="180"
                align="center"
                class-name="small-padding fixed-width"
                fixed="right"
              >
                <template #default="scope">
                  <!--JD生成-->
                  <el-button
                    link
                    type="primary"
                    @click="handleJDGenerate(scope.row.id, scope.row.name)"
                  >
                    JD Generate
                  </el-button>

                  <!--技能生成-->
                  <el-button
                    link
                    type="primary"
                    @click="handleSkillGenerate(scope.row.id, scope.row.name)"
                  >
                    Skill Generate
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </ContentWrap>
        </el-scrollbar>
      </Pane>
    </Splitpanes>
  </div>
</template>


### 再次vi

<script setup lang="ts">
import { useRouter } from 'vue-router'
import 'splitpanes/dist/splitpanes.css'
import { Pane, Splitpanes } from 'splitpanes'
import { ElMessage, ElTree } from 'element-plus'
import { OrgType } from '@/enums/OrgType'
import { getSection, sectTreeSelect } from '@/api/system/section'
import { JobDescApi } from '@/api/edp/jobDesc'
import { useI18n } from 'vue-i18n'

defineOptions({
  name: 'ProMatrix'
})

const { t } = useI18n()
const router = useRouter()
const positionList = ref<any[]>([])
const allPositionList = ref<any[]>([]) // 存储所有数据
const loading = ref(true)
const postTotal = ref(0)
const data = reactive<{
  form: any
  queryParams: any
  rules: any
}>({
  form: {},
  queryParams: {
    compId: undefined,
    id: undefined,
    name: undefined,
    code: undefined,
    sort: undefined,
    status: undefined,
    remark: undefined,
    createTime: undefined,
    companyName: undefined,
    comShortname: undefined,
    deptId: undefined,
    deptName: undefined,
    depShortname: undefined,
    sectId: undefined,
    sectName: undefined,
    shortName: undefined,
    parentId: undefined,
    ancestors: undefined,
    dataSource: undefined
  },
  rules: {
    name: [{ required: true, message: t('sys.post.positionNameRule'), trigger: 'blur' }],
    postCode: [{ required: false, message: t('sys.post.positionCodeRule'), trigger: 'blur' }],
    orderNum: [{ required: true, message: t('sys.post.orderNumRule'), trigger: 'blur' }]
  }
})

const { queryParams } = toRefs(data)
const deptName = ref('')
const deptOptions = ref<any[]>([])
const postTreeRef = ref<InstanceType<typeof ElTree>>()
/** 当前选中的部门信息 */
const selDept = ref()
const defaultExpand = ref() /** 左侧-查询公司部门下拉树结构 */
const getDeptTree = async () => {
  const response = await sectTreeSelect({ type: 0, status: 1 })
  const data = response.data || response
  console.log('部门树数据:', data)
  deptOptions.value = data
  selDept.value = deptOptions.value[0]
  queryParams.value.compId = deptOptions.value[0].id
  // 展开默认选中节点
  const firstOption = deptOptions.value[0]
  defaultExpand.value = [
    deptOptions.value[0].virtualId,
    firstOption.level === OrgType.Company && firstOption.children
      ? deptOptions.value[0].children[0].virtualId
      : undefined
  ]
  await nextTick(() => {
    postTreeRef.value?.setCurrentNode(deptOptions.value[0])
  })
  // 调用handleNodeClick传递第一个节点，模拟点击部门节点
  await handleNodeClick(deptOptions.value[0])
}
/** 通过条件过滤节点  */
const filterNode = (value: string, data: any) => {
  if (!value) return true
  return data.label.toLocaleLowerCase().includes(value.toLocaleLowerCase())
}
/** 根据名称筛选部门树 */
watch(deptName, (val) => {
  postTreeRef.value!.filter(val)
})
/** 查询Position列表 */
const getList = async () => {
  loading.value = true
  try {
    console.log('查询Position列表:', queryParams.value)
    // 使用接口获取所有数据
    const data = await JobDescApi.getPostList(queryParams.value)
    console.log('原始接口返回数据:', data)

    // 检查数据结构
    if (data && Array.isArray(data)) {
      // 如果直接返回数组
      allPositionList.value = data
      console.log('数据是数组，直接使用:', data.slice(0, 3)) // 打印前3条看结构
    } else if (data && data.list && Array.isArray(data.list)) {
      // 如果是包装在对象中的数组
      allPositionList.value = data.list
      console.log('数据在.list中，使用data.list:', data.list.slice(0, 3)) // 打印前3条看结构
    } else {
      allPositionList.value = []
      console.log('数据格式不正确:', data)
    }

    // 应用前端过滤
    filterPositionList()
    console.log('过滤后的positionList:', positionList.value.slice(0, 3)) // 打印前3条
  } finally {
    loading.value = false
  }
}

/** 构建树形数据 */
const buildTreeFromFlatData = (flatData: any[]) => {
  console.log('构建树形数据，输入数据:', flatData)

  const map = new Map()
  const tree: any[] = []

  // 先将所有项添加到map中，初始化children数组
  flatData.forEach((item) => {
    map.set(item.id, { ...item, children: [] })
  })

  // 构建父子关系
  flatData.forEach((item) => {
    const treeItem = map.get(item.id)
    if (item.parentId && map.has(item.parentId)) {
      const parent = map.get(item.parentId)
      parent.children.push(treeItem)
    } else {
      // 没有parentId或parentId不存在的作为根节点
      tree.push(treeItem)
    }
  })

  console.log('构建完成的树形数据:', tree)
  return tree
}

/** 前端过滤岗位列表 */
const filterPositionList = () => {
  console.log('开始过滤，allPositionList长度:', allPositionList.value.length)
  console.log('当前过滤条件:', {
    name: queryParams.value.name,
    code: queryParams.value.code,
    status: queryParams.value.status
  })

  let filteredList = [...allPositionList.value]
  console.log('初始filteredList长度:', filteredList.length)

  // 按名称过滤
  if (queryParams.value.name) {
    filteredList = filteredList.filter(
      (item) => item.name && item.name.toLowerCase().includes(queryParams.value.name.toLowerCase())
    )
    console.log('按名称过滤后长度:', filteredList.length)
  }

  // 按编码过滤
  if (queryParams.value.code) {
    filteredList = filteredList.filter(
      (item) => item.code && item.code.toLowerCase().includes(queryParams.value.code.toLowerCase())
    )
    console.log('按编码过滤后长度:', filteredList.length)
  }

  // 按状态过滤
  if (queryParams.value.status !== undefined) {
    filteredList = filteredList.filter((item) => item.status === queryParams.value.status)
    console.log('按状态过滤后长度:', filteredList.length)
  }

  // 构建树形结构
  const treeData = buildTreeFromFlatData(filteredList)
  positionList.value = treeData
  postTotal.value = filteredList.length
  console.log('树形数据:', JSON.stringify(treeData, null, 2))
}

// 节点点击事件
const handleNodeClick = async (node: any) => {
  selDept.value = node

  if (node.level === OrgType.Company) {
    queryParams.value.compId = node.id
    queryParams.value.deptId = undefined
    queryParams.value.sectId = undefined
    queryParams.value.sectName = undefined
  }
  if (node.level === OrgType.Department) {
    queryParams.value.name = node.label
    queryParams.value.deptId = node.id
    queryParams.value.compId = undefined
    queryParams.value.sectId = undefined
    queryParams.value.sectName = undefined
  }
  if (node.level === OrgType.Section) {
    queryParams.value.sectName = node.label
    queryParams.value.sectId = node.id
    queryParams.value.deptId = undefined
    queryParams.value.name = undefined
    queryParams.value.compId = undefined
    const data = await getSection(node.id as number)
    queryParams.value.deptId = data.department.deptId
    queryParams.value.name = data.department.positionName
  }
  await getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryParams.value.name = undefined
  queryParams.value.code = undefined
  queryParams.value.status = undefined
  // 重置后重新过滤
  if (allPositionList.value.length > 0) {
    filterPositionList()
  } else {
    getList()
  }
}

// 监听搜索条件变化，实时过滤
watch(
  () => [queryParams.value.name, queryParams.value.code, queryParams.value.status],
  () => {
    if (allPositionList.value.length > 0) {
      filterPositionList()
    }
  },
  { deep: true }
)

// JD生成
const handleJDGenerate = (postId: number, positionName: string) => {
  console.log('Generating JD for position:', positionName)

  // 检查岗位名称是否为空
  if (!positionName) {
    console.warn('Position name is empty!')
    return
  }

  router.push({
    path: '/edp/chatbot',
    query: {
      postId,
      positionName,
      deptName: selDept.value?.label || '',
      createChat: 'true'
    }
  })
}

// 技能生成
const handleSkillGenerate = async (positionId: number, positionName: string) => {
  try {
    console.log('点击的岗位', positionId)
    if (!positionId) return

    const params = {
      pageNo: 1,
      pageSize: 10,
      positionId: positionId,
      arContent: '' // 添加必需的参数
    }

    const res = await JobDescApi.getJDVersionList(params)
    console.log('该岗位JD版本列表:', res)

    // 判断是否有JD版本
    const hasJDVersion = res.list.some((item) => item.id)
    if (!hasJDVersion) {
      ElMessage.warning('Please generate the job description for this position first.')
      return
    }

    // 判断岗位是否有已发布JD版本
    const hasPublishedVersion = res.list.some((item) => item.status === 2)
    if (!hasPublishedVersion) {
      ElMessage.warning('Please generate the job description for this position first.')
      return
    }

    await router.push({
      path: '/edp/skill-generate',
      query: {
        jdId: res.list[0].id,
        positionId,
        positionName,
        deptName: selDept.value?.label || ''
      }
    })
  } catch (error) {
    console.error('Skill Generate跳转失败', error)
  }
}

/** 初始化 */
onMounted(() => {
  getDeptTree()
})
</script>

<template>
  <div class="app-main-height">
    <Splitpanes class="default-theme">
      <Pane :size="26" class="!bg-white">
        <el-scrollbar class="!app-main-height p-2.5" height="calc(100vh)">
          <div class="head-container">
            <el-input
              v-model="deptName"
              :placeholder="t('sys.user.deptNamePH')"
              clearable
              class="mb-[20px]"
            >
              <template #prefix>
                <Icon class="mr-5px" icon="ep:search" />
              </template>
            </el-input>
          </div>
          <div class="head-container">
            <el-tree
              ref="postTreeRef"
              :data="deptOptions"
              :props="{ label: 'label', children: 'children' }"
              :expand-on-click-node="false"
              :default-expanded-keys="defaultExpand"
              :filter-node-method="filterNode"
              node-key="virtualId"
              highlight-current
              @node-click="handleNodeClick"
            >
              <template #default="{ node }">
                <div class="flex justify-between">
                  <div class="me-2.5">
                    <el-tag
                      v-if="node.data.level === OrgType.Company"
                      :title="node.data.shortName"
                      :style="{
                        '--el-tag-text-color': '#630EB8',
                        '--el-tag-bg-color': '#F3F1FF',
                        '--el-tag-border-color': '#D3CEF0'
                      }"
                    >
                      {{ node.data.shortName ? node.data.shortName : 'Company' }}
                    </el-tag>
                    <el-tag
                      v-else-if="node.data.level === OrgType.Department"
                      :title="node.data.shortName"
                    >
                      {{ node.data.shortName ? node.data.shortName : 'Dept' }}
                    </el-tag>
                    <el-tag v-else-if="node.data.level === OrgType.Section" type="info">
                      {{ 'Section' }}
                    </el-tag>
                  </div>
                  <span :title="node.label" class="whitespace-normal line-clamp-1 break-all">
                    {{ node.label }}</span
                  >
                </div>
              </template>
            </el-tree>
          </div>
        </el-scrollbar>
      </Pane>
      <Pane class="!app-main-height !bg-white">
        <el-scrollbar class="app-main-height p-2.5" height="calc(100vh)">
          <div class="pt-3 flex items-center">
            <div class="p-2">
              <el-tag
                v-if="selDept?.level === OrgType.Company"
                :style="{
                  '--el-tag-text-color': '#630EB8',
                  '--el-tag-bg-color': '#F3F1FF',
                  '--el-tag-border-color': '#D3CEF0'
                }"
              >
                {{ t('global.company') }}
              </el-tag>
              <el-tag v-if="selDept?.level === OrgType.Department">
                {{ t('sys.user.department') }}
              </el-tag>
              <el-tag v-else-if="selDept?.level === OrgType.Section" type="info">
                {{ t('sys.user.section') }}
              </el-tag>
              <span class="ms-2.5">{{ selDept?.label }}</span>
            </div>
          </div>
          <OrgTotalBar :number="postTotal" :text="t('sys.post.totalTip')" />

          <!-- 搜索工作栏 -->
          <ContentWrap>
            <el-form class="-mb-15px" :model="queryParams" :inline="true">
              <el-form-item label="岗位名称" prop="name">
                <el-input
                  v-model="queryParams.name"
                  placeholder="请输入岗位名称"
                  clearable
                  class="!w-240px"
                />
              </el-form-item>
              <el-form-item label="岗位编码" prop="code">
                <el-input
                  v-model="queryParams.code"
                  placeholder="请输入岗位编码"
                  clearable
                  class="!w-240px"
                />
              </el-form-item>
              <el-form-item>
                <el-button @click="resetQuery">
                  <Icon icon="ep:refresh" class="mr-5px" /> Reset
                </el-button>
              </el-form-item>
            </el-form>
          </ContentWrap>

          <!--JD列表-->
          <ContentWrap>
            <el-table
              v-loading="loading"
              :data="positionList"
              row-key="id"
              border
              default-expand-all
            >
              <!--岗位名称-->
              <el-table-column :label="t('sys.post.postName')" prop="name" min-width="360" />

              <!--岗位编码-->
              <el-table-column prop="code" align="center" label="Unique Code" width="150" />

              <!--数据来源-->
              <el-table-column
                prop="dataSource"
                align="center"
                :label="t('sys.company.dataSource')"
                width="110"
              />

              <!-- 状态 -->
              <el-table-column align="center" label="Status" width="110">
                <template #default="scope">
                  <span v-if="scope.row.status === 0">enable</span>
                  <span v-else>disable</span>
                </template>
              </el-table-column>

              <!-- 是否已经创建JD -->
              <el-table-column align="center" label="Confirm" width="110">
                <template #default="scope">
                  <span v-if="scope.row.jdPublished === true">Yes</span>
                  <span v-else>No</span>
                </template>
              </el-table-column>

              <!--创建者-->
              <el-table-column
                prop="createBy"
                align="center"
                :label="t('sys.company.creator')"
                width="110"
              />

              <!--操作-->
              <el-table-column
                :label="t('global.action')"
                min-width="180"
                align="center"
                class-name="small-padding fixed-width"
                fixed="right"
              >
                <template #default="scope">
                  <!--JD生成-->
                  <el-button
                    link
                    type="primary"
                    @click="handleJDGenerate(scope.row.id, scope.row.name)"
                  >
                    JD Generate
                  </el-button>

                  <!--技能生成-->
                  <el-button
                    link
                    type="primary"
                    @click="handleSkillGenerate(scope.row.id, scope.row.name)"
                  >
                    Skill Generate
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </ContentWrap>
        </el-scrollbar>
      </Pane>
    </Splitpanes>
  </div>
</template>
