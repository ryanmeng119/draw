<template>
  <el-dialog
    :visible="visible"
    :append-to-body="true"
    width="390px"
    @close="$emit('update:visible', false)"
    class="c-LotteryConfig"
    :show-close="false"
  >
    <div class="c-LotteryConfigtitle" slot="title">
      <span :style="{ fontSize: '16px', marginRight: '20px' }">
        抽獎設定
      </span>
      <el-button size="mini" @click="addLottery">增加獎項</el-button>
      <el-button size="mini" type="primary" @click="onSubmit"
        >儲存設定</el-button
      >
      <el-button size="mini" @click="$emit('update:visible', false)"
        >取消</el-button
      >
    </div>
    <div class="container">
      <el-form ref="form" :model="form" size="mini">
        <el-form-item label="抽獎標題">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="抽獎總人數">
          <el-input
            v-model="form.number"
            :min="1"
            :step="1"
            disabled
          ></el-input>
        </el-form-item>
        <el-form-item
          :label="lottery.name"
          v-for="lottery in storeNewLottery"
          :key="lottery.key"
        >
          <el-input
            v-model="form[lottery.key]"
            @input="UpdateNumberOfAwards($event, lottery)"
          ></el-input>
        </el-form-item>
      </el-form>
    </div>

    <el-dialog
      :visible.sync="showAddLottery"
      :append-to-body="true"
      width="300px"
      class="dialog-showAddLottery"
    >
      <div class="add-title" slot="title">增加獎項</div>
      <el-form ref="newLottery" :model="newLottery" size="mini">
        <el-form-item label="獎項名稱">
          <el-input v-model="newLottery.name"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addHandler">增加獎項</el-button>
          <el-button @click="showAddLottery = false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </el-dialog>
</template>
<script>
import { setData, configField } from '@/helper/index';
import { randomNum } from '@/helper/algorithm';
export default {
  name: 'LotteryConfig',
  props: {
    visible: Boolean
  },
  computed: {
    form: {
      get() {
        return this.$store.state.config;
      },
      set(val) {
        // this.$store.commit('setConfig', val);
        return val;
      }
    },
    storeNewLottery() {
      return this.$store.state.newLottery;
    },
    quantityVerification() {
      return this.form.number >= this.totalQueued;
    },
    totalQueued() {
      return Object.keys(this.form)
        .filter(key => {
          return key !== 'name' && key !== 'number';
        })
        .reduce((total, key) => {
          return total + Number(this.$store.state.config[key]);
        }, 0);
    }
  },
  data() {
    return {
      showAddLottery: false,
      newLottery: { name: '' }
    };
  },
  methods: {
    onSubmit() {
      if (this.quantityVerification) {
        setData(configField, this.form); // 字串 config
        this.$store.commit('setConfig', this.form);
        this.$emit('update:visible', false);

        this.$message({
          message: '儲存成功',
          type: 'success'
        });

        this.$nextTick(() => {
          this.$emit('resetconfig');
        });
      } else {
        this.$message({
          message: '獎項頒發數量數超出抽獎人數，請重新確認！',
          type: 'warning'
        });
      }
    },

    addLottery() {
      this.showAddLottery = true;
    },
    randomField() {
      const str = 'abcdefghijklmnopqrstuvwxyz';
      let fieldStr = '';
      for (let index = 0; index < 10; index++) {
        fieldStr += str.split('')[randomNum(1, 27) - 1];
      }
      return `${fieldStr}${Date.now()}`;
    },
    addHandler() {
      const field = this.randomField(); // 取得亂碼，做識別碼用.
      const data = {
        key: field,
        name: this.newLottery.name
      };
      this.$store.commit('setNewLottery', data);

      this.showAddLottery = false;
    },
    UpdateNumberOfAwards(value, item) {
      console.log(this.totalQueued);
      const result = Number(value) ? Number(value) : 0;
      this.form[item.key] = result;
    }
  }
};
</script>
<style lang="scss">
.c-LotteryConfig {
  .el-dialog__body {
    height: 340px;
    .container {
      height: 100%;
      overflow-y: auto;
      padding: 0 10px;
    }
  }
}
.dialog-showAddLottery {
  .el-dialog {
    height: 186px;
  }
}
</style>
