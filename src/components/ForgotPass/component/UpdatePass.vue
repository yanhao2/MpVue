<template>
    <div class="ForgotPasswordCodeCard">
        <div class="ForgotPasswordCardSize">
            <div class="ForgotPasswordTitle">Enter Verification Code</div>
            <div class="ForgotPasswordText">A verification code has been sent to your registration email if the username exists, please enter the verification code found in the email below: </div>
            <Form ref="formCustom" :model="formCustom" :rules="ruleCustom"
                  @keydown.native.enter.prevent="handleSubmit('formCustom')">
                <FormItem prop="code">
                    <div class="row">
                        <div class="col">
                            <Input v-model="formCustom.code" class="ForgotPasswordInput"/>
                        </div>
                        <div class="ForgotPasswordCodeButtonCode" @click="OnCode()">{{CodeName}}</div>
                    </div>

                </FormItem>
                <FormItem>
                    <div :class="!formCustom.code ? 'ForgotPasswordContentButtonCode' : 'ForgotPasswordContentButtonIsCode'" @click="handleSubmit('formCustom')">
                        Next
                    </div>
                </FormItem>
                <div class="ForgotPasswordCodeRightText" @click="onOpenModel()">
                    <a>Didn’t receive any codes?</a>
                </div>
            </Form>
        </div>
        <div class="Modal" v-show="isModal">
            <div class="ModalSize">
                <div class="ModalContent">
                    <a class="modalClose" @click="onCloseModel()">
                        <img src="/static/Close.png" alt="" style="width: 10px;height: 10px">
                    </a>
                    <div class="ModalTitle">Contact us for account support</div>
                    <div class="ModalContentPadd">
                        <div class="row">
                            <img src="/static/telephone.png" alt="" class="ModalContentTelephone">
                            <div class="col ModalContentTelephoneText">Contact our operation team at <b>662-296-4555</b></div>
                        </div>
                        <div class="row ModalContentEmail">
                            <img src="/static/email.png" alt="" class="ModalContentTelephone">
                            <div class="col ModalContentTelephoneText">Email us at <b>Krungsri_authorization@krungsri.com</b></div>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import api from '../../../api/api'
    export default {
        name: "UpdatePass",
        data () {
            return {
                flag: true,
                isModal: false,
                isCode: false,
                CodeName: 'Resend Code',
                time: 60,
                formCustom: {
                    code: ''
                },
                ruleCustom: {
                    code: [
                        { required: true,  message: 'Invalid code. Try again, or click Resend Code to receive a new code.', trigger: 'blur' }
                    ]
                }
            };
        },
        computed: {
            username () {
                return this.$route.params.username
            },
        },

        methods: {
            onOpenModel () {
                this.isModal = true
            },
            onCloseModel () {
                this.isModal = false
            },
            handleSubmit (name) {
                if (this.formCustom.code) {
                    setTimeout(() => {
                        this.$refs[name].validate((valid) => {
                            if (valid) {
                                this.formCustom.username = this.username
                                this.LoadData(this.formCustom)
                            }
                        })
                    }, 1000)
                }
            },
            OnCode () {
                if (Number(this.username) !== '') {
                    if (Number(this.time) === 60) {
                        this.LoadCodeData();
                        let int = setInterval(() => {
                            this.time --;
                            this.CodeName = 'Resend Code（' + (this.time) + ')';
                            if (this.time === 0) {
                                this.time = 60
                                this.CodeName = 'Resend Code',
                                 clearInterval(int);
                            }
                        }, 1000);
                    }
                }
            },

            async LoadCodeData () {
                try {
                    let data = await api.getCode({'username': this.username});
                    if (Number(data.code) === 200) {
                        this.$message({
                            message: 'A new verification code has been sent to your email.',
                            type: 'success',
                        });
                    } else if (Number(data.code) === 40009){
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                    }  else if (Number(data.code) === 40016) {
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                        this.$store.commit('out');
                        this.$router.push({path: `/mp/login`});
                    } else if (Number(data.code) === 40018) {
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                        this.$store.commit('out');
                        this.$router.push({path: `/mp/login`});
                    } else if (Number(data.code) === 10001) {
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                        this.$store.commit('out');
                        this.$router.push({path: `/mp/login`});
                    } else {
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                    }
                }
                catch (e) {
                    console.log(e);
                }
            },
            async LoadData (record) {
                try {
                    let data = await api.checkCode(record)
                    if (Number(data.code) === 20013) {
                        this.onOpenModel()
                    } else if (Number(data.code) === 200) {
                        this.$router.push({path: `/mp/newPass/${this.username}/${this.formCustom.code}`});
                    } else {
                        this.$message({
                            message: data.msg,
                            type: 'error'
                        });
                    }
                }
                catch (e) {
                    console.log(e);
                }
            }
        },
        created () {
        },

        mounted () {
        },

        updated () {
        }
    }
</script>

<style scoped>
    @import "../../../css/index.css";
    @import "../../../css/reset.css";
</style>

