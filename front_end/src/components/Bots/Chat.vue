<template>
  <div class="bots-chat-container">
    <div class="my-page">
      <div class="header">
        <img src="@/assets/bots/bot-avatar.png" alt="avatar" />
        {{ botInfo.bot_name }}
      </div>
      <div id="chat" class="chat">
        <ul id="chat-ul" ref="scrollDom">
          <div class="ai">
            <div class="content">
              <img class="avatar" src="@/assets/home/ai-avatar.png" alt="头像" />
              <p class="question-text" v-html="botInfo.welcome_message"></p>
            </div>
          </div>
          <li v-for="(item, index) in QA_List" :key="index">
            <div v-if="item.type === 'user'" class="user">
              <img class="avatar" src="@/assets/home/avatar.png" alt="头像" />
              <p class="question-text">{{ item.question }}</p>
            </div>
            <div v-else class="ai">
              <div class="content">
                <img class="avatar" src="@/assets/home/ai-avatar.png" alt="头像" />
                <p
                  v-if="!item.onlySearch"
                  class="question-text"
                  :class="[
                    !item.source.length && !item?.picList?.length ? 'change-radius' : '',
                    item.showTools ? '' : 'flashing',
                  ]"
                >
                  <HighLightMarkDown v-if="item.answer" :content="item.answer" />
                  <span v-else>{{ item.answer }}</span>
                  <ChatInfoPanel
                    v-if="Object.keys(item?.itemInfo?.tokenInfo || {}).length"
                    :chat-item-info="item.itemInfo"
                  />
                </p>
              </div>
              <template v-if="item?.picList?.length">
                <div
                  v-for="(picItem, picIndex) in item.picList"
                  :key="picItem + picIndex"
                  :class="[
                    'data-picList',
                    !item.source.length && picIndex + 1 === item.picList.length
                      ? 'picList-radius'
                      : '',
                  ]"
                >
                  <a-image :width="150" :src="picItem" class="responsive-image" />
                </div>
              </template>
              <template v-if="item.source.length">
                <div
                  :class="[
                    'source-total',
                    !showSourceIdxs.includes(index) ? 'source-total-last' : '',
                  ]"
                >
                  <span v-if="language === 'zh'">找到了{{ item.source.length }}个信息来源：</span>
                  <span v-else>Found {{ item.source.length }} source of information</span>
                  <SvgIcon
                    v-show="!showSourceIdxs.includes(index)"
                    name="down"
                    @click="showSourceList(index)"
                  />
                  <SvgIcon
                    v-show="showSourceIdxs.includes(index)"
                    name="up"
                    @click="hideSourceList(index)"
                  />
                </div>
                <div v-show="showSourceIdxs.includes(index)" class="source-list">
                  <div
                    v-for="(sourceItem, sourceIndex) in item.source"
                    :key="sourceIndex"
                    class="data-source"
                  >
                    <p v-show="sourceItem.file_name" class="control">
                      <span class="tips">{{ common.dataSource }}{{ sourceIndex + 1 }}:</span>
                      <a
                        v-if="sourceItem.file_id.startsWith('http')"
                        :href="sourceItem.file_id"
                        target="_blank"
                      >
                        {{ sourceItem.file_name }}
                      </a>
                      <span
                        v-else
                        :class="[
                          'file',
                          checkFileType(sourceItem.file_name) ? 'filename-active' : '',
                        ]"
                        @click="handleChatSource(sourceItem)"
                      >
                        {{ sourceItem.file_name }}
                      </span>
                      <SvgIcon
                        v-show="sourceItem.showDetailDataSource"
                        name="iconup"
                        @click="hideDetail(item, sourceIndex)"
                      />
                      <SvgIcon
                        v-show="!sourceItem.showDetailDataSource"
                        name="icondown"
                        @click="showDetail(item, sourceIndex)"
                      />
                    </p>
                    <Transition name="sourceitem">
                      <div v-show="sourceItem.showDetailDataSource" class="source-content">
                        <p v-html="sourceItem.content?.replaceAll('\n', '<br/>')"></p>
                        <p class="score">
                          <span class="tips">{{ common.correlation }}</span
                          >{{ sourceItem.score }}
                        </p>
                      </div>
                    </Transition>
                  </div>
                </div>
              </template>
              <div v-if="item.showTools" class="feed-back">
                <div class="reload-box" @click="reAnswer(item)">
                  <SvgIcon name="reload"></SvgIcon>
                  <span class="reload-text">{{ common.regenerate }}</span>
                </div>
                <div class="tools">
                  <SvgIcon
                    :style="{
                      color: item.copied ? '#4D71FF' : '',
                    }"
                    name="copy"
                    @click="myCopy(item)"
                  ></SvgIcon>
                  <SvgIcon
                    :style="{
                      color: item.like ? '#4D71FF' : '',
                    }"
                    name="like"
                    @click="like(item, $event)"
                  ></SvgIcon>
                  <SvgIcon
                    :style="{
                      color: item.unlike ? '#4D71FF' : '',
                    }"
                    name="unlike"
                    @click="unlike(item)"
                  ></SvgIcon>
                </div>
              </div>
            </div>
          </li>
          <div v-show="showLoading" class="stop-placeholder"></div>
          <div v-show="showLoading" ref="stopBtn" class="stop-btn">
            <a-button @click="stopChat">
              <template #icon>
                <SvgIcon name="stop" :class="showLoading ? 'loading' : ''"></SvgIcon>
              </template>
              {{ common.stop }}
            </a-button>
          </div>
        </ul>
      </div>
      <div class="question-box">
        <div class="question">
          <!--          <a-popover placement="topLeft">-->
          <!--            <template #content>-->
          <!--              <p v-if="network">退出联网检索</p>-->
          <!--              <p v-else>开启联网检索</p>-->
          <!--            </template>-->
          <!--            <span :class="['network', `network-${network}`]">-->
          <!--              <SvgIcon name="network" @click="networkChat" />-->
          <!--            </span>-->
          <!--          </a-popover>-->
          <!--          <a-popover v-if="chatType === 'share'" placement="topLeft">-->
          <!--            <template #content>-->
          <!--              <p v-if="control">{{ bots.multiTurnConversation2 }}</p>-->
          <!--              <p v-else>{{ bots.multiTurnConversation1 }}</p>-->
          <!--            </template>-->
          <!--            <span :class="['control', `control-${control}`]">-->
          <!--              <SvgIcon name="chat-control" @click="controlChat" />-->
          <!--            </span>-->
          <!--          </a-popover>-->

          <!--          <span v-if="chatType === 'share'" class="download" @click="downloadChat">-->
          <!--            <SvgIcon name="chat-download" />-->
          <!--          </span>-->
          <ChatTextarea v-model:input-value="question" :options="mentionOptions" @send="send">
            <a-button type="primary" :disabled="showLoading" @click="send">
              <SvgIcon name="sendplane"></SvgIcon>
            </a-button>
          </ChatTextarea>
          <!--          <a-input-->
          <!--            v-model:value="question"-->
          <!--            max-length="200"-->
          <!--            :placeholder="common.problemPlaceholder"-->
          <!--            @keyup.enter="send"-->
          <!--          >-->
          <!--            <template #suffix>-->
          <!--              <div class="send-plane">-->
          <!--                <a-button type="primary" :disabled="showLoading" @click="send">-->
          <!--                  <SvgIcon name="sendplane"></SvgIcon>-->
          <!--                </a-button>-->
          <!--              </div>-->
          <!--            </template>-->
          <!--          </a-input>-->
        </div>
      </div>
    </div>
    <div v-if="!botInfo.kb_ids || !botInfo.kb_ids.length" class="mask">
      <img src="@/assets/bots/lock.png" alt="icon" />
      <p>{{ bots.bindKbtoPreview }}</p>
    </div>
    <ChatSettingDialog ref="chatSettingForDialogRef" />
  </div>
  <DefaultModal :content="content" :confirm-loading="confirmLoading" @ok="confirm" />
</template>
<script lang="ts" setup>
import { apiBase } from '@/services';
import { IChatItem } from '@/utils/types';
import { useClipboard } from '@vueuse/core';
import { message } from 'ant-design-vue';
import SvgIcon from '../SvgIcon.vue';
import { fetchEventSource } from '@microsoft/fetch-event-source';
import { useBotsChat } from '@/store/useBotsChat';
import { useChat } from '@/store/useChat';
import { useChatSource } from '@/store/useChatSource';
import { Typewriter } from '@/utils/typewriter';
import DefaultModal from '../DefaultModal.vue';
import html2canvas from 'html2canvas';
import { getLanguage } from '@/language/index';
import { useLanguage } from '@/store/useLanguage';
import { userId, userPhone } from '@/services/urlConfig';
import urlResquest from '@/services/urlConfig';
import { ChatInfoClass, resultControl, throttle } from '@/utils/utils';
import { useChatSetting } from '@/store/useChatSetting';
import ChatInfoPanel from '@/components/ChatInfoPanel.vue';
import HighLightMarkDown from '@/components/HighLightMarkDown.vue';
import ChatSettingDialog from '@/components/ChatSettingDialog.vue';
import ChatTextarea from '@/components/ChatTextarea.vue';

const props = defineProps({
  chatType: {
    type: String,
    default: 'edit',
  },
  botInfo: {
    type: Object as any,
    default: () => {},
  },
});

const common = getLanguage().common;
const bots = getLanguage().bots;

const typewriter = new Typewriter((str: string) => {
  if (str) {
    QA_List.value[QA_List.value.length - 1].answer += str || '';
  }
});

const { QA_List } = storeToRefs(useBotsChat());
const { chatSettingFormActive } = storeToRefs(useChatSetting());
const { copy } = useClipboard();
const { setChatSourceVisible, setSourceType, setSourceUrl, setTextContent } = useChatSource();
const { language } = storeToRefs(useLanguage());

//当前问的问题
const question = ref('');

//问答的上下文
const history = computed(() => {
  const context = chatSettingFormActive.value.context;
  if (context === 0) return [];
  const usefulChat = QA_List.value.filter(item => item.type === 'ai');
  const historyChat = context === 11 ? usefulChat : usefulChat.slice(-context);
  return historyChat.map(item => [item.question, item.answer]);
});

//当前是否回答中
const showLoading = ref(false);

const showSourceIdxs = ref([]);

//取消请求用
let ctrl: AbortController;

const scrollDom = ref(null);
const stopBtn = ref(null);

const scrollBottom = () => {
  nextTick(() => {
    scrollDom.value?.scrollIntoView(false);
  });
};

const like = throttle((item, e) => {
  item.like = !item.like;
  item.unlike = false;
  if (item.like) {
    e.target.parentNode.style.animation = 'shake ease-in .5s';
    const timer = setTimeout(() => {
      clearTimeout(timer);
      e.target.parentNode.style.animation = '';
    }, 600);
  }
}, 800);
const unlike = (item: IChatItem) => {
  item.unlike = !item.unlike;
  item.like = false;
};

//拷贝
const myCopy = (item: IChatItem) => {
  copy(item.answer)
    .then(() => {
      item.copied = !item.copied;
      message.success(common.copySuccess, 1);
      const timer = setTimeout(() => {
        clearTimeout(timer);
        item.copied = !item.copied;
      }, 1000);
    })
    .catch(() => {
      message.error(common.copyFailed, 1);
    });
};

const addQuestion = q => {
  QA_List.value.push({
    question: q,
    type: 'user',
  });
  scrollBottom();
};

const addAnswer = (question: string) => {
  QA_List.value.push({
    answer: '',
    question,
    onlySearch: chatSettingFormActive.value.capabilities.onlySearch,
    type: 'ai',
    copied: false,
    like: false,
    unlike: false,
    source: [],
    picList: null,
    showTools: false,
  });
};

const chatInfoClass = new ChatInfoClass();

const stopChat = () => {
  if (ctrl) {
    ctrl.abort();
  }
  typewriter.done();
  showLoading.value = false;
  QA_List.value[QA_List.value.length - 1].showTools = true;
};

// Mention 的 配置项
const mentionOptions = ref<string[]>([]);
const getMentionOptions = async () => {
  const res: any = await resultControl(
    await urlResquest.getTags({
      kb_ids: props.botInfo.kb_ids,
    })
  );
  mentionOptions.value = res.tags;
};
onMounted(() => {
  getMentionOptions();
});
watch(
  () => props.botInfo,
  () => {
    getMentionOptions();
  },
  {
    immediate: true,
    deep: true,
  }
);
// 统计几个 @ 超过10个报错
const computedCallNumber = (question: string) => {
  const atCount = (question.match(/@/g) || []).length;
  return atCount <= 10;
};

//发送问答消息
const send = async () => {
  if (!question.value.trim().length) {
    return;
  }
  if (!question.value.length) {
    message.warn('正在聊天中...请等待结束');
    return;
  }
  if (!(await checkChatSetting())) {
    message.error('模型设置错误，请先检查模型配置');
    return;
  }
  if (!computedCallNumber(question.value)) {
    message.error('不可@超过10个');
    return;
  }

  const q = question.value;
  question.value = '';
  addQuestion(q);
  // if (history.value.length >= 3) {
  //   history.value = [];
  // }
  showLoading.value = true;
  scrollDom.value?.scrollIntoView(true);
  ctrl = new AbortController();

  const headers = {
    'Content-Type': 'application/json',
    Accept: ['text/event-stream', 'application/json'],
    'Transfer-Encoding': 'chunked',
    Connection: 'keep-alive',
  };

  fetchEventSource(apiBase + '/local_doc_qa/local_doc_chat', {
    method: 'POST',
    headers: headers,
    openWhenHidden: true,
    body: JSON.stringify({
      user_id: userId,
      user_info: userPhone,
      bot_id: props.botInfo.bot_id,
      history: history.value,
      question: q,
      streaming: chatSettingFormActive.value.capabilities.onlySearch === false,
      // networking: chatSettingFormActive.value.capabilities.networkSearch,
      product_source: 'saas',
      // rerank: chatSettingFormActive.value.capabilities.rerank,
      // only_need_search_results: chatSettingFormActive.value.capabilities.onlySearch,
      // hybrid_search: chatSettingFormActive.value.capabilities.mixedSearch,
      // max_token: chatSettingFormActive.value.maxToken,
      // api_base: chatSettingFormActive.value.apiBase,
      // api_key: chatSettingFormActive.value.apiKey,
      // model: chatSettingFormActive.value.apiModelName,
      // api_context_length: chatSettingFormActive.value.apiContextLength,
      // chunk_size: chatSettingFormActive.value.chunkSize,
      // top_p: chatSettingFormActive.value.top_P,
      // top_k: chatSettingFormActive.value.top_K,
      // temperature: chatSettingFormActive.value.temperature,
    }),
    signal: ctrl.signal,
    onopen(e: any) {
      console.log('open', e);
      addAnswer(q);
      if (e.ok && e.headers.get('content-type') === 'text/event-stream') {
        // 模型配置添加进去
        chatInfoClass.addChatSetting(chatSettingFormActive.value);
        typewriter.start();
      } else if (e.headers.get('content-type') === 'application/json') {
        typewriter.add('Error 请检查模型是否配置正确');
      }
    },
    onmessage(msg: { data: string }) {
      console.log('message');
      const res: any = JSON.parse(msg.data);
      console.log(res);
      if (res?.code == 200 && res?.response && res.msg === 'success') {
        // QA_List.value[QA_List.value.length - 1].answer += res.result.response;
        typewriter.add(res?.response.replaceAll('\n', '<br/>'));
        scrollBottom();
      } else {
        const timeObj = res.time_record.time_usage;
        delete timeObj['retriever_search_by_milvus'];
        chatInfoClass.addTime(res.time_record.time_usage);
        chatInfoClass.addToken(res.time_record.token_usage);
        chatInfoClass.addDate(Date.now());
      }

      if (res?.source_documents?.length) {
        QA_List.value[QA_List.value.length - 1].source = res?.source_documents;
      }

      // if (res?.history.length) {
      //   history.value = res?.history;
      // }
    },
    onclose(e: any) {
      console.log('close', e);
      typewriter.done();
      ctrl.abort();
      showLoading.value = false;
      QA_List.value[QA_List.value.length - 1].showTools = true;
      // 将chat info添加进回答中
      QA_List.value.at(-1).itemInfo = chatInfoClass.getChatInfo();
      nextTick(() => {
        scrollBottom();
      });
    },
    onerror(err: any) {
      console.log('error', err);
      typewriter.done();
      ctrl.abort();
      showLoading.value = false;
      QA_List.value[QA_List.value.length - 1].showTools = true;

      nextTick(() => {
        scrollBottom();
      });
      throw err;
    },
  });
};

const reAnswer = (item: IChatItem) => {
  console.log('reAnswer');
  question.value = item.question;
  send();
};

const hideDetail = (item: IChatItem, index) => {
  item.source[index].showDetailDataSource = false;
};

//点击查看是否显示详细来源
const showDetail = (item: IChatItem, index) => {
  item.source[index].showDetailDataSource = !item.source[index].showDetailDataSource;
};

const showSourceList = index => {
  showSourceIdxs.value.push(index);
};

const hideSourceList = index => {
  showSourceIdxs.value = showSourceIdxs.value.filter(item => item !== index);
};

//下载 清除聊天记录相关
const { showModal } = storeToRefs(useChat());
const { clearQAList } = useBotsChat();
const confirmLoading = ref(false);
const content = ref('');
const type = ref('');
// const controlChat = () => {
//   // control.value = !control.value;
// };

// const downloadChat = () => {
//   type.value = 'download';
//   showModal.value = true;
//   content.value = common.saveTip;
// };

const confirm = async () => {
  confirmLoading.value = true;
  if (type.value === 'download') {
    console.log('download');
    try {
      const ele = document.getElementById('chat-ul');
      const canvas = await html2canvas(ele as HTMLDivElement, {
        useCORS: true,
      });
      const imgUrl = canvas.toDataURL('image/png');
      const tempLink = document.createElement('a');
      tempLink.style.display = 'none';
      tempLink.href = imgUrl;
      tempLink.setAttribute('download', 'chat-shot.png');
      if (typeof tempLink.download === 'undefined') tempLink.setAttribute('target', '_blank');

      document.body.appendChild(tempLink);
      tempLink.click();
      document.body.removeChild(tempLink);
      window.URL.revokeObjectURL(imgUrl);
      message.success(bots.downloadSuccessful);
      Promise.resolve();
    } catch (e) {
      console.log(e);
      message.error(e.message || e.msg || '出错了');
    }
  } else if (type.value === 'delete') {
    console.log('delete');
    // history.value = [];
    clearQAList();
  }
  type.value = '';
  content.value = '';
  confirmLoading.value = false;
  showModal.value = false;
};

// 模型配置是否正确
const checkSettingOk = inject('checkSettingOk') as Function;
const checkChatSetting = async () => {
  return await checkSettingOk();
};

// 检查信息来源的文件是否支持窗口化渲染
let supportSourceTypes = [
  'md',
  'txt',
  'pdf',
  'jpg',
  'png',
  'jpeg',
  'doc',
  'docx',
  'xls',
  'xlsx',
  'ppt',
  'pptx',
  'jsonl',
  'csv',
  'eml',
];
const checkFileType = filename => {
  if (!filename) {
    return false;
  }
  const arr = filename.split('.');
  if (arr.length) {
    const suffix = arr.pop();
    if (supportSourceTypes.includes(suffix)) {
      return true;
    } else {
      return false;
    }
  } else {
    return false;
  }
};

const handleChatSource = file => {
  console.log('handleChatSource', file);
  const isSupport = checkFileType(file.file_name);
  if (isSupport) {
    queryFile(file);
  }
};

async function queryFile(file) {
  try {
    setSourceUrl(null);
    const res: any = await resultControl(await urlResquest.getFile({ file_id: file.file_id }));
    console.log('queryFile', res);
    const suffix = file.file_name.split('.').pop();
    const b64Type = getB64Type(suffix);
    console.log('b64Type', b64Type);
    setSourceType(suffix);
    setSourceUrl(`data:${b64Type};base64,${res.base64_content}`);
    if (suffix === 'txt') {
      const decodedTxt = atob(res.base64_content);
      const correctStr = decodeURIComponent(escape(decodedTxt));
      console.log('decodedTxt', correctStr);
      setTextContent(correctStr);
      setChatSourceVisible(true);
    } else {
      setChatSourceVisible(true);
    }
  } catch (e) {
    message.error(e.msg || '获取文件失败');
  }
}

let b64Types = [
  'text/markdown', // md
  'text/plain', // txt
  'application/pdf', // pdf
  'image/jpeg', // jpg
  'image/png', // png
  'image/jpeg', // jpeg
  'application/msword', // doc
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document', // docx
  'application/vnd.ms-excel', // xls
  'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', // xlsx
  'application/vnd.ms-powerpoint', // ppt
  'application/vnd.openxmlformats-officedocument.presentationml.presentation', // pptx
  'application/jsonl', // jsonl
  'text/csv', // csv
  'message/rfc822', // eml
];

function getB64Type(suffix) {
  const index = supportSourceTypes.indexOf(suffix);
  return b64Types[index];
}

// const networkChat = () => {
//   network.value = !network.value;
// };

scrollBottom();
</script>

<style lang="scss" scoped>
.bots-chat-container {
  width: 100%;
  height: calc(100% - 22px);
  // padding-top: 16px;
  border-radius: 12px;
  background: #fff;
  font-family: PingFang SC;
  position: relative;
  // background-color: #26293b;
}

.my-page {
  position: relative;
  height: 100%;
  display: flex;
  flex-direction: column;
  margin: 0 auto;
  // border-radius: 12px 0 0 0;
  // background: #f3f6fd;
}

.header {
  width: 100%;
  height: 52px;
  font-size: 14px;
  font-weight: 500;
  color: #222222;
  border-top-right-radius: 12px;
  border-top-left-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-bottom: 1px solid #ededed;

  img {
    width: 32px;
    height: 32px;
    margin-right: 8px;
  }
}

.chat {
  margin: 0 auto;
  width: calc(90% - 52px);
  // min-width: 900px;
  max-width: 1239px;
  flex: 1;
  // height: calc(100vh - 64px - 22px - 66px - 52px - 48px - 80px);
  overflow-x: hidden;
  overflow-y: auto;
  padding-top: 28px;

  #chat-ul {
    // background: #f3f6fd;
    position: relative;
  }

  .avatar {
    width: 32px;
    height: 32px;
    margin-right: 16px;
  }

  .user {
    display: flex;
    margin-bottom: 16px;

    .question-text {
      padding: 13px 20px;
      font-size: 14px;
      font-weight: normal;
      line-height: 22px;
      color: #222222;
      background: #e9e1ff;
      border-radius: 12px;
      word-wrap: break-word;
    }
  }

  .ai {
    margin: 16px 0 28px 0;

    .content {
      display: flex;

      .question-text {
        flex: 1;
        padding: 13px 20px;
        font-size: 14px;
        font-weight: normal;
        line-height: 22px;
        color: $title1;
        background: #f9f9fc;
        border-radius: 12px 12px 0 0;
        word-wrap: break-word;
      }

      .flashing {
        &:after {
          -webkit-animation: blink 1s steps(5, start) infinite;
          animation: blink 1s steps(5, start) infinite;
          content: '▋';
          margin-left: 0.25rem;
          vertical-align: baseline;
        }
      }

      .change-radius {
        border-radius: 12px;
      }
    }

    .source-total {
      padding: 13px 20px;
      margin-left: 48px;
      background: #f9f9fc;
      display: flex;
      align-items: center;

      span {
        margin-right: 5px;
      }

      svg {
        width: 16px !important;
        height: 16px !important;
        cursor: pointer !important;
      }
    }

    .source-total-last {
      border-radius: 0px 0 12px 12px;
    }

    .source-list {
      margin-left: 48px;
      background: #f9f9fc;
      border-radius: 0px 12px 12px 12px;
    }

    .data-source {
      padding: 13px 20px;
      font-size: 14px;
      line-height: 22px;
      color: $title1;

      &:nth-last-of-type(2) {
        border-radius: 0px 0px 12px 12px;
      }

      &:nth-first-of-type(1) {
        border-radius: 0px 12px 12px 12px;
      }

      .control {
        width: 100%;
        overflow-wrap: break-word;
        display: flex;
        align-items: center;
        flex-wrap: wrap;

        .file {
          max-width: 100%;
          word-wrap: break-word;
          overflow-wrap: break-word;
        }
      }

      .score {
        margin-top: 26px;
      }

      .source-content {
        margin-top: 26px;
      }

      .tips {
        height: 22px;
        line-height: 22px;
        color: $title2;
        margin-right: 8px;
      }

      .file {
        color: $baseColor;
        margin-right: 8px;
      }

      .filename-active {
        color: #5a47e5;
        text-decoration: underline;
        cursor: pointer;
      }

      svg {
        width: 14px;
        height: 14px;
        color: $baseColor;
        cursor: pointer;
      }

      a {
        color: #5a47e5;
        text-decoration: underline;
        cursor: pointer;
      }
    }

    .data-picList {
      margin-left: 48px;
      background: #f9f9fc;
      padding: 10px 20px;
    }

    .picList-radius {
      border-radius: 0 0 12px 12px;
    }

    .feed-back {
      display: flex;
      height: 20px;
      margin-top: 8px;
      margin-left: 48px;

      .reload-box {
        display: flex;
        cursor: pointer;
        align-items: center;
        margin-right: auto;
        color: #5a47e5;

        .reload-text {
          height: 22px;
          line-height: 22px;
        }
      }

      .tools {
        display: flex;
        align-items: center;

        svg {
          margin-left: 16px;
        }
      }

      svg {
        width: 16px !important;
        height: 16px !important;
        cursor: pointer !important;
      }
    }
  }
}

.stop-btn {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px 0;
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);

  :deep(.ant-btn) {
    width: 92px;
    height: 32px;
    border: 1px solid #e2e2e2;
    color: $title2;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  svg {
    width: 12px;
    height: 12px;
    margin-right: 4px;
  }

  .loading {
    animation: loading 3s infinite;
  }
}

.stop-placeholder {
  width: 100%;
  height: 52px;
}

.question-box {
  // position: absolute;
  // bottom: 28px;
  // left: 280px;
  width: 100%;
  margin-top: 10px;

  .question {
    position: relative;
    width: 90%;
    // min-width: 900px;
    //max-width: 1239px;
    //height: 48px;
    margin: 0 auto;
    display: flex;
    align-items: center;

    .download,
    .delete,
    .network {
      cursor: pointer;
      padding: 8px;
      display: flex;
      margin-right: 16px;
      border-radius: 8px;
      background: #ffffff;
      border: 1px solid #e5e5e5;
      color: #666666;

      &:hover {
        border: 1px solid #5a47e5;
        color: #5a47e5;
      }

      svg {
        width: 24px;
        height: 24px;
      }

      &.network-true {
        border: 1px solid #5a47e5;
        color: #5a47e5;
      }

      &.network-false {
        border: 1px solid #e5e5e5;
        color: #666666;
      }
    }

    .send-plane {
      width: 56px;
      height: 36px;
      border-radius: 8px;
      color: #fff;
      background: #5a47e5;

      :deep(.ant-btn-primary) {
        height: 100%;
        display: flex;
        align-items: center;
        background: linear-gradient(300deg, #7b5ef2 1%, #c383fe 97%);
      }

      :deep(.ant-btn-primary:disabled) {
        background: linear-gradient(300deg, #7b5ef2 1%, #c383fe 97%);
        color: #fff !important;
        border-color: transparent !important;
      }

      svg {
        width: 24px;
        height: 24px;
      }
    }

    :deep(.ant-input-affix-wrapper) {
      width: 100%;
      max-width: 1108px;
      border-color: #e5e5e5;
      box-shadow: none !important;

      &:hover,
      &:focus,
      &:active {
        border-color: #5a47e5 !important;
        box-shadow: none !important;
      }
    }

    :deep(.ant-input:hover) {
      border-color: $baseColor;
    }

    :deep(.ant-input:focus) {
      border-color: $baseColor;
    }

    :deep(.ant-input-affix-wrapper) {
      padding: 4px 4px 4px 11px;
    }
  }
}

.mask {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.2);
  display: flex;
  color: #fff;
  font-size: 16px;
  border-radius: 12px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 0;
  left: 0;

  img {
    width: 40px;
    height: 40px;
    margin-bottom: 10px;
  }

  p {
    padding: 0 40px;
  }
}

.sourceitem-leave, // 离开前,进入后透明度是1
.sourceitem-enter-to {
  opacity: 1;
}

.sourceitem-leave-active,
.sourceitem-enter-active {
  transition: opacity 0.5s; //过度是.5s秒
}

.sourceitem-leave-to,
.sourceitem-enter {
  opacity: 0;
}
</style>
<style lang="scss">
@keyframes shake {
  0% {
    transform: rotate(0deg);
  }

  10% {
    transform: rotate(10deg);
  }

  20% {
    transform: rotate(20deg);
  }
  30% {
    transform: rotate(20deg);
  }
  40% {
    transform: rotate(20deg);
  }

  50% {
    transform: rotate(15deg);
  }

  60% {
    transform: rotate(0deg);
  }
  70% {
    transform: rotate(-15deg);
  }
  80% {
    transform: rotate(-30deg);
  }
  90% {
    transform: rotate(-15deg);
  }

  100% {
    transform: rotate(0deg);
  }
}

@keyframes blink {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

@keyframes loading {
  0% {
    transform: rotate(0deg);
  }

  25% {
    transform: rotate(90deg);
  }
  50% {
    transform: rotate(180deg);
  }
  75% {
    transform: rotate(270deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
