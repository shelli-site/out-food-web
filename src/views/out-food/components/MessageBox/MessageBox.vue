<template>
    <div class="message-box"
         @click="status.popoverShow = !status.popoverShow"
         v-loading="status.websocketConnecting">
        <el-popover
                placement="bottom-end"
                width="300"
                trigger="manual"
                v-model="status.popoverShow">
            <div class="tabs">
                <el-tabs v-model="activeTab" @tab-click="handleTabsClick">
                    <el-tab-pane name="notice">
                        <div slot="label" class="tab-label" style="margin-right: 12px;">
                            通知 {{`(${message.notice.length})`}}
                        </div>
                        <transition name="slide-left">
                            <div v-if="activeTab==='notice'">
                                <template v-if="message.notice.length>0">
                                    <div class="msg-content">
                                        <div v-for="(msg, index) in message.notice" :key="index"
                                             @click="msgRowOnclick(msg)"
                                             class="msg-content-item">
                                            <div class="left">
                                                <el-image
                                                        style="width: 32px; height: 32px;border-radius: 50%;"
                                                        src="https://gw.alipayobjects.com/zos/rmsportal/ThXAXghbEsBCCSDihZxY.png"
                                                        fit="cover"></el-image>
                                            </div>
                                            <div class="right">
                                                <div class="title">{{msg.title}}</div>
                                                <div class="datetime">{{msg.date}}</div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="msg-button">
                                        <div class="clear-msg" @click="()=>{message.notice = []}">清空 通知</div>
                                        <div class="load-more">查看更多</div>
                                    </div>
                                </template>
                                <div v-else class="empty-msg">
                                    <img style="display: inline-block;height: 76px;margin-bottom: 16px;"
                                         src="https://gw.alipayobjects.com/zos/rmsportal/sAuJeJzSKbUmHfBQRzmZ.svg">
                                    <div>你已查看所有通知</div>
                                </div>
                            </div>
                        </transition>
                    </el-tab-pane>
                    <el-tab-pane name="systemMessage">
                        <div slot="label" class="tab-label" style="margin-left: 12px;">
                            系统消息 {{`(${message.systemMessage.length})`}}
                        </div>
                        <transition name="slide-right">
                            <div v-if="activeTab==='systemMessage'">
                                <template v-if="message.systemMessage.length>0">
                                    <div class="msg-content">
                                        <div v-for="(msg, index) in message.systemMessage" :key="index"
                                             @click="msgRowOnclick(msg)"
                                             class="msg-content-item">
                                            <div class="left">
                                                <el-image
                                                        style="width: 32px; height: 32px;border-radius: 50%;"
                                                        src="https://gw.alipayobjects.com/zos/rmsportal/fcHMVNCjPOsbUGdEduuv.jpeg"
                                                        fit="cover"></el-image>
                                            </div>
                                            <div class="right">
                                                <div class="title">{{msg.title}}</div>
                                                <div class="description">{{msg.description}}</div>
                                                <div class="datetime">{{msg.date}}</div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="msg-button">
                                        <div class="clear-msg" @click="()=>{message.systemMessage = []}">清空 系统消息</div>
                                        <div class="load-more">查看更多</div>
                                    </div>
                                </template>
                                <div v-else class="empty-msg">
                                    <img style="display: inline-block;height: 76px;margin-bottom: 16px;"
                                         src="https://gw.alipayobjects.com/zos/rmsportal/sAuJeJzSKbUmHfBQRzmZ.svg">
                                    <div>你已查看所有系统消息</div>
                                </div>
                            </div>
                        </transition>
                    </el-tab-pane>
                </el-tabs>
            </div>
            <el-badge :is-dot="false"
                      :value="(message.systemMessage.length + message.notice.length)||''"
                      slot="reference"
                      :max="99">
                <!--<svg-icon icon-class="gonggao"/> el-icon-bell -->
                <i class="el-icon-message-solid"/>
            </el-badge>
        </el-popover>
    </div>
</template>

<script>
  export default {
    name: 'MessageBox',
    data() {
      return {
        activeTab: 'notice',
        message: {
          notice: [{ title: '你收到了 14 条评论', date: '1分钟前' }],
          systemMessage: [{ title: '老王 评论了订单', description: '这家回锅肉不错！', date: '1分钟前' },
            { title: '小明 评论了订单', description: '妈妈说这家的饭菜干净又健康！', date: '00:15:24' }, {
              title: '刷评论的 评论了订单',
              description: '其实一开始让我评论我是拒绝的，因为你不能让我评就评的吧，第一我要看一下，因为我不愿意评论后加一些特技上去，duang一下，很炫很屌，这样大家一定会骂我评不好。我评了一下，感觉还不错。后来我就叫大家也来评论了，不要加特效。因为我要让大家看到我评论了以后是这样你们评论以后也会是这样。duang！ duang～duang～！',
              date: '3天前'
            }
          ]
        },
        status: {
          websocketConnecting: false,
          popoverShow: false
        }
      }
    },
    created() {
      this.initWebSocket()
    },
    mounted() {
      /*setTimeout(() => {
        this.webSocketSend(JSON.stringify({ msgType: 'INFO', msg: 'Hello World!' }))
      }, 2000)*/
    },
    methods: {
      handleTabsClick(tab) {
        console.log(tab)
      },
      msgRowOnclick(msg) {
        console.log(msg)
      },
      initWebSocket() {
        this.status.websocketConnecting = true
        const wsUri = process.env.VUE_APP_WS_API + '/webSocket/message'
        this.websock = new WebSocket(wsUri)
        this.websock.onopen = () => {
          this.status.websocketConnecting = false
          console.log('连接成功' + this.$store.getters.token)
          // 连接成功发送 token
        }
        // this.websock.open()
        this.websock.onerror = this.webSocketOnError
        this.websock.onmessage = this.webSocketOnMessage
      },
      webSocketOnError(e) {
        this.$notify({
          title: 'WebSocket连接发生错误',
          type: 'error',
          duration: 0
        })
      },
      webSocketOnMessage(e) {
        const data = JSON.parse(e.data)
        /** 读取消息类型 具体分为
         * 1. 获取未读消息个数(初次连接成功)CONNECT
         * 2. 通知信息(APP触发消息)PUSH
         * 3. 自定义消息通知(个性化推送消息)INFO
         * */
        if (data.msgType === 'INFO') {
          this.$notify({
            title: '',
            message: data.msg,
            type: 'success',
            dangerouslyUseHTMLString: true,
            duration: 0
          })
        } else if (data.msgType === 'ERROR') {
          this.$notify({
            title: '',
            message: data.msg,
            dangerouslyUseHTMLString: true,
            type: 'error',
            duration: 0
          })
        }
      },
      webSocketSend(agentData) {
        this.websock.send(agentData)
      }
    }
  }
</script>

<style lang="scss" scoped>
    .message-box:hover {
        background: #d1d3d6 !important;
    }

    .tabs {
        margin: -12px;
    }

    .tab-label {
        width: 100px;
        text-align: center;
    }

    .msg-content {
        .msg-content-item {
            width: 100%;
            padding: 12px 24px;
            display: flex;
            border-bottom: 1px solid #e8e8e8;
            cursor: pointer;

            .left {
                width: 48px;
                padding-top: 4px;
            }

            .right {
                flex: 1;
                color: rgba(0, 0, 0, .45);

                .title {
                    text-align: left;
                    color: rgba(0, 0, 0, .65);
                    font-size: 14px;
                    height: 22px;
                    margin: 0 0 8px;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                }

                .description {
                    font-size: 12px;
                    line-height: 18px;
                    display: -webkit-box;
                    -webkit-box-orient: vertical;
                    -webkit-line-clamp: 2;
                    overflow: hidden;
                }

                .datetime {
                    font-size: 12px;
                    line-height: 18px;
                    margin: 4px 0 0;
                }
            }
        }

        .msg-content-item:hover {
            background-color: #e6f7ff;
        }

        .is-read.msg-content-item:hover {
            background-color: #ffffff;
        }
    }

    .msg-button {
        height: 46px;
        cursor: pointer;
        display: flex;

        .clear-msg, .load-more {
            font-size: 14px;
            flex: 1;
            line-height: 46px;
            text-align: center;
        }

        .load-more {
            border-left: 1px solid #e8e8e8;
        }

    }

    .is-read {
        opacity: .4;
    }

    .empty-msg {
        padding: 73px 0 88px;
        color: rgba(0, 0, 0, .45);
        text-align: center;

        div {
            display: block;
        }
    }
</style>
<style lang="scss">
    .message-box {
        .el-badge {
            .el-badge__content.is-fixed.is-dot {
                top: 10px;
            }

            .el-badge__content.is-fixed {
                top: 13px;
            }
        }
    }

    .el-tabs__nav-wrap::after {
        height: 1px !important;
    }

    .tabs {
        .el-tabs {
            .el-tabs__header {
                margin-bottom: 0px !important;
            }
        }
    }

    #tab-notice {
        padding-left: 17px;
    }


    .slide-left-enter-active, .slide-left-leave-active, .slide-right-enter-active, .slide-right-leave-active {
        transition: all .3s cubic-bezier(.25, .1, .25, 1);
    }

    .slide-left-enter, .slide-left-leave-to {
        transform: translateX(-100%);
        opacity: 0;
    }

    .slide-right-enter, .slide-right-leave-to {
        transform: translateX(100%);
        opacity: 0;
    }

</style>
