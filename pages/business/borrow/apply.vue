<template>
    <view class="body">
        <view class="input-group inner">
            <view class="input-row input-flex border">
                <label class="title">{{$t('bus.jingKuanJinge')}}</label>
                <m-input type="text" v-model="amount" :placeholder="$t('tip.qingShuRu')+$t('bus.jingKuanJinge')" @input="onAmountCostChange"></m-input>
                <label class="rfix">({{$t('common.currency')}})</label>
            </view>
            <view class="input-row input-flex border">
                <label class="title">{{$t('bus.jieKuanShiChang')}}</label>
                <m-input type="number" v-model="timeLimit" :placeholder="$t('tip.qingShuRu')+$t('bus.jieKuanShiChang')" @input="onAmountCostChange"></m-input>
                <label class="rfix">({{$t('bus.timesu')}})</label>
            </view>
            <view class="input-row input-flex border">
                <label class="title">{{$t('bus.fuWuFei')}}</label>
                <label class="flex1">{{servicefee}}</label>
                <label class="rfix">({{$t('common.currency')}})</label>
            </view>
            <view class="input-row input-flex border">
                <label class="title">{{$t('bus.jieKuanLiXi')}}</label>
                <label class="flex1">{{interestfee}}</label>
                <label class="rfix">({{$t('common.currency')}})</label>
            </view>
        </view>

        <view class="input-group inner">
            <view class="input-row input-flex border">
                <label class="title">{{$t('bus.shiJiDaoZhang')}}</label>
                <label class="flex1">{{repAmount}}</label>
                <label class="rfix">({{$t('common.currency')}})</label>
            </view>
            <view class="input-row input-flex border">
                <label class="title long">{{$t('bus.shouKuanHang')}}</label>
                <label class="flex1">{{bankName}}</label>
                <label class="rfix"></label>
            </view>
            <view class="input-row input-flex border">
                <label class="title">{{$t('user.yingHangKaHao')}}</label>
                <label class="flex1">{{bankCard}}</label>
                <label class="rfix"></label>
            </view>
        </view>

        <view class="input-group inner">
            <view class="input-row input-flex border">
                <label class="title half">{{$t('bus.huanKuanZongE')}}</label>
                <label class="flex1">{{repTotal}}</label>
                <label class="rfix">({{$t('common.currency')}})</label>
            </view>
            <view class="input-row input-flex border">
                <label class="title half">{{$t('bus.huanKuanShiJian')}}</label>
                <label class="flex1">{{repTime}}</label>
                <label class="rfix"></label>
            </view>
        </view>

        <view class="input-line inner">
            <view class="input-flex">
                <label class="input-flex">
                    <checkbox value="cb" checked="true" style="transform:scale(0.6)" />
                    <text>{{ $t('user.woYiYueDuBingTongYi') }}</text>
                </label>
                <text class="text-link" @tap="onAgreement">{{ $t('bus.jieKuanXieYi') }}</text>
            </view>
        </view>

        <view class="input-line inner">
            <button type="primary" class="primary" @tap="onSubmit">{{ $t('common.apply2') }}</button>
        </view>
    </view>
</template>

<script>
    import {
        mapState,
        mapMutations
    } from 'vuex'

    import mInput from '@/static/components/m-input.vue';

    import util from '@/util/util.js'
    import appService from '@/services/application.js';
    import busService from '@/services/business.js';
    import userService from '@/services/user.js';

    export default {
        data() {
            return {
                query: { amount: "0" },
                amount: "0",
                timeLimit: "8",
                servicefee: "0",
                interestfee: "0",
                fee:'0',
                repAmount: '0',
                repTime: '',
                repTotal:'0',
                bankName: '',
                bankCard: '',
                lastInput: (new Date())
            }
        },
        components: {
            mInput
        },
        computed: mapState(["userId", "quota", "borrow"]),
        methods: {
            ...mapMutations(["pageAuth"]),
            onInit() {
                var that = this;
                that.amount = util.toMoney(that.borrow)+'';
                that.amountCost();
                userService.getBanks({ userId: that.userId }, function (obj, msg, code) {
                    that.bankName = obj["bank"];
                    that.bankCard = obj["cardNo"];
                })
            },
            onSubmit() {
                var that = this;

                var input = {
                    userId: that.userId,
                    amount: that.amountGet(),
                    timeLimit: that.timeLimit,
                    bankCard: that.bankCard
                };
                busService.borrowApply(input, function (obj, msg, code) {
                    util.success(msg || (that.$t('common.apply')), {
                        over() {
                            uni.navigateTo({
                                url: 'borrows'
                            });
                        }
                    });
                })
            },
            onAgreement() {
                var that = this;
                uni.navigateTo({
                    url: 'agreement?userId=' + that.userId + '&amount=' + that.amountGet() + '&limit=' + that.timeLimit + '&fee=' + that.fee
                });
            },
            onAmountCostChange(e) {
                var that = this;
                that.lastInput = new Date();
                setTimeout(() => {
                    var cur = new Date();
                    if (cur - that.lastInput >= 490) {
                        that.amountCost();
                    }
                }, 500)
            },
            amountCost(callback) {
                var that = this;
                that.amount = util.toMoney(that.amount);
                if (util.isEmpty(that.amountGet()) || util.isEmpty(that.timeLimit)) {
                    that.repAmount = '0';
                    that.interestfee ='0';
                    that.repTotal = '0';
                    that.fee = '0';
                    that.repTime = "";
                    return;
                }
                busService.borrowCost({ userId: that.userId, amount: that.amountGet(), timeLimit: that.timeLimit }, function (obj, msg, code) {
                    that.repAmount = util.toMoney(obj.realAmount) + '';
                    that.servicefee = util.toMoney(obj.totalFee)+'';
                    that.interestfee = util.toMoney(obj.interest) + '';
                    that.repTotal = util.toMoney((obj.realAmount) + (obj.interest) + (obj.totalFee)) + '';
                    that.fee = util.toMoney(obj.fee) + '';
                    that.repTime = obj.repayTime;
                })
            },
            amountGet() {
                var that = this;
                return (that.amount + '').replace(/,/g, "");
            }
        },
        onLoad: function (option) {
            var that = this;
            option = option || { amount: 0 };
            that.query = option;
        },
        onShow: function () {
            var that = this;
            that.pageAuth({ callback: that.onInit });
        }
    }
</script>

<style>
</style>
