<!--
 * @Author: 祝占朋 wb.zhuzhanpeng01@mesg.corp.netease.com
 * @Date: 2023-12-26 14:49:41
 * @LastEditors: Ianarua 306781523@qq.com
 * @LastEditTime: 2024-08-06 10:09:33
 * @FilePath: front_end/src/components/OptionList.vue
 * @Description: 
-->
<template>
  <a-config-provider :theme="{ token: { colorPrimary: '#5a47e5' } }">
    <div class="list-page">
      <div class="content">
        <div class="options">
          <div class="to-chat" @click="goChat">
            <img src="../assets/home/icon-back.png" alt="back" />
            <span>{{ home.conversation }}</span>
          </div>
          <p class="kb-name">
            <span class="name">
              {{ currentKbName.slice(0, 15) }}
            </span>
            <span class="id"> {{ home.knowledgeID }} {{ currentId }} </span>
          </p>
          <div v-if="navIndex === 0" class="kb-tag">
            <a-popover trigger="click" placement="leftBottom">
              <template #content>
                <TagsInput
                  @confirm-tag="
                    newTags => {
                      tagConfirm('kb', currentId, newTags);
                    }
                  "
                />
              </template>
              <a-button type="primary" style="margin-right: 10px">所有文件一键添加tag</a-button>
            </a-popover>
            <a-popover trigger="click" placement="leftBottom">
              <template #content>
                <TagsInput
                  @confirm-tag="
                    newTags => {
                      tagConfirm('fileBatch', [...selectedKeys.keys()], newTags);
                    }
                  "
                />
              </template>
              <a-button>批量添加tag</a-button>
            </a-popover>
          </div>
        </div>
        <div class="nav-info">
          <div class="navs">
            <div :class="['nav-item', 'nav-item-active']">
              {{ navIndex === 0 ? home.docSet : home.qaSet }}
            </div>
          </div>
          <div v-if="navIndex === 0" class="nav-progress">
            <UploadProgress :data-source="dataSource" />
          </div>
          <div class="handle-btn">
            <a-button v-if="navIndex === 0" danger class="clear-upload" @click="clearUpload">
              {{ home.clearAllFile }}
            </a-button>
            <a-button v-if="navIndex === 0" class="upload" @click="showFileUpload">
              {{ home.upload }}
            </a-button>
            <a-button v-if="navIndex === 0" class="add-link" @click="showUrlUpload">
              {{ home.addUrl }}
            </a-button>
            <a-button v-if="navIndex === 1" class="upload" @click="showEditQaSet">
              {{ home.inputQa }}
            </a-button>
          </div>
        </div>
        <div class="table">
          <a-table
            v-if="navIndex === 0"
            :data-source="dataSource"
            :columns="columns"
            :pagination="kbPaginationConfig"
            :locale="{ emptyText: home.emptyText }"
            :hide-on-single-page="true"
            :show-size-changer="false"
            :row-selection="{ selectedRowKeys: [...selectedKeys.keys()], onSelect, onSelectAll }"
            @change="kbOnChange"
          >
            <template #headerCell="{ column }">
              <!--            fileIdName-->
              <template v-if="column.key === 'status'">
                <span>{{ home.documentStatus }}</span>
                <a-tooltip color="#5a47e5">
                  <template #title>
                    {{ home.documentStatusNode }}
                  </template>
                  <img
                    src="@/assets/home/icon-question.png"
                    style="width: 18px; margin-left: 5px"
                    alt="&copy"
                  />
                </a-tooltip>
              </template>
            </template>

            <template #bodyCell="{ column, record }">
              <template v-if="column.key === 'fileIdName'">
                <a-tooltip color="#fff" placement="topLeft">
                  <template #title>
                    <span style="color: #666; user-select: text">{{ record.fileIdName }}</span>
                  </template>
                  <span>{{ record.fileIdName }}</span>
                </a-tooltip>
              </template>
              <template v-else-if="column.key === 'fileTag'">
                <Tags
                  v-if="record.status === 'green'"
                  :tags="record.fileTag"
                  @update:tags="
                    newTags => {
                      record.fileTag = newTags;
                    }
                  "
                  @confirm-tag="
                    newTags => {
                      tagConfirm('file', [record.fileId], newTags);
                    }
                  "
                />
              </template>
              <template v-else-if="column.key === 'status'">
                <div class="status-box">
                  <span class="icon-file-status">
                    <LoadingImg
                      v-if="record.status === 'gray' || record.status === 'yellow'"
                      class="file-status"
                    />
                    <SvgIcon
                      v-else
                      class="file-status"
                      :name="record.status === 'green' ? 'success' : 'error'"
                    />
                  </span>
                  <span> {{ parseStatus(record.status) }}</span>
                </div>
              </template>
              <template v-else-if="column.key === 'remark'">
                <div v-if="typeof record.remark === 'string'">{{ record.remark }}</div>
                <div v-else>
                  <p v-for="(value, key) in record.remark" :key="key">
                    {{ `${key}: ${value}` }}
                  </p>
                </div>
              </template>
              <template v-else-if="column.key === 'options'">
                <a-popconfirm
                  overlay-class-name="del-pop"
                  placement="topRight"
                  :title="common.deleteTitle"
                  :ok-text="common.confirm"
                  :cancel-text="common.cancel"
                  @confirm="confirm"
                >
                  <!-- :disabled="record.status == 'gray' || record.status === 'yellow'" -->
                  <a-button type="text" class="delete-item" @click="deleteItem(record)">
                    {{ common.delete }}
                  </a-button>
                </a-popconfirm>
                <a-button
                  type="text"
                  class="view-item"
                  :disabled="!(record.status === 'green')"
                  @click="viewItem(record)"
                >
                  {{ common.view }}
                </a-button>
              </template>
            </template>
          </a-table>
          <a-table
            v-else
            :data-source="faqList"
            :columns="qaColumns"
            :locale="{ emptyText: home.emptyText }"
            :loading="loading"
            :pagination="paginationConfig"
            @change="onChange"
          >
            <template #bodyCell="{ column, record }">
              <template v-if="column.key === 'status'">
                <div class="status-box">
                  <span class="icon-file-status">
                    <LoadingImg
                      v-if="record.status === 'gray' || record.status === 'yellow'"
                      class="file-status"
                    />
                    <SvgIcon
                      v-else
                      class="file-status"
                      :name="record.status === 'green' ? 'success' : 'error'"
                    />
                  </span>
                  <span> {{ parseFaqStatus(record.status) }}</span>
                </div>
              </template>
              <template v-else-if="column.key === 'options'">
                <div class="options">
                  <a-button
                    class="edit-item"
                    type="link"
                    :disabled="record.status !== 'green'"
                    @click="editQaItem(record)"
                  >
                    {{ bots.edit }}
                  </a-button>
                  <a-popconfirm
                    overlay-class-name="qa-del-pop"
                    placement="topRight"
                    :title="home.deleteQaSetText"
                    :ok-text="common.confirm"
                    :cancel-text="common.cancel"
                    @confirm="qaConfirm"
                  >
                    <a-button class="delete-item" danger type="link" @click="deleteQaItem(record)">
                      {{ common.delete }}
                    </a-button>
                  </a-popconfirm>
                </div>
              </template>
            </template>
          </a-table>
        </div>
      </div>
    </div>
    <ChunkViewDialog :kb-id="currentId" :file-id="fileId" :file-name="fileIdName" />
    <FileUploadDialog :dialog-type="0" />
  </a-config-provider>
</template>
<script lang="ts" setup>
import urlResquest from '@/services/urlConfig';
import { useKnowledgeBase } from '@/store/useKnowledgeBase';
import { useKnowledgeModal } from '@/store/useKnowledgeModal';
import { useChunkView } from '@/store/useChunkView';
import { useOptiionList } from '@/store/useOptiionList';
import { pageStatus } from '@/utils/enum';
import { resultControl } from '@/utils/utils';
import { message, Modal } from 'ant-design-vue';
import { getLanguage } from '@/language';
import LoadingImg from '@/components/LoadingImg.vue';
import UploadProgress from '@/components/UploadProgress.vue';
import ChunkViewDialog from '@/components/ChunkViewDialog.vue';
import FileUploadDialog from '@/components/FileUploadDialog.vue';
// import { PlusOutlined } from '@ant-design/icons-vue';
import Tags from '@/components/Tags.vue';
import TagsInput from '@/components/TagsInput.vue';

const { setDefault } = useKnowledgeBase();
const { currentKbName, currentId } = storeToRefs(useKnowledgeBase());
const { setModalVisible, setUrlModalVisible, setModalTitle } = useKnowledgeModal();
const { showChunkModel } = storeToRefs(useChunkView());
const {
  getDetails,
  setEditQaSet,
  setEditModalVisible,
  getFaqList,
  setFaqType,
  setPageNum,
  setKbPageNum,
} = useOptiionList();
const {
  dataSource,
  faqList,
  timer,
  faqTimer,
  total,
  pageNum,
  pageSize,
  loading,
  kbTotal,
  kbPageNum,
  kbPageSize,
} = storeToRefs(useOptiionList());

const home = getLanguage().home;
const common = getLanguage().common;
const bots = getLanguage().bots;

const navIndex = computed(() => computedKbType.value);

// const navList = [
//   {
//     name: home.docSet,
//     value: 0,
//   },
//   {
//     name: home.qaSet,
//     value: 1,
//   },
// ];

const computedKbType = computed(() => {
  if (currentId.value.endsWith('_FAQ')) {
    return 1;
  } else {
    return 0;
  }
});

const columns = [
  {
    title: home.documentId,
    dataIndex: 'fileId',
    key: 'fileId',
    width: '8%',
  },
  {
    title: home.documentName,
    dataIndex: 'fileIdName',
    key: 'fileIdName',
    width: '12%',
    ellipsis: true,
  },
  {
    title: home.documentTag,
    dataIndex: 'fileTag',
    key: 'fileTag',
    width: '12%',
  },
  {
    title: home.documentStatus,
    dataIndex: 'status',
    key: 'status',
    width: '10%',
    ellipsis: true,
  },
  {
    title: home.fileSize,
    dataIndex: 'bytes',
    key: 'bytes',
    width: '8%',
  },
  {
    title: home.contentLength,
    dataIndex: 'contentLength',
    key: 'contentLength',
    width: '10%',
  },
  {
    title: home.creationDate,
    dataIndex: 'createtime',
    key: 'createtime',
    width: '10%',
  },
  {
    title: home.remark,
    dataIndex: 'remark',
    key: 'remark',
    width: '15%',
  },
  {
    title: home.operate,
    key: 'options',
    width: '10%',
  },
];

const qaColumns = [
  {
    title: 'ID',
    dataIndex: 'id',
    key: 'id',
    width: '8%',
  },
  {
    title: home.question,
    dataIndex: 'question',
    key: 'question',
    width: '43%',
    ellipsis: true,
  },
  {
    title: home.status,
    dataIndex: 'status',
    key: 'status',
    width: '10%',
    ellipsis: true,
  },
  {
    title: home.characterCount,
    dataIndex: 'bytes',
    key: 'bytes',
    width: '10%',
  },
  {
    title: home.creationDate,
    dataIndex: 'createtime',
    key: 'createtime',
    width: '11%',
  },
  {
    title: home.operate,
    key: 'options',
    width: '10%',
  },
];

// 新增 / 删除 标签调用
const tagConfirm = async (
  type: 'file' | 'fileBatch' | 'kb',
  id: string[] | string,
  newTags: Array<string>
) => {
  // TODO 调用接口
  if (type === 'file') {
    console.log(type, id, newTags);
    await resultControl(
      await urlResquest.updateTags({
        tags: newTags,
        file_ids: id,
        is_replace: true,
      })
    );
    message.success('成功修改标签');
  } else if (type === 'fileBatch') {
    await resultControl(
      await urlResquest.updateTags({
        tags: newTags,
        file_ids: id,
        is_replace: false,
      })
    );
    message.success('成功批量添加标签');
    selectedKeys.value.clear();
    await getDetails();
  } else if (type === 'kb') {
    console.log(type, id, newTags);
    await resultControl(
      await urlResquest.updateTags({
        tags: newTags,
        kb_id: id,
        is_replace: false,
      })
    );
    message.success('成功为所有文件添加标签');
    await getDetails();
  }
};

// 知识库tag
// const kbTag = ref([
//   '阿斯蒂芬规划局快乐美女吧v冲洗着',
//   '阿a蒂芬规划局快乐美女吧v冲洗着',
//   '阿斯大三蒂芬规划局快乐美女吧v冲洗着',
//   '阿斯大三蒂芬规对对对局快乐美女吧v冲洗着',
//   '阿斯大三蒂芬vvv规划局快乐美女吧v冲洗着',
//   '阿斯大三蒂芬啊撒大声地规划局快乐美女吧v冲洗着',
// ]);

// 知识库的分页参数
const kbPaginationConfig = computed(() => ({
  current: kbPageNum.value, // 当前页码
  pageSize: kbPageSize.value, // 每页条数
  total: kbTotal.value, // 数据总数
  showSizeChanger: false,
  showTotal: total => `共 ${total} 条`,
}));

// faq的分页参数
const paginationConfig = computed(() => ({
  current: pageNum.value, // 当前页码
  pageSize: pageSize.value, // 每页条数
  total: total.value, // 数据总数
  showSizeChanger: false,
  showTotal: total => `共 ${total} 条`,
}));

let optionItem: any = {};

const deleteItem = item => {
  optionItem = item;
};

// 预览chunks
const fileId = ref('');
const fileIdName = ref('');
const viewItem = async item => {
  fileId.value = item.fileId;
  fileIdName.value = item.fileIdName;
  // 打开弹窗
  showChunkModel.value = true;
};

const confirm = async () => {
  try {
    await resultControl(
      await urlResquest.deleteFile({ file_ids: [optionItem.fileId], kb_id: currentId.value })
    );
    message.success('删除成功');
    await getDetails();
    if (kbPageNum.value !== 1 && dataSource.value.length === 0) {
      kbPageNum.value -= 1;
      await getDetails();
    }
  } catch (e) {
    message.error(e.msg || '删除失败');
  }
};

let qaOptionItem: any = {};

const deleteQaItem = item => {
  qaOptionItem = item;
};

const qaConfirm = async () => {
  try {
    await resultControl(
      await urlResquest.deleteFile({
        kb_id: `${currentId.value}_FAQ`,
        file_ids: [qaOptionItem.faqId],
      })
    );
    message.success('删除成功');
    await getFaqList();
    if (pageNum.value !== 1 && faqList.value.length === 0) {
      pageNum.value -= 1;
      await getFaqList();
    }
  } catch (e) {
    message.error(e.msg || '删除失败');
  }
};

const editQaItem = item => {
  setFaqType('edit');
  setEditQaSet(item);
  setEditModalVisible(true);
};
const goChat = () => {
  setDefault(pageStatus.normal);
};

const showFileUpload = () => {
  setModalVisible(true);
  setModalTitle(home.upload);
};

const showUrlUpload = () => {
  setUrlModalVisible(true);
  setModalTitle(common.addUrl);
};

const showEditQaSet = () => {
  setFaqType('upload');
  setEditModalVisible(true);
};

const clearUpload = () => {
  Modal.confirm({
    title: home.clearAllFile,
    content: h('p', home.clearAllFileConfirm),
    centered: true,
    maskClosable: true,
    okText: common.confirm,
    okType: 'danger',
    // cancelText: common.cancel,
    async onOk() {
      try {
        await resultControl(await urlResquest.clearUpload({ status: 'gray', kb_ids: [] }));
        message.success('操作成功');
        getDetails();
      } catch (e) {
        message.error(e.msg || '操作失败');
      }
    },
  });
};

const parseStatus = status => {
  let str: string;
  switch (status) {
    case 'gray':
      str = common.inLine;
      break;
    case 'yellow':
      str = common.parsing;
      break;
    case 'green':
      str = common.succeeded;
      break;
    default:
      str = common.failed;
      break;
  }
  return str;
};

const parseFaqStatus = status => {
  let str = common.failed;
  switch (status) {
    case 'gray':
      str = common.uploadCompleted;
      break;
    case 'yellow':
      str = common.inLine;
      break;
    case 'green':
      str = common.learningCompleted;
      break;
    default:
      break;
  }
  return str;
};

// const checkKbIsCreate = async () => {
//   try {
//     const res: any = await resultControl(await urlResquest.kbList());
//     if (res.find(item => item.kb_id === `${currentId.value}_FAQ`)) {
//       return true;
//     }
//   } catch (e) {
//     message.error(e.msg || common.error);
//     return true;
//   }
//   return false;
// };

// const addKnowledge = async () => {
//   const isCreate = await checkKbIsCreate();
//   if (isCreate) {
//     return;
//   }
//   //获取到知识库id后  赋值给newId
//   try {
//     await resultControl(
//       await urlResquest.createKb({
//         kb_name: `${currentKbName.value}_FAQ`,
//         kb_id: `${currentId.value}_FAQ`,
//         // faq: true,
//       })
//     );
//   } catch (e) {
//     message.error(e.msg || common.error);
//     return false;
//   }
//   return true;
// };

// const navClick = value => {
//   navIndex.value = value;
//   if (value === 0) {
//     setKbPageNum(1);
//     clearTimeout(faqTimer.value);
//     getDetails();
//   } else {
//     setPageNum(1);
//     clearTimeout(timer.value);
//     getFaqList();
//     addKnowledge();
//   }
// };

const onChange = pagination => {
  const { current } = pagination;
  setPageNum(current);
  getFaqList();
};

const kbOnChange = pagination => {
  setKbPageNum(pagination.current);
  getDetails();
};

// 多选
const selectedKeys = ref<Map<string, string>>(new Map());

// 点击多选框
const onSelect = (selectedRow: any) => {
  const key = selectedRow.fileId;
  const fileIdName = selectedRow.fileIdName;
  if (selectedKeys.value.has(key)) {
    selectedKeys.value.delete(key);
  } else {
    selectedKeys.value.set(key, fileIdName);
  }
};

// 点击全选框
const onSelectAll = (...args) => {
  const changeRows = args[2];
  changeRows.map(item => {
    const key = item.key;
    const fileIdName = item.fileIdName;
    if (selectedKeys.value.has(key)) {
      selectedKeys.value.delete(key);
    } else {
      selectedKeys.value.set(key, fileIdName);
    }
  });
};

watch(
  currentId,
  () => {
    // navIndex.value = 0;
    setKbPageNum(1);
    setPageNum(1);
    getDetails();
  },
  {
    immediate: true,
  }
);

onBeforeUnmount(() => {
  clearTimeout(timer.value);
  clearTimeout(faqTimer.value);
  setKbPageNum(1);
  setPageNum(1);
});
</script>

<style lang="scss" scoped>
.list-page {
  overflow: hidden;
  width: 100%;
  height: 100%;
  font-family: PingFang SC;

  .content {
    height: calc(100vh - 64px);
    // margin-top: 16px;
    padding: 24px 32px;
    background: #f3f6fd;
    border-radius: 12px 0 0 0;
  }
}

.options {
  display: flex;
  align-items: center;

  .to-chat {
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 40px;
    background: #5a47e5;
    border-radius: 6px;
    padding: 8px 20px;

    img {
      margin-right: 4px;
      width: 20px;
      height: 20px;
    }

    span {
      white-space: nowrap;
      overflow: hidden;
      font-size: 16px;
      font-weight: 500;
      line-height: 24px;
      color: #ffffff;
    }
  }

  .kb-name {
    margin-left: 20px;
    margin-right: 30px;
    font-size: 24px;
    font-weight: 500;
    line-height: 32px;
    color: #222222;

    .name {
      margin-right: 20px;
    }

    .id {
      font-size: 14px;
      font-weight: 400;
      color: #999;
    }
  }

  .kb-tag {
    flex: 1;
    display: flex;
    justify-content: flex-end;
  }
}

.nav-info {
  width: 100%;
  height: 40px;
  margin: 20px 0 14px 0;
  display: flex;
  justify-content: space-between;

  .navs {
    height: 40px;
    padding: 4px;
    border-radius: 8px;
    background: #e4e9f4;
    display: flex;

    .nav-item {
      min-width: 100px;
      padding: 0 20px;
      height: 32px;
      font-size: 16px;
      color: #666666;
      border-radius: 6px;
      text-align: center;
      line-height: 32px;
      //cursor: pointer;
    }

    .nav-item-active {
      background: #fff;
      font-weight: 500;
      color: #5a47e5;
    }
  }

  .nav-progress {
    width: 40%;
    margin: 0 10px 50px;
    //margin-right: 50px;
    display: flex;
    //flex: 1;
    align-items: center;
  }

  .handle-btn {
    display: flex;

    .clear-upload {
      //width: 100px;
      height: 40px;
      margin-right: 10px;
    }

    .upload {
      cursor: pointer;
      height: 40px;
      padding: 8px 20px;
      border-radius: 4px;
      background: #5a47e5;
      font-size: 16px;
      font-weight: 500;
      line-height: 24px;
      color: #ffffff;
    }

    .add-link {
      cursor: pointer;
      height: 40px;
      margin-left: 16px;
      padding: 8px 20px;
      border-radius: 4px;
      background: #ffffff;
      border: 1px solid #5a47e5;
      font-size: 16px;
      font-weight: 500;
      line-height: 24px;
      color: #5a47e5;
    }
  }
}

.table {
  height: calc(100% - 120px);
  margin-bottom: 32px;
  overflow: auto;
  border-radius: 12px;
  background-color: #fff;

  &::-webkit-scrollbar {
    width: 0;
  }

  .options {
    width: 80px;
    display: flex;
    justify-content: space-between;
  }

  .delete-item {
    padding: 2px;
    font-size: 14px;
    font-weight: normal;
    line-height: 22px;
    margin-right: 5px;
    /* 错误颜色 */
    color: #ff524c;
  }

  .view-item {
    padding: 2px;
    font-size: 14px;
    font-weight: normal;
    line-height: 22px;
    margin-right: 5px;
    color: #4d71ff;
  }

  .edit-item {
    padding: 0;
  }

  .file-status {
    width: 16px;
    height: 16px;
  }

  .status-box {
    display: flex;
    align-items: center;

    .icon-file-status {
      display: flex;
      align-items: center;
    }

    span {
      display: block;

      margin-right: 8px;

      svg {
        width: 16px;
        height: 16px;
      }
    }
  }

  //:deep(.ant-table-cell-ellipsis[colstart='2']) {
  //  //display: flex;
  //  //align-items: center;
  //  //img {
  //  //  margin-top: 5px;
  //  //}
  //}
}

:deep(.ant-table-wrapper .ant-table-thead > tr > th) {
  font-size: 16px !important;
  font-weight: 500 !important;
  line-height: 24px !important;
  padding: 15px 0 15px 36px !important;

  color: #222222 !important;
  background-color: #e9edf7;

  .small {
    font-size: 12px !important;
  }

  &:before {
    width: 0 !important;
  }
}

:deep(.ant-table-tbody > tr > td) {
  font-size: 14px;
  font-weight: normal;
  line-height: 22px;
  color: #666666;
  background-color: #fff;
  padding: 20px 0 20px 36px !important;
  border: 0 !important;
  box-shadow: inset 0px -1px 0px 0px rgba(0, 0, 0, 0.05);

  &:hover {
    background-color: rgba(233, 237, 247, 0.3);
  }
}

// :deep(.ant-pagination-item-link) {
//   border-radius: 4px !important;
//   box-sizing: border-box !important;
//   border: 1px solid #dde2ec !important;
// }

:deep(.ant-pagination) {
  margin: 16px 20px !important;
}

:deep(.ant-pagination-item) {
  box-sizing: border-box !important;
  border: 1px solid #dde2ec !important;
}

:deep(.ant-pagination-item-active) {
  background: #5a47e5 !important;
  color: #fff !important;

  a {
    color: #fff !important;
  }
}

:deep(.options > .ant-btn) {
  height: auto;
}

:deep(.ant-btn-link) {
  color: #5a47e5;
}

:deep(.ant-btn-link:disabled) {
  color: rgba(0, 0, 0, 0.25);
}

:deep(.ant-table-empty .ant-table-placeholder .ant-table-cell) {
  color: #999999 !important;
}
</style>

<style lang="scss">
.del-pop {
  margin-right: 10px;

  .ant-popover-content {
    .ant-btn-default {
      padding: 1px 8px;
      border: 1px solid rgba(0, 0, 0, 0.15) !important;

      span {
        line-height: 1;
      }
    }

    .ant-btn-primary {
      background-color: rgba(90, 71, 229, 1) !important;
      color: #ffffff;
      padding: 1px 8px;
    }

    .ant-popover-inner {
      padding: 12px 16px;
      transform: translateX(44px);
    }

    .ant-popconfirm-message-icon {
      svg {
        font-size: 16px;
      }
    }

    .ant-popconfirm-message-title {
      width: 168px;
      height: 36px;
      line-height: 36px;
    }

    .ant-popconfirm-message {
      align-items: center !important;
    }
  }
}

.qa-del-pop {
  .ant-popover-inner {
    padding-top: 20px;
    height: 100px;
  }

  .ant-popconfirm-buttons {
    margin-top: 16px;
  }

  .ant-btn-primary {
    background-color: rgba(90, 71, 229, 1) !important;
    color: #ffffff;
    padding: 1px 8px;
  }

  .ant-btn-sm {
    width: 60px;
  }

  .ant-btn-sm.ant-btn-loading {
    width: auto !important;
  }
}
</style>
