<template>
  <transition name="fade">
    <div class="shelf-group-dialog-bg" v-if="visible">
      <!-- 移动到... 弹出层 -->
      <div class="shelf-group-dialog-wrapper" v-if="selectGroupDialogVisible">
        <div class="dialog-list">
          <div class="dialog-title-wrapper">
            <span class="dialog-title-text">{{$t('shelf.moveBook')}}</span>
          </div>
          <div class="dialog-list-wrapper">
            <template v-for="(item, index) in categoryList">
              <div class="dialog-list-item"
                :key="index"
                :class="{'is-add': item.edit ? item.edit === 1 : false}"
                @click="onGroupClick(item)"
                v-if="(item.edit === 2 && isInGroup) || item.edit !== 2 || !item.edit">
                <div class="dialog-list-item-text">{{item.title}}</div>
                <div class="dialog-list-icon-wrapper" v-if="category && item.id ? category.id === item.id : false">
                  <span class="icon-check"></span>
                </div>
              </div>
            </template>
          </div>
        </div>
        <div class="dialog-btn-wrapper" @click="hide">
          <div class="dialog-btn">{{$t('shelf.cancel')}}</div>
        </div>
      </div>
      <!-- 新建分组 - 弹出层 -->
      <div class="shelf-group-dialog-wrapper" v-if="newGroupDialogVisible">
        <div class="dialog-list">
          <div class="dialog-title-wrapper">
            <!-- 新建分组 -->
            <span class="dialog-title-text">{{$t('shelf.newGroup')}}</span>
          </div>
          <div class="dialog-input-title-wrapper">
            <!-- 分组名 -->
            <span class="dialog-input-title">{{$t('shelf.groupName')}}</span>
          </div>
          <div class="dialog-input-wrapper">
            <div class="dialog-input-inner-wrapper">
              <input type="text" class="dialog-input" v-model="newGroupName" ref="dialogInput">
              <div class="dialog-input-clear-wrapper" @click="clear" v-show="newGroupName.length > 0">
                <span class="icon-close-circle-fill"></span>
              </div>
            </div>
          </div>
        </div>
        <div class="dialog-btn-wrapper">
          <div class="dialog-btn" @click="hide">{{$t('shelf.cancel')}}</div>
          <div class="dialog-btn" @click="createNewGroup" :class="{'is-empty': newGroupName.length === 0}">{{$t('shelf.confirm')}}</div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    visible: Boolean, // 是否显示
    data: Array, // 书籍列表
    bookList: Array, // 书架信息
    isInGroup: Boolean, // 是否在书籍分组内
    category: Object, // 当前书籍分组信息
    isEditGroupName: { // 是否编辑分组名
      type: Boolean,
      default: false
    }
  },
  computed: {
    // 默认分类
    defaultCategory () {
      return [
        {
          title: this.$t('shelf.newGroup'), // 新建分组
          edit: 1
        },
        {
          title: this.$t('shelf.groupOut'), // 移除分组
          edit: 2
        }
      ]
    },
    // 分类列表
    categoryList () {
      // 过滤书架列表中的书籍分组(item.type: 2)
      const list = this.bookList ? this.bookList.filter(item => item.type === 2) : []
      return [...this.defaultCategory, ...list]
    }
  },
  data () {
    return {
      newGroupDialogVisible: false, // 是否显示新建分组弹窗
      selectGroupDialogVisible: true, // 是的显示选择分组
      newGroupName: '' // 新分组名称
    }
  },
  methods: {
    // 创建新分组
    createNewGroup () {
      if (this.isEditGroupName) {
        this.$emit('editGroupName', this.category, this.newGroupName)
        this.hide()
      } else {
        if (this.newGroupName.length > 0) {
          this.$emit('group', 2, {
            id: this.bookList[this.bookList.length - 2].id + 1,
            itemList: [],
            selected: false,
            title: this.newGroupName,
            type: 2
          })
        }
        this.hide()
      }
    },
    clear () {
      this.newGroupName = ''
    },
    onGroupClick (item) {
      if (item.edit && item.edit === 1) {
        // 新建分组
        this.showCreateGroupDialog()
      } else if (item.edit && item.edit === 2) {
        // 移出分组
        this.$emit('group', 3, item)
        this.hide()
      } else {
        // 移入分组
        this.$emit('group', 1, item)
        this.hide()
      }
    },
    // 显示创建分组弹窗：隐藏选择分组，并将输入框置为空，获取焦点
    showCreateGroupDialog () {
      this.newGroupDialogVisible = true
      this.selectGroupDialogVisible = false
      this.newGroupName = ''
      this.$nextTick(() => {
        this.$refs.dialogInput.focus()
      })
    },
    showEditGroupDialog () {
      this.newGroupDialogVisible = true
      this.selectGroupDialogVisible = false
      this.newGroupName = this.category.title
      this.$nextTick(() => {
        this.$refs.dialogInput.focus()
      })
    },
    show () {
      this.$emit('update:visible', true)
    },
    hide () {
      this.$emit('update:visible', false)
      this.newGroupDialogVisible = false
      this.selectGroupDialogVisible = true
    }
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/styles/global";

  .shelf-group-dialog-bg {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    z-index: 120;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, .2);
    font-size: px2rem(16);
    color: #333;
    @include center;
    .shelf-group-dialog-wrapper {
      position: relative;
      display: flex;
      flex-direction: column;
      width: 60%;
      max-height: 80%;
      background: white;
      border-radius: px2rem(10);
      .dialog-list {
        width: 100%;
        padding: 0 px2rem(20);
        box-sizing: border-box;
        @include scroll;
        .dialog-title-wrapper {
          font-size: px2rem(22);
          font-weight: bold;
          text-align: center;
          padding: px2rem(30) 0 px2rem(20) 0;
        }
        .dialog-list-wrapper {
          font-size: px2rem(14);
          .dialog-list-item {
            display: flex;
            padding: px2rem(15) 0;
            box-sizing: border-box;
            color: #666;
            &.is-add {
              color: $color-blue;
              &:active {
                color: $color-blue-transparent;
              }
            }
            &:active {
              color: rgba(102, 102, 102, .5)
            }
            .dialog-list-item-text {
              flex: 1;
            }
            .dialog-list-icon-wrapper {
              flex: 0 0 px2rem(30);
              color: $color-blue;
              @include right;
            }
          }
        }
      }
      .dialog-btn-wrapper {
        display: flex;
        width: 100%;
        background: $color-blue;
        font-size: px2rem(14);
        font-weight: bold;
        color: white;
        text-align: center;
        padding: px2rem(15) 0;
        box-sizing: border-box;
        border-radius: 0 0 px2rem(10) px2rem(10);
        .dialog-btn {
          flex: 1;
          &.is-empty {
            color: rgba(255, 255, 255, .5);
          }
          &:active {
            color: rgba(255, 255, 255, .5)
          }
        }
      }
      .dialog-input-title-wrapper {
        font-size: px2rem(10);
        margin-top: px2rem(20);
      }
      .dialog-input-wrapper {
        width: 100%;
        padding: 0 0 px2rem(30) 0;
        box-sizing: border-box;
        .dialog-input-inner-wrapper {
          display: flex;
          width: 100%;
          padding: px2rem(10) 0;
          box-sizing: border-box;
          border-bottom: px2rem(1) solid #eee;
          font-size: px2rem(14);
          color: #666;
          .dialog-input {
            flex: 1;
            border: none;
            &:focus {
              outline: none;
            }
          }
          .dialog-input-clear-wrapper {
            flex: 0 0 px2rem(30);
            color: #ccc;
            @include center;
            &:active {
              color: #999;
            }
          }
        }
      }
    }
  }
</style>
