<template>
  <div id="app">
    <h1>{{ message }}</h1>

    <div>
      <h3>{{ form_label.header }}</h3>

      <div>
        <label>{{ form_label.project_name }}:</label>
        <input v-model.lazy="settings.name" placeholder="The Origin" />
      </div>

      <div>
        <label>{{ form_label.principle }}:</label>
        <input
          v-model.lazy.number="settings.principle"
          placeholder="1000000"
          type="number"
        />
      </div>
      <div v-show="handle.showSetting">
        <div>
          <h3>{{ form_label.rate_table }}</h3>
          <table v-show="settings.interest_rate.length > 0">
            <thead>
              <tr>
                <th>Month</th>
                <th>Rate / Month</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rate, ind) in settings.interest_rate" :key="ind">
                <td>{{ rate.month }}</td>
                <td>{{ rate.rate }}%</td>
                <td>
                  <button @click="deleteRate(ind)">delete</button>
                </td>
              </tr>
            </tbody>
          </table>

          <div>
            Month from
            <input
              v-model.number="form_tmp.month_form"
              placeholder="1"
              type="number"
            />
            to
            <input
              v-model.number="form_tmp.month_to"
              placeholder="12"
              type="number"
            />
            with rate
            <input
              v-model.number="form_tmp.with_rate"
              placeholder="12"
              type="number"
            />
            <button @click="addRate()">add rate</button>
          </div>
        </div>
        <!-- <---------------------->
        <div>
          <h3>{{ form_label.paid_table }}</h3>
          <table v-show="settings.paid.length > 0">
            <thead>
              <tr>
                <th>Month</th>
                <th>Baht / Month</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, ind) in settings.paid" :key="ind">
                <td>{{ item.month }}</td>
                <td>{{ item.amount | digits }}</td>
                <td>
                  <button @click="deletePaid(ind)">delete</button>
                </td>
              </tr>
            </tbody>
          </table>

          <div>
            Month from
            <input
              v-model.number="form_tmp.paid_month_form"
              placeholder="1"
              type="number"
            />
            to
            <input
              v-model.number="form_tmp.paid_month_to"
              placeholder="12"
              type="number"
            />
            with amount
            <input
              v-model.number="form_tmp.paid_amount"
              placeholder="12"
              type="number"
            />
            <button @click="addPaid()">add amount</button>
          </div>
        </div>
      </div>
    </div>

    <button @click="handle.showSetting = !handle.showSetting">
      <span v-show="!handle.showSetting">Show Settings</span>
      <span v-show="handle.showSetting">Hide Settings</span>
    </button>

    <button @click="calReport()">Report</button>
    <button @click="calEnd()">Find End</button>
    <button @click="save()">Save</button>

    <div>
      <table v-show="calResult.report.length > 0">
        <thead>
          <tr>
            <th>งวดที่</th>
            <th>อัตราดอกเบี้ย</th>
            <th>จ่าย</th>
            <th>จ่ายเงินต้น</th>
            <th>จ่ายดอกเบี้ย</th>
            <th>เหลือหนี้</th>
            <th>ระยะหนี้</th>
            <th>เงินที่เสียทั้งหมด</th>
            <th>คิดเป็น % เงินต้น</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(report, ind) in calResult.report" :key="ind">
            <td>{{ report.month }}</td>
            <td>{{ report.rate }}%</td>
            <td>{{ report.paid_total | digits }}</td>
            <td>{{ report.paid_principle | digits }}</td>
            <td>{{ report.paid_interest | digits }}</td>
            <td>{{ report.left | digits }}</td>
            <td>{{ report.sYear }} ปี {{ report.sMonth }} เดือน</td>
            <td>{{ report.total_pay | digits }}</td>
            <td>{{ report.total_in_percent | digits }}%</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="Object.keys(calResult.end).length > 0">
      ครบดอกเบี้ยในปีที่ {{ calResult.end.sYear }} เดือนที่
      {{ calResult.end.sMonth }}
      <br />
      ใช้เงินทั้งสิ้น {{ calResult.end.total_pay | digits }}
      <br />
      คิดเป็น {{ calResult.end.total_in_percent | digits }}% จากเงินต้น
      {{ settings.principle | digits }}
    </div>

    <div v-show="calResult.link != ''">
      Copy this link to save: {{ calResult.link }}
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      message: "Interest Calculator!",

      form_tmp: {
        month_to: "",
        month_form: "",
        with_rate: "",
        paid_month_to: "",
        paid_month_form: "",
        paid_amount: ""
      },

      form_label: {
        header: "ตั้งค่า",
        project_name: "ชื่อโครงการ",
        principle: "เงินต้น",
        rate_table: "กำหนดอัตราดอกเบี้ย",
        paid_table: "กำหนดค่างวด"
      },

      handle: {
        showSetting: false
      },

      settings: {
        name: "", // e.g. Miti Chiva
        principle: 0, // 100000
        interest_rate: [],
        paid: []
      },

      calResult: {
        paid: {},
        report: [],
        end: {},
        link: "",
      }
    };
  },

  filters: {
    digits: function (value) {
      if (!value) return "N/A";
      value = parseFloat(value).toFixed(2);
      return new Intl.NumberFormat().format(value);
    }
  },

  mounted: function () {
    let urlSearchParams = new URLSearchParams(window.location.search);
    let params = Object.fromEntries(urlSearchParams.entries());

    if (params.load != undefined) {
      this.settings = JSON.parse(decodeURI(params.load));
    }
  },

  methods: {
    save() {
      let saveLink = window.location.origin;
      let data = encodeURI(JSON.stringify(this.settings));
      saveLink = saveLink + "?load=" + data;
      this.calResult.link = saveLink;
    },

    calReport() {
      this.calResult.end = {};
      this.calResult.report = this.getReport();
    },

    calEnd() {
      this.calResult.report = [];
      this.calPaid();
      let stMonth,
        left,
        month,
        rate,
        paidInterest,
        paidPrinciple,
        totalPay,
        totalInPercent;
      let latest = this.getReport().pop();

      if (latest.left > 0) {
        while (latest.left > 0) {
          paidInterest = ((latest.left * (latest.rate / 100)) / 365) * 30;
          paidPrinciple = latest.paid_total - paidInterest;
          left = latest.left - paidPrinciple;
          month = latest.month + 1;

          totalPay = latest.total_pay + latest.paid_total;
          totalInPercent = (totalPay * 100) / this.settings.principle;
          latest = {
            month: month,
            rate: latest.rate,
            paid_total: latest.paid_total,
            paid_interest: paidInterest,
            paid_principle: paidPrinciple,
            left: left,
            sYear: parseInt(month / 12),
            sMonth: month % 12,
            total_pay: totalPay,
            total_in_percent: totalInPercent
          };
        }
      }

      this.calResult.end = latest;
    },

    getReport() {
      this.calPaid();

      let rate,
        paidTotal,
        paidInterest,
        paidPrinciple,
        totalInPercent,
        minM,
        maxM;
      let result = [];
      let month = 1;
      let left = this.settings.principle;
      let totalPay = 0;

      this.settings.interest_rate.every((item) => {
        let breakThis = false;
        let months = item.month.replaceAll(" ", "").split("-");
        if (months.length == 1) {
          minM = months[0];
          maxM = months[0];
        } else {
          minM = months[0];
          maxM = months[1];
        }

        for (let i = minM; i <= maxM; i++) {
          month = i;
          rate = item.rate;
          paidTotal = this.calResult.paid[month];
          paidInterest = ((left * (rate / 100)) / 365) * 30;
          paidPrinciple = paidTotal - paidInterest;
          left = left - paidPrinciple;

          totalPay = totalPay + paidTotal;
          totalInPercent = (totalPay * 100) / this.settings.principle;
          result.push({
            month: month,
            rate: rate,
            paid_total: paidTotal,
            paid_interest: paidInterest,
            paid_principle: paidPrinciple,
            left: left,
            sYear: parseInt(month / 12),
            sMonth: month % 12,
            total_pay: totalPay,
            total_in_percent: totalInPercent
          });

          if (left <= 0) {
            breakThis = true;
            break;
          }
        }

        if (breakThis) {
          return false;
        }

        return true;
      });

      return result;
    },

    calPaid() {
      let minM, maxM;
      this.settings.paid.forEach((item) => {
        let months = item.month.replaceAll(" ", "").split("-");
        if (months.length == 1) {
          minM = months[0];
          maxM = months[0];
        } else {
          minM = months[0];
          maxM = months[1];
        }

        for (let i = minM; i <= maxM; i++) {
          this.calResult.paid[i] = item.amount;
        }
      });
    },

    deletePaid(ind) {
      alert("Paid " + ind + " deleted");
      this.settings.paid.splice(ind, 1);
      return true;
    },

    addPaid() {
      let paid_month_to = this.form_tmp.paid_month_to;
      let paid_month_form = this.form_tmp.paid_month_form;
      let paid_amount = this.form_tmp.paid_amount;

      if (paid_month_to < paid_month_form) {
        alert('Cannot set "month from" less than "month to"');
        return false;
      }

      let month = `${paid_month_form}-${paid_month_to}`;
      if (paid_month_form === paid_month_to) {
        month = paid_month_form + "";
      }

      this.settings.paid.push({
        month: month,
        amount: paid_amount
      });
    },

    deleteRate(ind) {
      alert("Rate " + ind + " deleted");
      this.settings.interest_rate.splice(ind, 1);
      return true;
    },

    addRate() {
      let fromMonth = this.form_tmp.month_form;
      let toMonth = this.form_tmp.month_to;
      let rate = this.form_tmp.with_rate;
      console.log(fromMonth);
      console.log(this.form_tmp);

      if (toMonth < fromMonth) {
        alert('Cannot set "month from" less than "month to"');
        return false;
      }

      if (rate == "" || rate == 0) {
        alert("Cannot set rate to 0");
        return false;
      }

      let month = `${fromMonth}-${toMonth}`;
      if (fromMonth === toMonth) {
        month = fromMonth + "";
      }

      this.settings.interest_rate.push({
        month: month,
        rate: rate
      });

      return true;
    }
  }
};
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

a,
button {
  color: #4fc08d;
}

button {
  background: none;
  border: solid 1px;
  border-radius: 2em;
  font: inherit;
  padding: 0.75em 2em;
}
</style>
