<style scoped>
.chongzhi {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background-color: #ddd;
}

.czhiview {
  width: 375px;
  margin-left: calc((100% - 375px) / 2);
  float: left;
}

.czhiview_form {
  width: 100%;
  float: left;
}
.czhiview_form_hblist {
  width: calc(100% - 30px);
  float: left;
  overflow: hidden;
  background-color: #fff;
  margin-top: 15px;
  margin-left: 15px;
  height: 50px;
  border-radius: 5px;
}
.czhiview_form_hblist_t {
  float: left;
  line-height: 50px;
  font-size: 14px;
  padding: 0 15px;
  opacity: 0.8;
}
.czhiview_form_hblist_ul {
  display: flex;
  overflow-x: scroll;
  height: 50px;
  align-items: center;
  margin-right: 15px;
}
.czhiview_form_hblist_item {
  margin: 0 10px;
  font-size: 14px;
  opacity: 0.5;
}
.czhiview_form_hblist_item.ac {
  color: rgb(19, 208, 255);
  opacity: 1;
}

.czhiview_form_shurk {
  width: calc(100% - 60px);
  margin-left: 15px;
  background-color: #fff;
  margin-top: 15px;
  float: left;
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  padding: 15px 15px;
}
.czhiview_form_shurk_t {
  font-size: 18px;
  opacity: 0.9;
}

.czhiview_form_shurk_input {
  display: flex;
  margin-top: 15px;
  align-items: center;
}
.czhiview_form_shurk_input_l {
  margin-right: 10px;
  font-size: 24px;
  font-weight: 500;
  opacity: 0.8;
}
.czhiview_form_shurk_input input {
  flex-grow: 1;
}
.czhiview_form_czhi {
  float: left;
  width: calc(100% - 30px);
  margin-left: 15px;
  margin-top: 30px;
}
.czhiview_form_shurk_msg {
  padding-top: 15px;
  opacity: 0.5;
  font-size: 14px;
  padding-left: 10px;
}
</style>
<template>
  <div class="chongzhi">
    <van-nav-bar title="drawing" right-text="" left-arrow :fixed="true" :placeholder="true" @click-left="goback" z-index="99" />

    <div class="czhiview">
      <div class="czhiview_form">
        <div class="czhiview_form_hblist">
          <div class="czhiview_form_hblist_t">{{ $t('message.wallet.selectCurrency')}}</div>
          <div class="czhiview_form_hblist_ul">
            <div class="czhiview_form_hblist_item" v-for="(li,index) in huobi" :key="index" @click="hbindex = index" :class="hbindex == index ? 'ac':''">{{li.id}}</div>
          </div>
        </div>
        <div class="czhiview_form_shurk">
          <div class="czhiview_form_shurk_t"> </div>
          <div class="czhiview_form_shurk_input">
            <div class="czhiview_form_shurk_input_l">{{ huobi[hbindex]['id'] }}</div>
            <el-input :placeholder="$t('message.enterContent')" v-model="je"></el-input>
          </div>
          <div class="czhiview_form_shurk_msg">
           {{ $t('message.wallet.canWithdraw')}} {{ huobi[hbindex]['je'] / (10**huobi[hbindex]['num'])  }}
          </div>
        </div>

        <div class="czhiview_form_czhi">
          <van-button type="primary" block @click="tixianajax">{{ $t('message.wallet.withdrawMoney')}}</van-button>
        </div>
      </div>
    </div>

  </div>
</template>
<script>
import { Toast } from 'vant';

import Web3 from "web3";
import Web3Modal from "web3modal";

import config from "@/config";

var hbarr = [];

for (const key in config['hbi'][config['key']]) {
  config['hbi'][config['key']][key]['je'] = 0;
  hbarr.push(config['hbi'][config['key']][key]);
}

//全局变量
var web3 = "";
var address = "";
var ethereum = window.ethereum;
export default {
  data() {
    return {
      huobi: hbarr,
      hbindex: 0,
      je: 0
    }
  },
  watch: {
    hbindex() {
      this.gethuobizichan();
    }
  },
  mounted() {
    if (this.$route.query.title) {
      for (let index = 0; index < this.huobi.length; index++) {
        if (this.huobi[index]['id'] == this.$route.query.title) {
          this.hbindex = index;
          break;
        }
      }
    }
    //监测用户是否安装MASK
    if (typeof ethereum === "undefined") {
      alert(this.$t('message.currencyOtc.install'));
    } else {
      //初始化
      webinit();
    }

    Toast.setDefaultOptions('loading', {
      forbidClick: false,
      closeOnClickOverlay: false,
      duration: 0,
      overlay: true
    });

    var dq = this;
    async function webinit() {
      const providerOptions = {
        /* See Provider Options Section */
      };
      const web3Modal = new Web3Modal({
        network: "mainnet",
        cacheProvider: true,
        providerOptions,
      });
      var provider = await web3Modal.connect();
      web3 = new Web3(provider);
      if (web3 && provider) {
        //其他钱包使用测试网络
        // if (window.ethereum.isImToken || window.ethereum.isMetaMask) {
        //     var wlcode = window.ethereum.networkVersion;
        //     //imtoken只能查看 无法操作 出发是ETF主网
        //     if (window.ethereum.isImToken) {
        //         web3.setProvider(config["hyue"][config["key"]]["Url"]);
        //     }
        //     //MetaMask 钱包不等于4  进入专用网络 等于4使用本地钱包
        //     if (window.ethereum.isMetaMask && wlcode != 4) {
        //         web3.setProvider(config["hyue"][config["key"]]["Url"]);
        //     }
        // }else{
        //     web3.setProvider(config["hyue"][config["key"]]["Url"]);
        // }
        address = provider.selectedAddress;
        dq.gethuobizichan();
      }
    }

  },
  methods: {
    goback() {
      this.$router.go(-1);
    },
    //如果过亿请转换
    getFNum(num_str) {
      num_str = num_str.toString();
      if (num_str.indexOf("+") != -1) {
        num_str = num_str.replace("+", "");
      }
      if (num_str.indexOf("E") != -1 || num_str.indexOf("e") != -1) {
        var resValue = "",
          power = "",
          result = null,
          dotIndex = 0,
          resArr = [],
          sym = "";
        var numStr = num_str.toString();
        if (numStr[0] == "-") {
          // 如果为负数，转成正数处理，先去掉‘-’号，并保存‘-’.
          numStr = numStr.substr(1);
          sym = "-";
        }
        if (numStr.indexOf("E") != -1 || numStr.indexOf("e") != -1) {
          var regExp = new RegExp(
            "^(((\\d+.?\\d+)|(\\d+))[Ee]{1}((-(\\d+))|(\\d+)))$",
            "ig"
          );
          result = regExp.exec(numStr);
          if (result != null) {
            resValue = result[2];
            power = result[5];
            result = null;
          }
          if (!resValue && !power) {
            return false;
          }
          dotIndex = resValue.indexOf(".") == -1 ? 0 : resValue.indexOf(".");
          resValue = resValue.replace(".", "");
          resArr = resValue.split("");
          if (Number(power) >= 0) {
            var subres = resValue.substr(dotIndex);
            var length = dotIndex == 0 ? 0 : subres.length;
            power = Number(power);
            //幂数大于小数点后面的数字位数时，后面加0
            for (var i = 0; i < power - length; i++) {
              resArr.push("0");
            }
            if (power - subres.length < 0) {
              resArr.splice(dotIndex + power, 0, ".");
            }
          } else {
            power = power.replace("-", "");
            power = Number(power);
            //幂数大于等于 小数点的index位置, 前面加0
            for (let i = 0; i < power - dotIndex; i++) {
              resArr.unshift("0");
            }
            var n = power - dotIndex >= 0 ? 1 : -(power - dotIndex);
            resArr.splice(n, 0, ".");
          }
        }
        resValue = resArr.join("");

        return sym + resValue;
      } else {
        return num_str;
      }
    },
    async gethuobizichan() {
      //查询资产余额
      var czconn = new web3.eth.Contract(
        config['hyue'][config['key']]['Ccdotc']['abi'],
        config['hyue'][config['key']]['Ccdotc']['heyue']
      );
      var pro = await czconn.methods.balancepro(address, this.huobi[this.hbindex]['heyue']).call();
      this.huobi[this.hbindex]['je'] = pro;
    },
    tixianajax() {
      Toast.loading({ message: this.$t('message.wallet.withdrawing1') });
      var dq_je = this.huobi[this.hbindex]['je'];
      var dq = this;
      var czconn = new web3.eth.Contract(
        config['hyue'][config['key']]['Ccdotc']['abi'],
        config['hyue'][config['key']]['Ccdotc']['heyue']
      );
      var tkje = this.je * (10 ** this.huobi[this.hbindex]['num']);
      tkje = dq.getFNum(tkje);
      if (this.huobi[this.hbindex]['je'] >= Number(tkje)) {
        //执行提款操作
        czconn.methods.withdraw(this.huobi[this.hbindex]['heyue'], tkje).send({
          from: address
        }, (err, ret) => {
          if (ret) {
            tikchaxun();
          } else {
            Toast.clear();
            Toast.fail(this.$t('message.wallet.submit1'));
          }
        });
      } else {
        Toast.clear();
        Toast.fail(this.$t('message.wallet.exceedLimit'));
      }

      //轮询查询是否提款成功
      function tikchaxun() {
        var tk_je = dq_je - (dq.je * (10 ** dq.huobi[dq.hbindex]['num']));
        czconn.methods.balancepro(address, dq.huobi[dq.hbindex]['heyue']).call((err, ret) => {
          if (ret) {
            if (ret == tk_je) {
              Toast.clear();
              Toast.success(this.$t('message.wallet.withdrawSuccess'));
              dq.huobi[dq.hbindex]['je'] = tk_je;
            } else {
              setTimeout(() => {
                tikchaxun();
              }, 3000);
            }
          }
        });
      }
    }
  }
}
</script>