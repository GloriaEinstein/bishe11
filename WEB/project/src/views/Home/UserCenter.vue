<template>
  <div class="user-center">
    <el-card>
      <h2>个人中心</h2>
      <el-divider />

      <div class="avatar-section" v-if="userInfo">
        <el-image
          :src="defaultAvatar"
          fit="cover"
          class="avatar"
        />
        <el-upload
          :action="uploadUrl"
          :headers="headers"
          :show-file-list="false"
          :on-success="handleUploadSuccess"
        >
          <!-- <el-button type="primary" size="small">更换头像</el-button> -->
        </el-upload>
      </div>

      <!-- 志愿者展示内容 -->
      <el-form :model="userInfo" label-width="80px" v-if="userInfo.user.userType === 'volunteer'">
        <el-form-item label="用户名">
          <el-input v-model="userInfo.user.username" disabled />
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model="userInfo.user.name" @blur="updateUser" />
        </el-form-item>
        <el-form-item label="学号">
          <el-input v-model="userInfo.user.studentId" @blur="updateUser" />
        </el-form-item>
        <el-form-item label="学院">
          <el-select 
            v-model="userInfo.user.college" 
            placeholder="请选择学院"
            @change="handleCollegeChange"
          >
            <el-option label="矿业工程学院" value="矿业工程学院" />
            <el-option label="环境与化工学院" value="环境与化工学院" />
            <el-option label="安全工程学院" value="安全工程学院" />
            <el-option label="电气与控制工程学院" value="电气与控制工程学院" />
            <el-option label="电子与信息工程学院" value="电子与信息工程学院" />
            <el-option label="机械工程学院" value="机械工程学院" />
            <el-option label="经济学院" value="经济学院" />
            <el-option label="管理学院" value="管理学院" />
            <el-option label="建筑工程学院" value="建筑工程学院" />
            <el-option label="马克思主义学院" value="马克思主义学院" />
            <el-option label="人文与社会科学学院" value="人文与社会科学学院" />
            <el-option label="计算机与信息工程学院" value="计算机与信息工程学院" />
            <el-option label="材料科学与工程学院" value="材料科学与工程学院" />
            <el-option label="理学院" value="理学院" />
            <el-option label="外国语学院" value="外国语学院" />
            <el-option label="国际教育学院" value="国际教育学院" />
          </el-select>
        </el-form-item>
        <!-- <el-form-item label="专业">
          <el-select 
            v-model="userInfo.user.major" 
            placeholder="请先选择学院"
            @blur="updateUser"
          >
            <el-option 
              v-for="major in currentMajors" 
              :key="major.value" 
              :label="major.label" 
              :value="major.value" 
            />
          </el-select>
        </el-form-item> -->
        <el-form-item label="邮箱">
          <el-input v-model="userInfo.user.email" @blur="updateUser" />
        </el-form-item>
        <el-form-item label="个性签名">
          <el-input
            v-model="userInfo.user.signature"
            type="textarea"
            @blur="updateUser"
          />
        </el-form-item>
      </el-form>

      <!-- 校组织展示内容 -->
      <el-form :model="userInfo" label-width="80px" v-if="userInfo.user.userType === 'schoolOrganization'">
        <el-form-item label="用户名">
          <el-input v-model="userInfo.user.username" disabled />
        </el-form-item>
        <el-form-item label="组织名称">
          <el-input v-model="userInfo.user.name" @blur="updateUser" />
        </el-form-item>
      </el-form>

      <!-- 校外组织展示内容 -->
      <el-form :model="userInfo" label-width="80px" v-if="userInfo.user.userType === 'offCampusOrganization'">
        <el-form-item label="用户名">
          <el-input v-model="userInfo.user.username" disabled />
        </el-form-item>
        <el-form-item label="组织名称">
          <el-input v-model="userInfo.user.name" @blur="updateUser" />
        </el-form-item>
      </el-form>

      <!-- 管理员展示内容 -->
      <el-form :model="userInfo" label-width="80px" v-if="userInfo.user.userType === 'admin'">
        <el-form-item label="用户名">
          <el-input v-model="userInfo.user.username" disabled />
        </el-form-item>
      </el-form>

      <div class="logout" v-if="userInfo">
        <el-button type="danger" @click="handleLogout">退出登录</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import api from '@/api'

const collegeMajorMap = {
  '矿业工程学院': [
    { label: '采矿工程', value: '采矿工程' },
    { label: '矿物加工工程', value: '矿物加工工程' },
    { label: '地质工程', value: '地质工程' },
    { label: '资源勘察工程', value: '资源勘察工程' },
    { label: '测绘工程', value: '测绘工程' },
    { label: '人文地理与城乡规划', value: '人文地理与城乡测绘' },
  ],
  '环境与化工学院': [
    { label: '化学工程与工艺', value: '化学工程与工艺' },
    { label: '应用化学', value: '应用化学' },
    { label: '环境工程', value: '环境工程' },
  ],
  '安全工程学院': [
    { label: '安全工程', value: '安全工程' }
  ],
  '电气与控制工程学院': [
    { label: '电气工程及其自动化', value: '电气工程及其自动化' },
    { label: '自动化', value: '自动化' },
    { label: '测控技术与仪器', value: '测控技术与仪器' },
    { label: '电气工程与智能控制', value: '电气工程与智能控制' },
  ],
  '电子与信息工程学院': [
    { label: '电子信息工程', value: '电子信息工程' },
    { label: '电子信息科学与技术', value: '电子信息工程' },
    { label: '通信工程', value: '通信工程' },
  ],
  '机械工程学院': [
    { label: '机械设计制造及其自动化', value: '机械设计制造及其自动化' },
    { label: '工业设计', value: '工业设计' },
    { label: '工业工程', value: '工业工程' },
    { label: '机械电子工程', value: '机械电子工程' },
    { label: '机械工艺技术', value: '机械工艺技术' },
  ],
  '经济学院': [
    { label: '国际经济与贸易', value: '国际经济与贸易' },
    { label: '金融学', value: '金融学' },
    { label: '经济学', value: '经济学' },
  ],
  '管理学院': [
    { label: '会计学', value: '会计学' },
    { label: '工商管理', value: '工商管理' },
    { label: '财务管理', value: '财务管理' },
    { label: '信息管理与信息系统', value: '信息管理与信息系统' },
    { label: '公共事业管理', value: '公共事业管理' },
    { label: '市场营销', value: '市场营销' },
  ],
  '建筑工程学院': [
    { label: '土木工程', value: '土木工程' },
    { label: '城乡规划', value: '城乡规划' },
    { label: '建筑学', value: '建筑学' },
    { label: '工程管理', value: '工程管理' },
    { label: '风景园林', value: '风景园林' },
    { label: '道路桥梁与渡河工程', value: '道路桥梁与渡河工程' },
  ],
  '马克思主义学院': [
    { label: '思想政治教育', value: '思想政治教育' },
  ],
  '人文与社会科学学院': [
    { label: '社会工作', value: '社会工作' },
    { label: '汉语言文学', value: '汉语言文学' },
    { label: '社会学', value: '社会学' },
    { label: '应用心理学', value: '应用心理学' },
  ],
  '计算机与信息工程学院': [
    { label: '计算机科学与技术', value: '计算机科学与技术' },
    { label: '软件工程', value: '软件工程' },
    { label: '物联网工程', value: '物联网工程' },
    { label: '数据科学与大数据技术', value: '数据科学与大数据技术' },
  ],
  '材料科学与工程学院': [
    { label: '材料成型及控制工程', value: '材料成型及控制工程' },
    { label: '金属材料工程', value: '金属材料工程' },
    { label: '无机非金属材料工程', value: '无机非金属材料工程' },
    { label: '焊接技术与工程', value: '焊接技术与工程' },
  ],
  '理学院': [
    { label: '数学与应用数学', value: '数学与应用数学' },
    { label: '工程力学', value: '工程力学' },
    { label: '应用物理学', value: '应用物理学' },
    { label: '应用统计学', value: '应用统计学' },
  ],
  '外国语学院': [
    { label: '英语', value: '英语' },
    { label: '商务英语', value: '商务英语' },
  ],
  '国际教育学院': [
    { label: '俄语', value: '俄语' },
  ],
}

export default {
  name: 'UserCenter',
  data() {
    return {
      defaultAvatar: require('@/assets/default-avatar.png'),
      uploadUrl: 'http://localhost:3000/api/users/uploadAvatar',
      collegeMajorMap,
    };
  },
  computed: {
    ...mapState('user', ['userInfo']),
    headers() {
      const token = localStorage.getItem('authToken');
      return {
        Authorization: `Bearer ${token}`,
      };
    },
    currentMajors() {
      const majors = this.userInfo.user.college 
        ? this.collegeMajorMap[this.userInfo.user.college] || []
        : [];
      console.log('当前专业列表:', majors); // 调试用
      return majors;
    },
  },
  mounted() {
    this.$store.dispatch('user/fetchUserInfo');
  },
  methods: {
    ...mapActions('user', ['updateProfile', 'logout']),

    handleCollegeChange() {
      this.userInfo.user.major = ''; // 清空已选专业
      this.updateUser();
    },

    async updateUser() {
      try {
        console.log('提交的用户信息:', this.userInfo.user);
        await this.updateProfile(this.userInfo.user);
        this.$message.success('资料更新成功');
      } catch (error) {
        this.handleError(error);
      }
    },

    handleError(error) {
      if (error.response && error.response.status) {
        if (error.response.status === 401) {
          this.$message.error('身份验证失败，请重新登录');
          this.$router.push('/login');
        } else {
          this.$message.error('操作失败');
        }
      } else {
        this.$message.error('网络错误或服务器未响应');
        console.error(error);
      }
    },

    async fetchUserInfo() {
      try {
        const response = await this.$axios.get('/user/info', {
          headers: this.headers,
        });
        this.$store.commit('user/setUser', response.data.data);
      } catch (error) {
        this.handleError(error);
      }
    },

    handleUploadSuccess(res) {
      this.userInfo.user.avatar = res.data.avatar;
      this.$message.success('头像更新成功');
      this.updateUser();
    },

    handleUploadError(err) {
      console.error('头像上传失败:', err);
      let errorMessage = '头像上传失败，请检查网络或服务器。';
      if (err && err.response && err.response.data && err.response.data.message) {
        errorMessage = `头像上传失败: ${err.response.data.message}`;
      } else if (err && err.message) { // Multer 错误可能直接在 err.message 中
        errorMessage = `头像上传失败: ${err.message}`;
      }
      this.$message.error(errorMessage);
    },

    handleLogout() {
      this.logout();
      this.$message.success('退出登录成功！');
      this.$router.push('/login');
    },
  },
};
</script>

<style scoped>
.user-center {
  padding: 40px 20px;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  min-height: 100vh;
}

.el-card {
  max-width: 680px;
  margin: 0 auto;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
  border: none;
  transition: transform 0.3s ease;
}

.el-card:hover {
  transform: translateY(-2px);
}

h2 {
  text-align: center;
  color: #2c3e50;
  font-size: 28px;
  font-weight: 600;
  letter-spacing: 1px;
  margin-bottom: 0;
}

.el-divider {
  margin: 20px 0;
  background-color: #e0e0e0;
}

/* 头像区域美化 */
.avatar-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 30px;
}

.avatar {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  margin-bottom: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border: 3px solid #fff;
}

.avatar:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

/* 自定义输入框 */
::v-deep .el-form-item__label {
  color: #6c757d;
  font-weight: 500;
}

::v-deep .el-input__inner {
  border-radius: 8px;
  border: 1px solid #e0e0e0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  padding: 12px 16px;
}

::v-deep .el-input__inner:focus {
  border-color: #5e72e4;
  box-shadow: 0 2px 6px rgba(94, 114, 228, 0.2);
}

::v-deep .el-input.is-disabled .el-input__inner {
  background-color: #f8f9fa;
  color: #6c757d;
}

/* 自定义选择框 */
::v-deep .el-select .el-input__inner {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%236c757d"><path d="M7 10l5 5 5-5z"/></svg>');
  background-repeat: no-repeat;
  background-position: right 12px center;
  background-size: 16px;
}

/* 文本域美化 */
::v-deep .el-textarea__inner {
  border-radius: 8px;
  min-height: 100px;
  padding: 12px 16px;
  line-height: 1.6;
}

/* 按钮美化 */
.el-button {
  border-radius: 8px;
  padding: 10px 24px;
  transition: all 0.3s ease;
  letter-spacing: 0.5px;
}

.el-button--primary {
  background: linear-gradient(135deg, #5e72e4 0%, #825ee4 100%);
  border: none;
}

.el-button--primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(94, 114, 228, 0.3);
}

.el-button--danger {
  background: linear-gradient(135deg, #f5365c 0%, #f56036 100%);
  border: none;
}

/* 登出按钮 */
.logout {
  margin: 40px 0 20px;
}

/* 表单间距优化 */
.el-form {
  padding: 0 30px;
}

.el-form-item {
  margin-bottom: 22px;
}

/* 学院选择动效 */
::v-deep .el-select-dropdown__item {
  padding: 10px 20px;
  transition: all 0.2s ease;
}

::v-deep .el-select-dropdown__item:hover {
  background-color: #f8f9fa;
  transform: translateX(4px);
}

/* 分割线动画 */
.el-divider__text {
  color: #6c757d;
  font-size: 14px;
  padding: 0 12px;
  background-color: transparent;
}
</style>
