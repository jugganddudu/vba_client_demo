<template>
  <div>{{ msg }}</div>
</template>

<script>
import CryptoJS from "crypto-js";
import api from "@/utils/api";
import { JSEncrypt } from 'jsencrypt'

export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "#",
      //公钥
      publicKey:
        "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCCIjEFl2tn3W9GssK0iDju2ILU5UlLwinOEy7CiZAF9Q/gYXPoAr8AWx3gQ8UVAszAEw1X25Qa2TPq63/boiLip6299pbzpCAToFFHzNhZT6SFm0HDvOQWJxs8k/kUDDMRF+TUbf9NJekhtg87bbMAB68fi3/jreeYJ7OYC5ZKSwIDAQAB",
      //测试接口url
      url: "https://112.65.144.19:9085/servlet/json",
      //银行账号
      acctNo: "6217001210031656620",
      //身份证号
      certSeq: "342422199203247574",
      //机构账号
      ptyAcct: "acctforzhouxx",
      //机构号
      ptyCD: "cdforzhouxx",
      //会话密钥
      ptyKey: "8f20363268a09922e760b83df1f8d1b1"
    };
  },
  mounted() {
    //this.doMount();

    var pub_key = '-----BEGIN PUBLIC KEY-----\n' +
          this.publicKey +
        '-----END PUBLIC KEY-----';

    let data='ptyacct='+this.ptyAcct+'&ptycd='+this.ptyCD+'&newptypwd=xnt123456&ptypwd=xnt123456';

    let encryptor = new JSEncrypt();

    encryptor.setPublicKey(pub_key);
    
    let encryptedStr=encryptor.encrypt(data);
//    console.log(CryptoJS.enc.Utf8.parse(encryptedStr));
    console.log('encryptedStr-1->',encryptedStr);

    // encryptedStr=CryptoJS.enc.Base64.stringify(
    //   CryptoJS.enc.Utf8.parse(encryptedStr)
    // );
    // console.log(encryptedStr);
    // encryptedStrBase64=CryptoJS.enc.Base64.stringify(
    //   CryptoJS.enc.Utf8.parse(encryptedStr)
    // );
    // encryptedStr=encodeURIComponent(encryptedStrBase64);
    encryptedStr=encodeURIComponent(encryptedStr);
    console.log('encryptedStr-2->',encryptedStr);

    encryptedStr=CryptoJS.enc.Base64.stringify(
      CryptoJS.enc.Utf8.parse(encryptedStr)
    );
    console.log('encryptedStr-3->',encryptedStr);


    let jsonParam={
      funcNo:"2000006",
      reqdata: encodeURIComponent(encryptedStr)
    };

    let _this=this;
    api.post({
      url: this.url,
      async: false,
      data: jsonParam,
      succ: function(data) {
        console.log("req-succ->", data);
        _this.msg="req-succ->"+JSON.stringify(data);
      },
      fail: function(data) {
        console.log("req-fail->", data);
      }
    });

  },
  methods: {
    getSignSrc(jsonObj, key) {
      key = key ? key : this.ptyKey;
      let arr = [];
      for (let key in jsonObj) {
        arr.push(key);
      }
      console.log("arr-sort->", arr, arr.sort());
      arr.sort();
      let str = "";
      for (let i in arr) {
        str += arr[i] + jsonObj[arr[i]];
      }
      return str + key;
    },
    getSignature(jsonSrc, key) {
      let signSrc = this.getSignSrc(jsonSrc);
      console.log("jsonSrc->", jsonSrc);
      console.log("signSrc->", signSrc);

      let signDes = CryptoJS.SHA512(signSrc);
      console.log("signDes->", signDes);

      return signDes;
    },
    getSHA1PRNG(keyStr) {
      keyStr = keyStr ? keyStr : this.ptyKey;
      let result = CryptoJS.SHA1(keyStr);
      console.log("getSHA1PRNG-shal-1->", result, result.toString());
      result = CryptoJS.SHA1(result);
      console.log("getSHA1PRNG-shal-2->", result, result.toString());

      console.log("getSHA1PRNG-3->", result, result.toString());
      result = result.toString().substring(0, 32);
      console.log("getSHA1PRNG-3-1->", result, result.toString());

      return result;
    },

    encrypt(src, keyStr) {
      console.log("encrypt:");
      console.log("src->", src);
      console.log("key->", keyStr);
      keyStr = keyStr ? keyStr : this.ptyKey;
      keyStr = this.getSHA1PRNG(keyStr);
      console.log("keyStr->", keyStr);

      // let key = keyStr;
      let key = CryptoJS.enc.Hex.parse(keyStr);
      console.log("key->", key);

      let srcs = CryptoJS.enc.Utf8.parse(src);
      console.log("srcs->", srcs);

      let encrypted = CryptoJS.AES.encrypt(srcs, key, {
        mode: CryptoJS.mode.ECB,
        padding: CryptoJS.pad.Pkcs7
      });
      console.log("encrypted->", encrypted);

      //下面为了明文显示加密值
      let result = encrypted.toString();
      console.log("result->", result);

      console.log();

      return result;

      //return CryptoJS.enc.Base64.stringify(result).toString();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
