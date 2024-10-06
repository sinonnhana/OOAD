<template>
  <div class="profile-page">
    <!-- 顶部背景图 -->
    <div class="profile-banner">
      <img :src="require('@/assets/background.png')" alt="banner" class="banner-img"/>
    </div>

    <!-- 用户基本信息区域 -->
    <div class="profile-info-box">
      <div class="profile-info">
        <div class="avatar-section">
          <img class="avatar" :src="require('@/assets/touxiang.jpeg')" alt="用户头像" />
          <div class="username-section">
            <h2>{{ user.username }}</h2>
            <p>{{ user.bio }}</p>
            <button class="edit-btn" @click="editProfile">编辑资料</button>
          </div>
        </div>

        <div class="stats-section">
          <div class="stat-item">
            <p>{{ user.points }}</p>
            <span>积分数</span>
          </div>
          <div class="stat-item">
            <p>{{ user.tokens }}</p>
            <span>Tokens</span>
          </div>
          <div class="stat-item">
            <p>{{ user.bots }}</p>
            <span>Bots数</span>
          </div>
        </div>
        <button class="recharge-btn" @click="showRechargePopup">充值积分 ¥</button>
        <button class="convert-btn" @click="showExchangePopup">兑换Tokens</button>

      </div>
    </div>

    <!-- 水平线 -->
    <hr class="custom-hr"/>

    <!-- 自我介绍编辑框 -->
    <div class="bio-section">
      <h3>自我介绍</h3>
      <textarea v-model="user.bio" placeholder="在这里写下你的自我介绍..."></textarea>
      <button @click="saveProfile" class="save-btn">保存自我介绍</button>
    </div>


    <!-- 我的bot展示 -->
    <div class="my-bots-section">
      <h3>我的bot</h3>
      <button @click="toggleBotList" class="bot-btn">查看我的bot</button>
      <div v-if="showBots" class="bot-list">
        <div v-for="bot in user.botsList" :key="bot.id" class="bot-item">
          <img :src="bot.image" alt="bot图片" />
          <p>{{ bot.name }}</p>
          <button @click="showBotDetails(bot)">查看详情</button>
        </div>
      </div>
    </div>

    <!-- 三个横向选项按钮 -->
    <div class="options-bar">
      <div v-for="(option, index) in options"
           :key="index"
           class="option"
           :class="{ active: selectedOption === index }"
           @click="selectOption(index)">
        {{ option }}
      </div>
    </div>

    <!-- 动态切换显示的内容区域 -->
    <transition name="slide-fade">
      <div class="content-area">
        <div v-if="selectedOption === 0">
          <!-- 动态内容 -->
          <h3>动态</h3>
          <p>这里展示的是动态内容</p>
        </div>
        <div v-if="selectedOption === 1">
          <!-- 我收藏的bot -->
          <h3>我收藏的bot</h3>
          <p>这里展示的是我收藏的bot</p>
        </div>
        <div v-if="selectedOption === 2">
          <!-- 我自定义的bot -->
          <h3>我的自定义bot</h3>
          <p>这里展示的是我自定义的bot</p>
        </div>
      </div>
    </transition>


    <!-- 充值弹窗 -->
    <div v-if="showRecharge" class="popup-overlay">
      <div class="popup">
        <h2>充值积分</h2>
        <input type="number" v-model="rechargeAmount" placeholder="请输入充值积分数"/>
        <button @click="confirmRecharge">确认充值</button>
        <button @click="closePopup">取消</button>
      </div>
    </div>

    <!-- 积分兑换 Tokens 弹窗 -->
    <div v-if="showConvertPopup" class="popup-overlay">
      <div class="popup">
        <h2>兑换 Tokens</h2>
        <p>当前积分数：{{ user.points }}</p>
        <input type="number" v-model="convertPoints" placeholder="输入要兑换的积分数"/>
        <button @click="convertPointsToTokens">兑换</button>
        <button @click="closeConvertPopup">取消</button>
      </div>
    </div>

    <!-- Bot详情弹窗 -->
    <div v-if="showBotPopup" class="popup-overlay">
      <div class="popup">
        <h2>{{ selectedBot.name }} 的简介</h2>
        <p>{{ selectedBot.description }}</p>
        <button @click="closeBotPopup">关闭</button>
      </div>
    </div>

    <!-- 积分兑换Tokens弹窗 -->
    <div v-if="showConvertPopup" class="popup-overlay">
      <div class="popup">
        <h2>兑换 Tokens</h2>
        <p>当前积分数：{{ user.points }}</p>
        <input type="number" v-model="convertPoints" placeholder="输入要兑换的积分数"/>
        <button @click="convertPointsToTokens">兑换</button>
        <button @click="closeConvertPopup">取消</button>
      </div>
    </div>

    <!-- 编辑资料弹窗 -->
    <el-dialog title="编辑资料" :visible.sync="showEditProfilePopup" width="30%">
      <el-form>
        <el-form-item label="用户名">
          <el-input v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="自我介绍">
          <el-input type="textarea" v-model="editForm.bio"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showEditProfilePopup = false">取消</el-button>
        <el-button type="primary" @click="confirmEditProfile">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        username: '测试用户',
        bio: '欢迎来到我的个人主页！',
        points: 1000, // 初始积分数
        tokens: 50000, // 初始tokens
        bots: 5, // 初始bots数
        botsList: [
          { id: 1, name: 'Chatbot 1', image: 'https://example.com/bot1.jpg', description: '这是第一个bot的简介' },
          { id: 2, name: 'Chatbot 2', image: 'https://example.com/bot2.jpg', description: '这是第二个bot的简介' },
        ]
      },

      options: ['动态', '我收藏的bot', '我的自定义bot'], // 选项数组
      selectedOption: 0 ,// 记录当前选中的选项

      showBots: false,
      showRecharge: false,
      showBotPopup: false,
      showConvertPopup: false,
      showEditProfilePopup: false, // 控制编辑资料弹窗显示
      selectedBot: {},
      rechargeAmount: '',
      convertPoints: '',
      editForm: { // 用于存储弹窗中的临时编辑数据
        username: '',
        bio: ''
      }
    };
  },
  methods: {
    // 切换选项
    selectOption(index) {
      this.selectedOption = index;
    },
    toggleBotList() {
      this.showBots = !this.showBots;
    },
    editProfile() {
      // 打开编辑资料弹窗时，将现有用户信息填入表单
      this.editForm.username = this.user.username;
      this.editForm.bio = this.user.bio;
      this.showEditProfilePopup = true;
    },
    confirmEditProfile() {
      // 确定修改后，更新用户信息并关闭弹窗
      this.user.username = this.editForm.username;
      this.user.bio = this.editForm.bio;
      this.showEditProfilePopup = false;
      alert('资料已更新');
    },
    saveProfile() {
      alert('自我介绍已保存：' + this.user.bio);
    },
    showRechargePopup() {
      this.showRecharge = true;
    },
    confirmRecharge() {
      this.user.points += parseInt(this.rechargeAmount);
      alert(`充值成功！当前积分数：${this.user.points}`);
      this.closePopup();
    },
    closePopup() {
      this.showRecharge = false;
    },
    showBotDetails(bot) {
      this.selectedBot = bot;
      this.showBotPopup = true;
    },
    closeBotPopup() {
      this.showBotPopup = false;
    },
    showConvertPopup() {
      this.showConvertPopup = true;
    },
    showExchangePopup() {
      this.showConvertPopup = true;
    },
    convertPointsToTokens() {
      const pointsToConvert = parseInt(this.convertPoints);
      if (pointsToConvert <= this.user.points) {
        const tokensToAdd = pointsToConvert * 100;
        this.user.points -= pointsToConvert;
        this.user.tokens += tokensToAdd;
        alert(`成功兑换 ${tokensToAdd} tokens！`);
        this.closeConvertPopup();
      } else {
        alert('积分不足');
      }
    },
    closeConvertPopup() {
      this.showConvertPopup = false;
    }
  }
};

</script>

<style scoped>
/* 页面整体布局 */
.profile-page {
  min-width: 70%;
  width: 90%;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

/* 背景图样式 */
.profile-banner {
  position: relative;
  width: 100%;
  height: 5%;
}

.banner-img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

/* 灰色大框 */
.profile-info-box {
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 10px;
  margin-top: 20px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.profile-info {
  position: relative;
  display: flex;
  align-items: center;
}

.avatar-section {
  display: flex;
  align-items: center;
}

.avatar {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  margin-right: 20px;
}

.username-section {
  flex-grow: 1;
}

.edit-btn {
  background-color: #ff4081;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.recharge-btn{
  background-color: #ff4081;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  position: absolute;
  bottom: 0;
  right: 206px;
  width: 15%;
  height: 20%;
}

.convert-btn{
  background-color: #ff4081;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  position: absolute;
  bottom: 0;
  right: 40px;
  width: 15%;
  height: 20%;
}


.stats-section {
  position: absolute;
  top: 0;
  right: 0;
  display: flex;
  justify-content: space-around;
  width: 40%;
}

.stat-item {
  text-align: center;
}

.stat-item p {
  font-size: 20px;
  font-weight: bold;
}

/* 水平线 */
.custom-hr {
  border: 0;
  height: 2px;
  background-color: #ddd;
  margin: 20px 0;
}

/* 弹窗样式 */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
  width: 400px;
}

.popup input, .popup textarea {
  display: block;
  padding: 10px;
  width: 80%;
  border-radius: 5px;
  border: 1px solid #ccc;
}


/* 三个横向选项按钮 */
.options-bar {
  display: flex;
  justify-content: space-between;
  border-bottom: 2px solid #ddd;
  position: relative;
  margin-bottom: 20px;
}

.option {
  flex-grow: 1;
  flex-basis: 0; /* 基础宽度为0，完全依赖于 flex-grow */
  text-align: center; /* 居中对齐文本 */
  padding: 10px;
  cursor: pointer;
  font-weight: bold;
  color: #333;
  position: relative;
  transition: color 0.3s;
}

/* 当前选中项的样式 */
.option.active {
  color: #ff4081;
}

.option:not(:last-child)::after {
  content: '';
  position: absolute;
  right: 0;
  top: 10px;
  bottom: 10px;
  width: 2px;
  background-color: #ddd;
}

.option.active::before {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  right: 0;
  height: 4px;
  background-color: #ff4081;
  transition: left 0.3s, right 0.3s;
}

/* 动态内容显示区域 */
.content-area {
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  min-height: 200px;
}

/* 滑动过渡效果 */
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: transform 0.3s ease;
}
.slide-fade-enter, .slide-fade-leave-to /* .slide-fade-leave-active in <2.1.8 */ {
  transform: translateX(100%);
}


</style>
