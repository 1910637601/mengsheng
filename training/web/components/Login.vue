<template>
    <div class="loginBody">
        <div class="loginDiv">
            <div class="login-content">
                <h1 class="login-title">运输管理系统</h1>
                <el-form :model="loginForm" label-width="100px" :rules="rules" ref="loginForm">
                    <el-form-item label="账号" prop="no">
                        <el-input type="text" v-model="loginForm.no" autocomplete="off" size="small" />
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input type="password" v-model="loginForm.password" show-password autocomplete="off"
                            size="small" @change="confirm" />
                    </el-form-item>
                    <el-form-item>
                        <div class="button-group">
                            <el-button type="primary" @click="confirm" :disabled="confirm_disable">登录</el-button>
                            <el-button type="danger" @click="register" :disabled="register_disable">注册</el-button>
                        </div>
                    </el-form-item>
                </el-form>
            </div>
        </div>

        <el-dialog v-model="registerDialogVisible" title="注册" width="500px" center :before-close="handleClose">
            <el-form ref="registerform" :rules="registerrules" :model="registerform" label-width="auto">
                <el-form-item label="账号" prop="no">
                    <el-input v-model="registerform.no" />
                </el-form-item>
                <el-form-item label="名字" prop="name">
                    <el-input v-model="registerform.name" />
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="registerform.password" />
                </el-form-item>
                <el-form-item label="年龄" prop="age">
                    <el-input v-model="registerform.age" />
                </el-form-item>
                <el-form-item label="性别">
                    <el-radio-group v-model="registerform.sex">
                        <el-radio value="0">男</el-radio>
                        <el-radio value="1">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="电话" prop="phone">
                    <el-input v-model="registerform.phone" />
                </el-form-item>
            </el-form>
            <template #footer>
                <div class="dialog-footer">
                    <el-button @click="registerDialogVisible = false">取消</el-button>
                    <el-button type="primary" @click="save">确定</el-button>
                </div>
            </template>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: "Login",
    data() {
        let checkAge = (rule, value, callback) => {
            if (value > 150) callback(new Error('年龄输入过大'));
            else callback();
        };
        let checkDuplicate = (rule, value, callback) => {
            if (this.registerform.id) return callback();
            this.$http.get(`/user/findByNo?no=${this.registerform.no}`).then(res => res.data)
                .then(res => res.code != 200 ? callback() : callback(new Error('账号已存在')));
        };
        return {
            registerDialogVisible: false,
            confirm_disable: false,
            register_disable: false,
            loginForm: { no: '', password: '' },
            rules: {
                no: [{ required: true, message: '请输入账号', trigger: 'blur' }],
                password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
            },
            registerrules: {
                no: [
                    { required: true, message: '请输入账号', trigger: 'blur' },
                    { min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' },
                    { validator: checkDuplicate, trigger: 'blur' }
                ],
                name: [{ required: true, message: '请输入名字', trigger: 'blur' }],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }
                ],
                age: [
                    { required: true, message: '请输入年龄', trigger: 'blur' },
                    { min: 1, max: 3, message: '长度在 1 到 3 位', trigger: 'blur' },
                    { pattern: /^([1-9][0-9]*){1,3}$/, message: '年龄必须为正整数', trigger: "blur" },
                    { validator: checkAge, trigger: 'blur' }
                ],
                phone: [
                    { required: true, message: '手机号不能为空', trigger: "blur" },
                    { pattern: /^1[3-9]\d{9}$/, message: "请输入正确的手机号", trigger: "blur" }
                ]
            },
            registerform: {
                id: '',
                no: '',
                name: '',
                password: '',
                age: '',
                phone: '',
                sex: '0',
                roleId: '1'
            }
        }
    },
    methods: {
        handleClose(done) {
            this.$confirm('确认关闭？').then(done).catch(() => {});
        },
        resetForm() {
            this.$refs.registerform.resetFields();
        },
        save() {
            this.$refs.registerform.validate(valid => {
                if (valid) {
                    this.$http.post('user/save', this.registerform).then(res => res.data)
                        .then(res => {
                            if (res.code == 200) {
                                this.$message({ message: '注册成功!', type: 'success' });
                                this.registerDialogVisible = false;
                            } else {
                                this.$message({ message: '操作失败!', type: 'error' });
                            }
                        });
                } else {
                    this.$message({ message: '校验失败 请填写完整数据', type: 'error' });
                    this.register_disable = false;
                }
            });
        },
        register() {
            this.registerDialogVisible = true;
            this.$nextTick(() => this.resetForm());
        },
        confirm() {
            this.confirm_disable = true;
            this.$refs.loginForm.validate(valid => {
                if (valid) {
                    this.$http.post('user/login', this.loginForm).then(res => res.data)
                        .then(res => {
                            if (res.code == 200) {
                                this.$message({ message: '登陆成功!', type: 'success' });
                                sessionStorage.setItem("LoginUser", JSON.stringify(res.data.user));
                                this.$store.commit("setMenu", res.data.menu);
                                this.$router.replace('/Indextest');
                            } else {
                                this.$message({ message: '校验失败 用户名或密码错误!', type: 'error' });
                                this.confirm_disable = false;
                            }
                        });
                } else {
                    this.$message({ message: '校验失败 请输入合法的用户名或密码', type: 'error' });
                    this.confirm_disable = false;
                }
            });
        }
    }
}
</script>

<style scoped>
.loginBody {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #B3C0D1 url("../assets/picture/huoyun.jpg") no-repeat center center;
    background-size: cover;
}

.loginDiv {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 450px;
    height: 330px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.login-title {
    margin: 20px 0;
    text-align: center;
    font-size: 24px;
    color: #333;
}

.login-content {
    width: 400px;
}

.button-group {
    display: flex;
    justify-content: space-between;
}

.dialog-footer {
    display: flex;
    justify-content: flex-end;
}
</style>
