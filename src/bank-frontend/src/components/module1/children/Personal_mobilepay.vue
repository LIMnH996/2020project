<template>
<div>
  <!-- <span content-position:center>请选择你要充值的按钮</span>
    <el-divider><i class="el-icon-mobile-phone"></i></el-divider> -->
    <el-steps :active="1">
  <el-step title="步骤 1" description="选择你要充值的项目"></el-step>
  <el-step title="步骤 2" description="输入充值金额和相关信息"></el-step>
  <el-step title="步骤 3" description="密码验证"></el-step>
</el-steps>
<el-divider><i class="el-icon-mobile-phone"></i></el-divider>
    <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  手机充值
   </el-button>
   <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  电费充值
   </el-button>
    <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  水费充值
   </el-button>
    <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  有线电视
   </el-button>
    <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  生活缴费
   </el-button>
   <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  城市服务
   </el-button>
   <el-button @click="dialog = true" type="primary" style="margin: 35px;">
  医疗健康
</el-button>
<el-divider><i class="el-icon-mobile-phone"></i></el-divider>
  <el-table :data="bill_list">
        <el-table-column prop="date" label="账单日期">
        </el-table-column>
        <el-table-column prop="money" label="账单金额" >
        </el-table-column>
        <el-table-column prop="balance" label="银行卡余额">
        </el-table-column>
        <el-table-column prop="method" label="支付方式">
        </el-table-column>
        <el-table-column prop="details" label="详情">
        </el-table-column>
  </el-table>
<el-drawer
  :before-close="handleClose"
  :visible.sync="dialog"
  custom-class="demo-drawer"
  size="50%"
  ref="drawer"
  >
  <div class="demo-drawer__content">
    <el-form ref="form" :model="form" label-width="200px">
      <el-form-item label="电话号码">
        <el-input v-model="form.mobile"></el-input>
      </el-form-item>
      <el-form-item label="支付金额">
        <el-input v-model="form.money"></el-input>
      </el-form-item>
    </el-form>
    <div class="demo-drawer__footer">
      <el-button @click="cancelForm">取 消</el-button>
      <el-button @click="innerDrawer = true">确 定</el-button>
        <el-drawer
        :append-to-body="true"
        :before-close="handleClose"
        :visible.sync="innerDrawer">
        <el-form :model="form">
      <el-form-item label="手机号" :label-width="formLabelWidth">
        <el-input v-model="mobile" autocomplete="off"></el-input>
      </el-form-item>
       <el-form-item label="输入密码" :label-width="formLabelWidth">
        <el-input v-model="money" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
      <el-button @click="cancelForm">取 消</el-button>
        <el-button type="primary" @click="$refs.drawer.closeDrawer()" :loading="loading">{{ loading ? '提交中 ...' : '确 定 支 付' }}</el-button>
   </el-drawer>
   
    </div>
  </div>
</el-drawer>

</div>
</template>


<script>
import axios from "axios";
  export default {
  data() {
    return {
      //本机用户名密码验证
      user: {
        mobile: "",
        name: "",
        password: "",
      },
      //传递收款人相关信息
      form: {
        money: "",
        payee: "",
        payer: "",
        details: "",
      },
      //表单
       bill_list: {
        mobile: "",
        name: "",
        date: "",
        money: "",
        method: "",
        details: "",
        balance: 0.0,
      },
      dialog: false,
      loading: false,
      innerDrawer: false,
      // form: {
      //   name: '',
      //   region: '',
      //   date1: '',
      //   date2: '',
      //   delivery: false,
      //   type: [],
      //   resource: '',
      //   desc: ''
      // },
      formLabelWidth: '80px',
      timer: null,
    };
  },
  mounted() {
    // this.bill_list.mobile = this.$route.query.mobile;
    // this.bill_list.name = this.$route.query.username;
    // console.log(this.bill_list.mobile);
   console.log(333);
    console.log(this.$cookies.get("mobile"))
    //表单 talble
    axios
      .post("/personal/wallet/", {
        mobile: this.$cookies.get("mobile")
      })
      .then((res) => {
        console.log(res.data.data);
        var data = [];
        var kind = ["转账汇款","手机支付","理财基金","黄金","转账收入","其他"];
        for(var k =0;k<res.data.data.bill_list.length ;k++){
          var obj ={};
          obj.date=this.dateFilter(res.data.data.bill_list[k].date);
          obj.money=this.numFilter(res.data.data.bill_list[k].money);
          obj.method=kind[res.data.data.bill_list[k].method];
          obj.details=res.data.data.bill_list[k].details;
          obj.balance=this.numFilter(res.data.data.bill_list[k].balance);
          data[res.data.data.bill_list.length - k-1]=obj;
          console.log(k);
        }
        this.bill_list=data;
        // this.bill_list.date = res.data.data.date;
        // this.bill_list.balance = this.numFilter(res.data.data.balance);
        // this.bill_list.money = this.numFilter(res.data.data.money);
        // this.bill_list.method = res.data.data.method;
        // this.bill_list.details = res.data.data.details;
        console.log(res.data.data.email);
     
      });
  },
  methods: {
    //点击确定按钮之后检查是否输入，验证输入手机号和姓名匹配
    onSubmit() {
      if (!this.form.money) {
        this.$message.error("请输入金额！");
      } else if (!this.form.payee) {
        this.$message.error("请输入收款人账号！");
      } else {
        //弹出对话框，验证手机号的主人和卡号，并且输入密码
        this.dialogVisible = true;
        axios
          .post("/personal/mobile/", {
            mobile: this.form.payee,
          })
          .then((res) => {
            console.log('输出7777',res.data);
            if (res.data.success === true) {

              this.test.name=res.data.data.username;
               
            }
          });

        return;
      }
    },
    //点确定按钮之后检查密码是否正确,余额是否足够
    inspect() {
      console.log(555);
      console.log(this.user);
      if (!this.user.password) {
        this.$message.error("请输入密码！");
        return;
      } else {
        
        //点确定后验证余额是否充足
        axios
          .post("/personal/pay/", {
            mode: "mobilepay",
            money: this.form.money,
            detail: this.form.details,
            password: this.user.password,
          })
          .then((res) => {
            // console.log('输出response.data.status', res.data.status);
            if (res.data.success === true) {
              var str = "转账成功!您的余额还有：" + res.data.data.balance;
              alert(str);
            } else {
              console.log(res.data)
              alert(res.data.error);
            }
          });
        
      }
    },
    numFilter(value) {
      let realVal = "";
      if (!isNaN(value) && value !== "") {
        // 截取当前数据到小数点后两位
        realVal = parseFloat(value).toFixed(2);
      } else {
        realVal = "--";
      }
      return realVal;
    },
    dateFilter(da){
      da = new Date(da);
    var year = da.getFullYear()+'年';
    var month = da.getMonth()+1+'月';
    var date = da.getDate()+'日';
    return ([year,month,date].join(' '));
  

    }
  },
//   methods: {
//     handleClose(done) {
//       if (this.loading) {
//         return;
//       }
//       this.$confirm('确定要提交表单吗？')
//         .then(_ => {
//           this.loading = true;
//           this.timer = setTimeout(() => {
//             done();
//             // 动画关闭需要一定的时间
//             setTimeout(() => {
//               this.loading = false;
//             }, 400);
//           }, 2000);
//         })
//     },
//     cancelForm() {
//       this.loading = false;
//       this.dialog = false;
//       clearTimeout(this.timer);
//     }
//   }
}
</script>