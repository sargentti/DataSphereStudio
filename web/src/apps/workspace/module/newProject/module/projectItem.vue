<template>
  <div>
    <slot></slot>
    <Row class="content-item">
      <i-col
        class="project-item project-header"
        :xs="12"
        :sm="8"
        :md="6"
        :lg="5"
        @click.native="addProject">
        <Icon class="add-icon" type="md-add" size="20">
        </Icon>
        <span>{{$t('message.Project.createProject')}}</span>
      </i-col>
      <template  v-if="dataList.length > 0">
        <i-col
          class="project-item"
          :xs="12"
          :sm="8"
          :md="6"
          :lg="5"
          @click.native="goto(currentData, subitem)"
          v-for="(subitem, index) in cachedataList"
          :key="subitem.id"
        >
          <div class="project-main" :ref="`card${index}`">
            <div class="top-bar">
              <span class="project-title" :title="subitem.name">
                <SvgIcon style="font-size: 16px;" color="#5580eb" icon-class="base"/>
                {{subitem.name}}
              </span>
              <div v-if="checkEditable(subitem, getUserName())" class="menu-bar">
                <Button size="small" @click.stop>管理</Button>
                <ul class="menu-list">
                  <li class="list-item" @click.stop="deleteProject(subitem)">删除</li>
                  <li class="list-item" @click.stop="modify(currentData.id, subitem)">配置</li>
                  <!-- <li class="list-item" @click.stop="publish(currentData.id, subitem)">发布</li> -->
                </ul>
              </div>
            </div>
            <div class="mid-bar" :title="subitem.description">
              {{subitem.description || $t('message.workflow.workflowItem.noDesc')}}
            </div>
            <div v-if="subitem.business" class="bottom-bar">
              <span v-for="(item, i) in subitem.business.split(',')" :key="i" :title="item" class="tag-item">{{item}}</span>
            </div>
          </div>
          <Tooltip
            v-if="!hidePublishAndcopy && isPercent(subitem.id)"
            style="width:100%"
            :content="$t('message.workflow.workflowItem.publishing')"
            placement="top"
          >
            <span class="queue-manager-status">

            </span>
          </Tooltip>
        </i-col>

      </template>
      <div class="no-data" v-else>{{$t('message.workflow.workflowItem.nodata')}}</div>
    </Row>
    <Page
      v-if="dataList.length > 0 && pagination.size < dataList.length "
      class="page-bar"
      :total="dataList.length"
      show-sizer
      :current="pagination.current"
      :page-size="pagination.size"
      :page-size-opts="pagination.opts"
      @on-change="pageChange"
      @on-page-size-change="pageSizeChange"
    ></Page>
  </div>
</template>
<script>
import mixin from '@/common/service/mixin';
export default {
  name: "WorkflowContentItem",
  props: {
    dataList: {
      type: Array,
      default: null
    },
    hideButtonBar: {
      type: Boolean,
      default: true
    },
    hidePublishAndcopy: {
      type: Boolean,
      default: true
    },
    currentData: {
      type: Object,
      default() {
        return {};
      }
    },
    readonly: {
      type: Boolean,
      default: false
    },
    precentList: {
      type: Array,
      default: () => []
    },
    tagProp: {
      type: String,
      default: "business"
    },
    applicationAreaMap: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      pagination: {
        size: 10,
        current: 1,
        total: 0,
        opts: [10, 30, 45, 60]
      },
      isToolbarShow: false,
      cardShowNum: 4,
    };
  },
  mixins: [mixin],
  computed: {
    cachedataList() {
      return this.dataList.filter((item, index) => {
        return (
          (this.pagination.current - 1) * this.pagination.size <= index &&
          index < this.pagination.current * this.pagination.size
        );
      });
    },
    pageNumer() {
      return Math.ceil(this.dataList.length / this.pagination.size);
    }
  },
  watch: {
    pageNumer(val) {
      if (val < this.pagination.current && val !== 0) {
        this.pagination.current = val;
      }
    }
  },
  methods: {
    addProject() {
      this.$emit('addProject');
    },
    // 选择操作项
    selectAction(name) {
      console.log(name, 'name')
    },
    checkEditable(item, name) {
      // 先判断是否可编辑
      if (item.editUsers && item.editUsers.length > 0) {
        return item.editUsers.some(e => e === name);
      } else {
        return false;
      }
    },
    modify(classifyId, project) {
      this.$emit("modify", classifyId, project);
    },
    goto(item, subItem) {
      if(this.hidePublishAndcopy && subItem.latestVersion.flowEditLockExist) return this.$Message.warning(this.$t("message.workflow.workflowItem.lockTip"))
      if (this.isPercent(subItem.id))
        return this.$Message.warning(this.$t("message.workflow.workflowItem.noView"));
      this.$emit("goto", item, subItem);
    },
    detail(classifyId, project) {
      this.$emit("detail", classifyId, project);
    },
    deleteProject(project) {
      this.$emit("delete", project);
    },
    copy(classifyId, project) {
      this.$emit("copy", classifyId, project);
    },
    isPercent(id) {
      let flag = false;
      this.precentList.map(item => {
        if (item.id === id) {
          flag = true;
        }
      });
      return flag;
    },
    percentVlaue(id) {
      let value = "";
      this.precentList.map(item => {
        if (item.id === id) {
          value = item.percent;
        }
      });
      return value;
    },
    pageChange(page) {
      this.pagination.current = page;
    },
    pageSizeChange(size) {
      this.pagination.size = size;
    },
    // showResourceView(classifyId, project) {
    //     this.$emit('showResourceView', classifyId, project);
    // },
    handleToolbarShow(item, flag) {
      this.$set(item, "isToolbarShow", flag);
    },
    tagInfo(subitem) {
      if (!Array.isArray(subitem[this.tagProp])) {
        return subitem[this.tagProp];
      }
      return subitem[this.tagProp].join(", ");
    }
  }
};
</script>
<style lang="scss" scoped>
@import '@/common/style/variables.scss';
// 待版本稳定后将card需要重新部样式
.content-item {
    margin: 15px 0px 25px 0px;
    .project-list-ul {
        padding: 10px 20px;
        display: flex;
        justify-content: space-around;
        align-items: center;
        flex-wrap: wrap;
    }
    .project-item {
        height: 142px;
        margin-right: 25px;
        margin-bottom: 25px;
        background: #fff;
        max-width: 319px;
        border-radius: 2px;
        border: 1px solid #dcdee2;
        padding: $padding-25;
        cursor: pointer;
        &:hover {
          box-shadow: 0 2px 12px 0 $shadow-color;
        }
        .project-main {
          height: 100%;
          .top-bar {
            width: 100%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            .project-title {
              flex: 1;
              font-size: $font-size-large;
              font-weight: 600;
              white-space: nowrap;
              text-overflow: ellipsis;
              overflow: hidden;
              font-family: PingFangSC-Medium;
              color: $text-title-color;
              letter-spacing: 0;
            }
            .menu-bar {
              position: relative;
              flex-basis: 40px;
              .menu-list {
                display: none;
                position: absolute;
                bottom: 25px;
                left: -8px;
                border-radius: 4px;
                padding: 5px 0;
                box-shadow: 0 1px 6px rgba(0,0,0,.2);
                z-index: 999;
                background-color: #fff;
                cursor: pointer;
                .list-item {
                  width: 60px;
                  padding: 5px 8px;
                  text-align: center;
                  &:hover {
                    background-color: #f3f3f3;
                  }
                }
              }
              &:hover {
                .menu-list {
                  display: inline;
                }
              }
            }
          }
          .mid-bar {
            width: 100%;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
            font-size: $font-size-14;
            color: rgba(0,0,0,0.5);
            margin: 10px 0;
          }
          .bottom-bar {
            display: flex;
            justify-content: flex-start;
            align-items: center;
            width: 100%;
            .tag-item {
              color: $text-desc-color;
              padding: 2px 10px;
              margin-right: 10px;
              border-radius: 14px;
              background-color:#F3F3F3;
              white-space: nowrap;
              text-overflow: ellipsis;
              overflow: hidden;
              font-family: PingFangSC-Regular;
            }
          }
        }
    }
    .project-header {
      text-align: center;
      font-size: $font-size-large;
      line-height: 102px;
      border: 1px dashed  #dcdee2;
      background: #F8F9FC;
      .add-icon {
        margin-top: -2px;
        margin-right: 5px;
      }
    }
}


.page-bar {
    padding: 0 30px;
    margin-bottom: 30px;
}

.no-data {
    text-align: center;
    padding: 20px;
}

.queue-manager-status {
    position: $relative;
    width: 100%;
    height: 10px;
    display: flex;
    align-items: center;
    font-size: 10px;
    border-radius: 10px;
    background-image: linear-gradient(60deg, transparent 0rem, transparent 0.8rem, rgb(88, 175, 251) 0.8rem, rgb(88, 175, 251) 1.6rem, transparent 1.6rem, transparent 2.4rem, rgb(88, 175, 251) 2.4rem);
    background-size: 21px 27px;
    box-shadow: 1px 1px 5px rgba(88, 175, 251, .6);
    -webkit-animation: process 800ms infinite linear;
    animation: process 800ms infinite linear;
    background-color: $background-color-base;
    &:after {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        height: 100%;
        border-radius: 10px;
        background-image: linear-gradient(to bottom, rgba(88, 175, 251, .4), rgba(88, 175, 251, .4) 15%, transparent 60%, rgba(88, 175, 251, .4));
    }
    .queue-manager-status-busy {
        position: $absolute;
        left: 0;
        background: $success-color;
        height: 100%;
        border-radius: 10px;
        z-index: 1;
    }
    .queue-manager-status-idle {
        background: $success-color;
        height: 100%;
        border-radius: 10px;
        position: $absolute;
        z-index: 0;
    }
    .queue-manager-status-label {
        position: $absolute;
        right: 6px;
        color: $tooltip-color;
    }
}


/* 动画 */

@-webkit-keyframes process {
    0% {
        background-position: 0 0;
    }
    100% {
        background-position: 20px 0;
    }
}

@keyframes process {
    0% {
        background-position: 0 0;
    }
    100% {
        background-position: 20px 0;
    }
}

</style>
