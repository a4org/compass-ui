<template>
    <div>
    <div>
    <input type="text" v-model="searchid" placeholder="fill in your id">
    <el-button size="mini" type="primary" plain @click = "show()">Enter</el-button>
    </div>
    <div>
    <el-table
	id="HistoryTable"
	:data= "sstudent"
	style="width: 100%"
	:row-style="{height: '30px'}"
	:cell-style="{padding: '0'}"
	:header-row-style="{height: '40px'}"
	:header-cell-style="{padding: '10px'}"
	stripe
	height="calc(100% - 1.8rem)"
	v-if="ShowTable">
	<el-table-column
	    align="center"
	    prop="Acct"
	    label="Account#"
	    width="150">
	    <template slot="header">
		<div style="font-size: 13px; line-height: 1.3">Account#</div>
	    </template>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="Time"
	    label="Date/Time"
	    width="150"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="Asset"
	    label="Asset Class"
	    width="150"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="Code"
	    label="Ticker Code"
	    width="150"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="Currency"
	    label="Currency"
	    width="150"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="Qty"
	    label="Quantity Traded"
	    width="200"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="AVG"
	    label="FX Average"
	    width="150"
	    sortable>
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="EOD"
	    label="Executed Price"
	    width="200">
	</el-table-column>
	<el-table-column
	    align="center"
	    prop="SUM"
	    label="Sum of Sec"
	    width="150"
	    sortable>
	</el-table-column>
    </el-table>
    </div>
    </div>

</template>

<script>
import XLSX from 'xlsx-style';

export default {
    name: 'HistoryTable',

    methods: {
	show: function() {
	    
	    this.sstudent = [];

	    if (this.searchid == '') {
		// Show all of them
		this.studentdata.forEach(student => {
		    this.sstudent.push(student);
		});
	    } else {
		var idtotal = this.searchid + " Total"
		this.studentdata.forEach(student => {
		    if (student.Acct == this.searchid || student.Acct == idtotal) {
			this.sstudent.push(student);
		    }
		});
	    }

	    // show table
	    this.ShowTable = true;
	},

	ExcelDateToJSDate: function(date) {
	  var ret = '';
	  if (date == '') return ret;
	  var date = new Date(Math.round((date - 25569)*86400*1000));
	  var year = date.getFullYear();
	  var month = date.getMonth() + 1;
	  var day = date.getDate();
	  ret += year.toString() + '-' + month.toString() + '-' + day.toString();
	  return ret;
	},

	FormatNum: function(num) {
	    if (num == '') return '';
	    // 1. to int
	    var ret = '';
	    var nums = '';
	    num = num.toString();

	    for (let i = 0; i < num.length; i++) {
		if (num[i] == '.') break;
		nums += num[i];
	    }

	    // 2. add point

	    var tri = 0;
	    for (let i = nums.length-1; i >= 0; i--) {
		ret += nums[i];
		tri++;

		if (tri == 3) {
		    if (i == 0) {
			break;
		    }
		    let tri = 0;
		    ret += ',';
		}
	    }

	    ret = this.reverse(ret);

	    return ret;
	},

	reverse: function(s){
	    return s.split("").reverse().join("");
	},
    },

    data() {
	return {
	    searchid: '',
	    next: true,
	    currstudent: '',  // deal with each student
	    studentdata: [],
	    sstudent: [],
	    ShowTable: false,
	}
    },

    mounted() {
	this.axios.get("./sheets/PnlDB.xlsx", {
	// this.axios.get("./sheets/traderecord.xlsx", {
	    responseType: "arraybuffer",
	    headers: {
		'Cache-control': 'no-cache',
		'Pragma': 'no-cache',
		'Expires': '0',
	    }
	}).then(resp => {
	    let data = new Uint8Array(resp.data);
	    let wb = XLSX.read(data, {type: 'array', cellStyles: true});
	    let ws = wb.Sheets[wb.SheetNames[2]];

	    let firstEntryRow = 6; // start index

	    // let change = false; // whether we need to change person
	    // this.next = true;


	    for (let i = firstEntryRow; ws[`A${i}`] != undefined; ++i) {
		// read each student 

		/*
		let pn = ws[`A${i}`]; // person or null

		if (pn.v == '(blank)') {
		    break;
		}

		if (this.next) {
		    // change student
		    this.currstudent = pn.v;
		    // console.log("Change!");
		    this.next = false;
		}

		if (pn.v != '') {
		    change = !change;
		    // console.log(change);
		}
		*/

		let student = {
		    Acct: ws[`A${i}`].v,
		    Time: this.ExcelDateToJSDate(ws[`B${i}`].v),
		    Asset: ws[`C${i}`].v,
		    Code: ws[`D${i}`].v,
		    Currency: ws[`E${i}`].v,
		    Qty: ws[`F${i}`].v,
		    AVG: ws[`G${i}`].v,
		    EOD: ws[`H${i}`].v,
		}

		this.studentdata.push(student);
	    }

	});

    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a {
  color: #42b983;
}

.el-checkbox {
  margin-right: 15px;
}

/* element-ui's bug */
#HistoryTable >>> .el-table__header-wrapper {
  height: 40px;
}
#HistoryTable >>> .el-table__body-wrapper {
  height: calc(100% - 40px) !important;
}
</style>
