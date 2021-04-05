<template>
  <el-container style="height:100%;">
    <el-aside style="height: 100%" width="200px">
      <div style="line-height:20px;"><img src="../assets/logoko.png" alt="" class="logo"/></div>
      <el-menu default-active="1-4-1" style="height: 90%" class="el-menu-vertical-demo">
        <el-menu-item index="1">
          <i class="el-icon-message"></i>
          <span slot="title">Email center</span>
        </el-menu-item>
        <el-menu-item index="2">
          <i class="el-icon-setting"></i>
          <span slot="title">Config</span>
        </el-menu-item>
      </el-menu>
    </el-aside>
    <el-container style="height:100%;">
      <el-header>
        <div style="margin-left: 600px;">
          <el-button type="info" icon="el-icon-message"></el-button>
          <el-input
            placeholder="Email Address"
            style="width: 310px"
            v-model="address"
            icon="el-icon-refresh" readonly>
            <el-button slot="append" icon="el-icon-refresh"></el-button>
          </el-input>
        </div>
        <el-divider></el-divider>
      </el-header>
      <el-main style="width: 100%">
        <div>
          <el-table
            :data="tableData"
            style="width: 100%"
            :row-class-name="tableRowClassName">
            <el-table-column
              prop="from"
              label="from"
              style="width:20%">
            </el-table-column>
            <el-table-column
              prop="subject"
              label="subject"
              style="width:20%">
            </el-table-column>
            <el-table-column
              prop="content"
              label="content"
              style="width:40%">
            </el-table-column>
            <el-table-column
              prop="time"
              label="time"
              style="width:20%">
            </el-table-column>
          </el-table>
        </div>
      </el-main>
      <el-footer></el-footer>
    </el-container>
  </el-container>
</template>

<script>
const axios = require('axios');
export default {
  name: "index",
  data() {
    return {
      address: null,
      tableData: [],
      ipUrl: 'ws://localhost:8080/join/',
      ws: null,
      lockReconnect: false,
      timeout: 28 * 1000,//30s one conn
      timeoutObj: null,
      serverTimeoutObj: null,
      timeoutnum: null,
    }
  },
  mounted() {
    this.getNewAddress();
  },
  // 页面销毁
  beforeDestroy() {
    this.isDestroyed = true
    this.timeoutnum && clearTimeout(this.timeoutnum)
    this.timeoutObj && clearTimeout(this.timeoutObj)
    this.serverTimeoutObj && clearTimeout(this.serverTimeoutObj)
    this.ws.close()
  },
  methods: {
    tableRowClassName({row, rowIndex}) {
      // console.log(row)
      // if (rowIndex === 1) {
      //   return 'warning-row';
      // } else if (rowIndex === 3) {
      //   return 'success-row';
      // }
      // return '';
    },
    getNewAddress() {
      axios.get('http://localhost:8080/recipient/getNewRecipient')
        .then((rep) => {
          this.address = rep.data.data
        }).catch(function (rep) {

      })
    },
    setTableData(data) {
      console.log(data)
      this.tableData.push(JSON.parse(data))
    },

    /*websocket*/
    initWebpack() {
      console.log('websocket init');
      // ws url
      this.ws = new WebSocket(this.ipUrl + this.address);
      this.ws.onopen = this.onopen;
      this.ws.onmessage = this.onmessage;
      this.ws.onclose = this.onclose;
      this.ws.onerror = this.onerror;
    },
    reconnect() {
      let that = this;
      if (that.lockReconnect) {
        return;
      }
      that.lockReconnect = true;
      that.timeoutnum && clearTimeout(that.timeoutnum);
      that.timeoutnum = setTimeout(() => {
        //new conn
        that.initWebpack();
        that.lockReconnect = false;
      }, 500);
    },
    reset() {
      let that = this;
      //clean time
      clearTimeout(that.timeoutObj);
      clearTimeout(that.serverTimeoutObj);
      //restart heart
      that.start();
    },
    //start heart check
    start() {
      const self = this;
      self.timeoutObj && clearTimeout(self.timeoutObj);
      self.serverTimeoutObj && clearTimeout(self.serverTimeoutObj);
      self.timeoutObj = setTimeout(() => {
        //send heart check
        if (self.ws.readyState === 1) {//如果连接正常
          self.ws.send("heartCheck");
        } else {
          self.reconnect();
        }
        self.serverTimeoutObj = setTimeout(function () {
          //timeout close
          self.ws.close();
        }, self.timeout);

      }, self.timeout)
    },
    onopen() {
      //开启心跳
      this.start();
    },
    onmessage(e) {
      console.log(e.data, 'return heart data');
      // set heart data to table
      this.setTableData(e.data);
      this.reset();
    },
    onclose(e) {
      console.log('websocket close: ' + e.code + ' ' + e.reason + ' ' + e.wasClean);
      // re conn
      this.reconnect();
    },
    onerror(e) {
      // re conn
      this.reconnect();
    }
  },
  watch: {
    address(new_val, old_val) {
      if (new_val !== null) {
        this.initWebpack();
      }
    }
  }
}
</script>

<style>
.el-header, .el-footer {
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  color: #333;
  text-align: center;
  line-height: 100vh;
}

.el-main {
  color: #333;
  line-height: 20px !important;
}

body > .el-container {
  margin-bottom: 40px;
}

.el-container:nth-child(5) .el-aside,
.el-container:nth-child(6) .el-aside {
  line-height: 260px;
}

.el-container:nth-child(7) .el-aside {
  line-height: 320px;
}

.logo {
  height: 75px;
}

.address {
  width: 220px;
  display: inline-block;
  text-align: right;
}

.demo-table-expand {
  font-size: 0;
}

.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}

.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}

.el-table__header tr,
.el-table__header th {
  padding: 0;
  height: 20px;
}

.el-table .warning-row {
  background: oldlace;
}

.el-table .success-row {
  background: #f0f9eb;
}
</style>
