<template>
    <div class="login_container">
        <!-- 登录块 -->
        <div class="login_box">
            <!-- 头像 -->
            <div class="avatar_box">
                <img src="../assets/user.png" alt/>
            </div>
            <!-- 添加表单组件 -->
            <el-form ref="loginFormRef" :rules="loginRules" :model="loginForm" class="login_form" label-width="0">
                <!-- 用户名 -->
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-denglu-copy"></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" prefix-icon="iconfont icon-mima" type="password"></el-input>
                </el-form-item>
                <!-- 按钮 -->
                <el-form-item class="btns">
                    <el-button type="primary" @click="login()">登录</el-button>
                    <el-button type="info" @click="resetLoginForm()">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>
<script>
import Cookie from 'js-cookie'
export default {
    data() {
        return {
            // 表单数据对象
            loginForm: {
                username:"triniti",
                password:"1"
            },
            // 验证对象
            loginRules: {
                // 校验用户名
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' }, //必填项验证
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' } // 长度验证
                ],
                // 校验密码
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' }, //必填项验证
                    { min: 1, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' } // 长度验证
                ],
            },
        }
    },
    methods: {
        // 重置表单内容
        resetLoginForm() {
            this.$refs.loginFormRef.resetFields();
        },
        // 登录方法
        login() {
            // 验证校验规则
            this.$refs.loginFormRef.validate(async valid => {
                if (!valid) return; //验证失败
                const {data:res} = await this.$http.post("login?password="+this.loginForm.password+"&username="+this.loginForm.username); //访问后台
                if (res.status == "0") {
                    window.sessionStorage.setItem('username', res.obj); // 存储user对象,路由守卫会用到
                    this.$message.success("登录成功"); // 信息提示
                    Cookie.set('username', this.loginForm.username);
                    console.log(Cookie.get('username'));
                    if (res.obj.roles == "ROLE_ACTIVITI_ADMIN") {
                        this.$router.push({path: "/home"}); // 管理员页面路由跳转
                    }
                    else if (res.obj.roles == "ROLE_ACTIVITI_USER") {
                        this.$router.push({path: "/userhome"}); // 用户页面路由跳转
                    }
                } else {
                    this.$message.error("登录失败"); // 错误提示
                }
            })
        }
    },
}
</script>
<style lang="less" scoped>
// 根结点样式
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}
.login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    .avatar_box{
        width: 130px;
        height: 130px;
        border: 1px solid #eee;
        border-radius: 50%;
        padding: 5px;
        box-shadow: 0 0 5px #ddd;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        background-color: #eee;
        img{
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-color: #fff;
        }
    }
}
.btns{
    display: flex;
    justify-content: flex-end;
}
.login_form{
    position: absolute;
    bottom: 0%;
    width: 100%;
    padding: 0 10px;
    box-sizing: border-box;
}
</style>
