<template>
  <v-layout column align-center justify-center>
    <v-flex xs12 sm6 d-flex>
      <form class="SetupPage">
        <v-icon v-show="connectionStatus" x-large color="success">wifi</v-icon>
        <v-icon x-large v-show="!connectionStatus" color="error">wifi_off</v-icon>

        <v-select
          :items="networks"
          v-model="ssidWifi"
          item-text="ssid"
          item-value="ssid"
          label="選擇無線網路"
          solo
          v-show="networks.length > 0"
        ></v-select>
        <div v-show="ssidWifi !=''">
          <v-text-field v-model="passwordWifi" solo placeholder="無線網路密碼" type="password"></v-text-field>
          <v-text-field
            v-model="deviceNumber"
            placeholder="設備序號"
            solo
            type="text"
            hint="此設備的序號附在盒子內" 
            :rules="nameRules"
            required
          ></v-text-field>
        </div>
      </form>
    </v-flex>
    <v-flex xs12 sm6 d-flex>
      <v-btn large @click="handleScan">偵測無線網路</v-btn>
      <div v-show="ssidWifi!=''">
        <v-btn large color="primary" @click="connectWifi">連線</v-btn>
      </div>
    </v-flex>
    <v-progress-linear v-show="scanStatus" :indeterminate="true"></v-progress-linear>
    <v-snackbar
      v-model="snackbar"
      :color="snackbarColor"
      :multi-line="false"
      :timeout="5000"
      :vertical="false"
    >
      {{ ssidWifi }} : {{snacbarText}}
      <v-btn dark flat @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-layout>
</template>
<script>
import axios from "axios";

export default {
  name: "SetupPage",

  data() {
    return {
      networks: [],

      ssidWifi: "",

      passwordWifi: "",

      deviceNumber: "",

      ip: "http://192.168.104.1:8081",

      connectionStatus: false,
      scanStatus: false,
      snackbar: false,
      snacbarText: "",
      snackbarColor: "",
      nameRules: [
        v => !!v || '序列號需要填寫',
        v => (v && v.length >= 10) || '序列號不可小過10字元'
      ],
    };
  },

  methods: {
    handleScan: async function() {
      this.scanStatus = true;
      this.networks = [];
      this.ssidWifi = "";
      this.passwordWifi = "";
      try {
        let result = await axios.get(this.ip + "/scanWifi");
        if (result.data.length <= 0) {
          this.snackbarColor = "error";
          this.snacbarText = "無偵測到無線網路";
          this.snackbar = true;
        }
        this.networks = result.data;
        // this.networks = [{ ssid: "test_01" }];
        this.scanStatus = false;
      } catch (err) {
        this.scanStatus = false;
        console.error(err);
      }
    },

    connectWifi: async function() {
      this.scanStatus = true;
      try {
        var wifiSettings = {
          ssid: this.ssidWifi,
          pass: this.passwordWifi,
          device: this.deviceNumber
        };
        if (this.deviceNumber == "" || this.deviceNumber.length < 10) {
          this.snacbarText = "設備序號不能為空並且不可小過10字元";
          this.snackbar = true;
          this.snackbarColor = "error";
          this.scanStatus = false;
        } else {
          let result = await axios.post(this.ip + "/setWifi", wifiSettings);
          if (result.data.connectionStatus == false) {
            this.snacbarText = result.data.message;
            this.snackbar = true;
            this.snackbarColor = "error";
          } else {
            this.snacbarText = result.data.message;
            this.snackbar = true;
            this.snackbarColor = "success";
            this.crossIframePassSerialNumber(wifiSettings.device.toString());
          }
          // this.snacbarText = result.data.message;
          // this.snackbar = true;
          // this.snackbarColor = "success";
          this.connectionStatus = result.data.connectionStatus;
        }
        this.scanStatus = false;
      } catch (err) {
        throw err;
      }
    },

    connectStatusHandle: async function() {
      try {
        let result = await axios.get(this.ip + "/statusConnection");
        this.connectionStatus = result.data.ssid ? true : false;
        this.deviceNumber = result.data.deviceNumber;
      } catch (err) {
        console.error(err);
      }
    },

    crossIframePassSerialNumber:function(snum){
      //send back value to parent from iframe
      parent.postMessage({serialNumber:snum} , '*')
    }

  },

  mounted: function() {
    this.connectStatusHandle();
  }
};
</script>
<style scoped>
</style>

