<template>
  <div class="home">
    <Button type="primary" size="large" @click="handleCoinType">开单</Button>
    <div class="form-wrapper">
      <Form @submit="onSubmit">
        <cell title="" size="large">
          <template #title>实时价格</template>
          {{ configData.currentPrice }}
          <template #label>{{ formData.coinName }}</template>
        </cell>
        <field
          v-model="formData.enterPrice"
          name="入场价格"
          label="入场价格"
          placeholder="入场价格"
          :rules="[{ required: true, message: '请输入入场价格' }]"
        />
        <field
          v-model="formData.dangerPrice"
          name="止损价位"
          label="止损价位"
          placeholder="止损价位"
        />
        <field name="slider" label="风控买入额">
          <template #input>
            <!-- 区分 多/空 模式 -->
            <slider
              :max="
                isShortMode ? maxOnceDangerLimitForShort : maxOnceDangerLimit
              "
              step="5"
              v-model="formData.maxBuy"
            />
          </template>
        </field>
        <cell title="" value-class="item-detail">
          <template #title>
            最大风险额度（{{ configData.dangerRatio }}%）：{{ maxBuyLimit }}
          </template>
          <span @click="handleSetInteger"
            >建议最大买入额：{{ formData.maxBuy }}</span
          >
        </cell>
        <cell title="" value-class="item-detail">
          <template #title> 购买数量 </template>
          {{ this.amount }}
        </cell>
        <field name="slider" label="实际买入额">
          <template #input>
            <slider
              :max="configData.moneyTotal"
              step="5"
              v-model="formData.actualBuy"
            />
          </template>
        </field>
        <cell title="" value-class="item-detail">
          <template #title>
            <Button
              type="primary"
              round
              size="mini"
              @click="handleActualAmount()"
              >最大风控额</Button
            >
          </template>
          <Button
            type="primary"
            round
            size="mini"
            @click="handleActualAmount(5)"
            >+5%</Button
          >
          <Button
            type="danger"
            round
            size="mini"
            @click="handleActualAmount(-5)"
            >-5%</Button
          >
        </cell>
        <cell title="" value-class="item-detail">
          <template #title> 实际购买数量：{{ formData.amount }} </template>
          金额:{{ formData.actualBuy }}
        </cell>
        <field name="slider" label="盈亏预览">
          <template #input>
            <slider
              :min="configData.previewPriceMin"
              :max="configData.previewPriceMax"
              step="5"
              v-model="configData.previewPrice"
            />
          </template>
        </field>
        <cell title="" value-class="item-detail">
          <template #title>
            <Button
              type="danger"
              round
              size="mini"
              @click="handleSimplePreview(-5)"
              >-5%</Button
            >
            <Button
              type="primary"
              round
              size="mini"
              @click="handleSimplePreview(5)"
              >5%</Button
            >
          </template>
          <Button
            type="primary"
            round
            size="mini"
            @click="handleRatioPreview(1)"
            >盈亏比1:1</Button
          >
          <Button
            type="primary"
            round
            size="mini"
            @click="handleRatioPreview(1.5)"
            >盈亏比1.5:1</Button
          >
          <Button
            type="primary"
            round
            size="mini"
            @click="handleRatioPreview(2)"
            >盈亏比2:1</Button
          >
        </cell>
        <cell title="" value-class="item-detail">
          <template #title> 预览目标价位 </template>
          <Button
            type="primary"
            round
            size="mini"
            class="btn-reset"
            @click="handleResetPreview()"
            >重置入场价</Button
          >{{ configData.previewPrice }}
        </cell>
        <cell title="" value-class="item-detail">
          <template #title> 预览盈亏 </template>
          <!-- 盈亏预览： （预览价-入场价）/ （入场价-止损价） -->
          <span class="win-ratio"
            >盈亏比({{
              (
                (configData.previewPrice - formData.enterPrice) /
                (formData.enterPrice - formData.dangerPrice)
              ).toFixed(2)
            }}:1)</span
          >
          {{ previewWin }}
        </cell>

        <div style="margin: 16px">
          <button round block type="info" native-type="submit" center>
            提交
          </button>
        </div>
      </Form>
    </div>

    <popup v-model="visibleCoin" position="top" :style="{ height: '50%' }">
      <picker
        title="币种类型"
        show-toolbar
        :columns="columns"
        @confirm="onConfirm"
        @cancel="onCancel"
        @change="onChange"
      />
    </popup>
  </div>
</template>

<script>
import { Button, Popup, Picker, Toast, Form, Field, Slider, Cell } from "vant";

export default {
  name: "Home",
  components: { Button, Popup, Picker, Form, Field, Slider, Cell },
  data() {
    return {
      visibleCoin: false,
      formData: {
        dangerPrice: "",
        enterPrice: "",
        maxBuy: 0,
        actualBuy: 0,
        amount: "", // 实际购买数量
        coinName: "btccoin", // 币种
      },
      configData: {
        currentPrice: 2800, // 实时币种价格
        previewPrice: 0, // 预览盈亏
        previewPriceMax: 5550, // 预览最大限制（+150%）
        previewPriceMin: 0, // 预览最小限制 （-50%）
        moneyTotal: 5000, // 操作总金额
        dangerRatio: 2, // 最大风险额度比率
        actualBuyLimit: 2000,
      },
      columns: ["btccoin", "eth", "dogecoin", "uni"],
    };
  },
  computed: {
    // 做空模式
    isShortMode() {
      return this.formData.dangerPrice > this.formData.enterPrice;
    },
    // 最大风险额
    maxBuyLimit() {
      return (this.configData.moneyTotal * this.configData.dangerRatio) / 100;
    },
    // 建议单笔最大风险买入额度
    // * (单笔最大风险额度/(入场价-止损价)) * 入场价
    maxOnceDangerLimit() {
      return (
        (this.maxBuyLimit /
          (this.formData.enterPrice - this.formData.dangerPrice)) *
        this.formData.enterPrice
      ).toFixed(2);
    },
    // 单笔最大风险金额限制(空)
    // * (单笔最大风险额度/(入场价-止损价)) * 入场价
    maxOnceDangerLimitForShort() {
      return (
        (this.maxBuyLimit /
          (this.formData.dangerPrice - this.formData.enterPrice)) *
        this.formData.enterPrice
      ).toFixed(2);
    },
    amount() {
      return (this.formData.maxBuy / this.formData.enterPrice).toFixed(3) || 0;
    },
    previewWin() {
      return (
        (this.configData.previewPrice / this.formData.enterPrice - 1) *
        this.formData.enterPrice *
        this.formData.amount
      ).toFixed(2);
    },
  },
  created() {
    this.initData();
  },
  methods: {
    initData() {
      // 入场价格，默认填充初始化实时价格
      this.formData.enterPrice = 2600;
      // 目标价位，默认填充入场价格
      this.configData.previewPrice = 2600;
      // 获取实时价格
      this.configData.currentPrice = 2688;
      // previewPriceMax: 5550, // 预览最大限制（+150%）
      // previewPriceMin: 0, // 预览最小限制 （-50%）
      // 计算相关区间
      this.configData.previewPriceMax = 5500;
      this.configData.previewPriceMin = 0;
    },
    handleCoinType() {
      this.visibleCoin = !this.visibleCoin;
    },
    onConfirm(value, index) {
      Toast(`当前值：${value}, 当前索引：${index}`);
    },
    onChange(picker, value, index) {
      this.formData.coinName = value;
      Toast(`当前值：${value}, 当前索引：${index}`);
    },
    onCancel() {
      this.visibleCoin = false;
      Toast("取消");
    },
    onSubmit() {},
    // 比例调整预览盈亏
    handleSimplePreview(ratio) {
      const price = this.configData.previewPrice * (1 + ratio * 0.01);
      this.configData.previewPrice = Number(price.toFixed(2));
    },
    handleActualAmount(ratio) {
      let price;
      if (ratio) {
        price = this.formData.actualBuy * (1 + ratio * 0.01);
      } else {
        price = this.formData.maxBuy;
      }
      this.formData.actualBuy = Number(price.toFixed(2));
      this.formData.amount = Number(price / this.formData.enterPrice).toFixed(
        3
      );
    },
    handleResetPreview() {
      this.configData.previewPrice = Number(this.formData.enterPrice);
    },
    handleSetInteger() {},
    // 盈亏比预览
    handleRatioPreview(ratio) {
      const previewPrice =
        ratio * (this.formData.enterPrice - this.formData.dangerPrice) +
        Number(this.formData.enterPrice);
      this.configData.previewPrice = Number(previewPrice.toFixed(2));
    },
  },
};
</script>
<style lang="scss" scoped>
.home {
  width: 100%;
  height: 100%;
  .form-wrapper {
    overflow: auto;
    width: 100%;
    height: calc(100% - 50px);
  }
}
.item-detail {
  text-align: right;
}
.btn-reset {
  margin-right: 20px;
  position: relative;
  top: -5px;
}
.win-ratio {
  margin-right: 10px;
}
</style>
