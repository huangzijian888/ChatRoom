<template>

    <div id="chat">

        <div class="input-name" v-if="!isConnect">
            昵称：
            <el-input v-model="userName" style="width: 200px;" placeholder="请输入昵称"
                      @keyup.enter.native="preConnected"></el-input>
            <span></span>
        </div>
        <div class="label-name" v-else>
            昵称：{{userName}}
        </div>
        <div class="button-connect">
            <el-button type="primary" v-if="!isConnect" @click="preConnected" plain>连接</el-button>
            <el-button type="danger" v-else @click="disconnect">断开连接</el-button>
        </div>

        <div class="text-message" v-if="isConnect">
            消息：
            <el-input v-model="content" style="width: 400px;" placeholder="请输入内容"
                      @keyup.enter.native="preSendMessage"></el-input>
        </div>
        <div class="button-content" v-if="isConnect">
            <el-button type="primary" @click="preSendMessage" plain>发送消息</el-button>
        </div>
        <el-divider></el-divider>
        <div id="greeting" v-for="item in messages">
            <div>{{item.name+'：'+item.content}}</div>
        </div>
    </div>
</template>

<script>
    import SockJS from 'sockjs-client';
    import Stomp from 'stompjs';
    import URLConfig from '../config/URLConfig.js';

    export default {
        data() {
            return {
                isConnect: false,
                stompClient: null,
                userName: '',
                content: '',
                messages: [],
                fullscreenLoading: false,
                URL: URLConfig
            }
        },
        methods: {
            preConnected() {
                if (this.userName.trim() === '') {
                    this.$notify.error({
                        title: '昵称为空',
                        message: '请输入昵称',
                        position: 'top-left'
                    });
                } else {
                    this.userName = this.userName.trim();
                    this.connect()
                }
            },
            setConnected() {
                this.isConnect = !this.isConnect;
            },

            connect() {
                const socket = new SockJS(this.URL.connectUrl);


                const stompClient = Stomp.over(socket);

                this.stompClient = stompClient;
                stompClient.connect({}, () => {
                    this.setConnected();
                    stompClient.subscribe(this.URL.subscribeUrl, (result) => {
                        this.messages.push(JSON.parse(result.body))
                    });
                }, () => {
                    this.$notify.error({
                        title: '与服务器建立连接失败，请检查连接地址',
                        message: this.URL.connectUrl,
                        position: 'top-left'
                    });
                });


            },
            preSendMessage() {

                if (this.content.trim() == '') {
                    this.$notify.error({
                        title: '消息为空',
                        message: '请输入消息',
                        position: 'top-left'
                    });

                } else {
                    this.content = this.content.trim();
                    this.sendMessage()
                }
            },
            sendMessage() {
                this.stompClient.send(this.URL.sendUrl, {}, JSON.stringify({
                    'name': this.userName,
                    'content': this.content
                }));
                this.content = ''

            },
            disconnect() {
                if (this.stompClient !== null) {
                    this.stompClient.disconnect();
                }
                this.setConnected();
                this.userName = ''
            },
        },
        mounted() {

        },
        beforeDestroy: function () {
            this.disconnect();
        }
    }
</script>

<style lang="scss">
    .input-name,
    .label-name,
    .button-connect,
    .text-message,
    .button-content {
        margin: 20px;
    }
</style>
