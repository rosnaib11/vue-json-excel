<template>
	<div
		:id="id_name"
		@click="generate">
		<slot>
			{{button_text}}
		</slot>
	</div>
</template>

<script>
	import download from 'downloadjs'

export default {
	data: function(){
		return {
			animate   : true,
			animation : '',
		}
	},
	props: {
		'type' : {
			type: String,
			default: "xls"
		},
		'button_text': {
			type: String,
			default: "Download Excel"
		},
		'data':{
			type: Array,
			required: true
		},
		'fields':{
			type: Object,
			required: true
		},
		'name':{
			type: String,
			default: "data.xls"
		},
		'meta':{
			type: Array,
			default: () => []
		}
	},
	created: function () {
	},
	computed:{
		id_name : function(){
			var now = new Date().getTime();
			return 'export_'+now;
		}
	},
	methods: {
		generate() {
			if (this.type == 'csv') {
				return this.exportCSV(this.data, this.name, this.fields);
			}
			return this.exportXLS(this.data, this.name, this.fields);
		},
		generate_excel: function () {
	    	this.exportXLS(this.data, this.name, this.fields)
		},
		jsonToXLS: function (data, header) {
			var xlsTemp = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40"><head><meta name=ProgId content=Excel.Sheet> <meta name=Generator content="Microsoft Excel 11"><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><!--[if gte mso 9]><xml><x:ExcelWorkbook><x:ExcelWorksheets><x:ExcelWorksheet><x:Name>{worksheet}</x:Name><x:WorksheetOptions><x:DisplayGridlines/></x:WorksheetOptions></x:ExcelWorksheet></x:ExcelWorksheets></x:ExcelWorkbook></xml><![endif]--></head><body><table>${table}</table></body></html>'

			var xlsData = '', keys = []
			if (header) {
				xlsData += '<thead>'
				for (var key in header) {
					keys.push(key)
					xlsData += '<th>' + key + '</th>'
				}
				xlsData += '</thead>'
				xlsData += '<tbody>'
				data.map(function (item, index) {
					xlsData += '<tr>'
					for (var i = 0; i < keys.length; i++) {
						if (keys[i].indexOf(".") !== -1) {
							var keyNestedSplit = keys[i].split(".");
							var valueFromNestedKey = item[keyNestedSplit[0]];
						
							for (var j = 1; j < keyNestedSplit.length; j++) {
								valueFromNestedKey = valueFromNestedKey[keyNestedSplit[j]];
							}

							xlsData += '<td>' + valueFromNestedKey + '</td>'
						} else {
							xlsData += '<td>' + item[keys[i]] + '</td>'
						}
					}
					xlsData += '</tr>'
				})
				xlsData += '</tbody>'
				return xlsTemp.replace('${table}', xlsData)
			}
			data.map(function (item, index) {
				xlsData += '<tbody><tr>'
				for (var key in item) {
					xlsData += '<td>' + item[key] + '</td>'
				}
				xlsData += '</tr></tbody>'
			})
			return xlsTemp.replace('${table}', xlsData)
	  },
		jsonToCSV: function (data, header) {
			var csvData = ''
			
			if (header) {
				for (var key in header) {
					csvData +=  key + ','
				}
				csvData = csvData.slice(0, csvData.length - 1)
				csvData += '\r\n'
			}
			data.map(function (item) {
				for (var k in item) {
					var escapedCSV = item[k] + ''; // cast Numbers to string
				 	if (escapedCSV.match(/[,"\n]/)) {
            			 		escapedCSV = '"' + escapedCSV.replace(/\"/g, "\"\"") + '"';
			         	}
				 	csvData += escapedCSV + ',';
				}
				csvData = csvData.slice(0, csvData.length - 1)
				csvData += '\r\n'
			})
			return csvData
	  },
    base64: function (s) {
	    return window.btoa(window.unescape(encodeURIComponent(s)))
    },
    exportXLS: function (data, fileName, header) {
		var XLSData = 'data:application/vnd.ms-excel;base64,' + this.base64(this.jsonToXLS(data, header))
		this.download(XLSData, fileName, 'application/vnd.ms-excel')
	},
    exportCSV: function (data, fileName, keys) {
		var CSVData = 'data:application/csv;base64,' + this.base64(this.jsonToCSV(data, keys))
		this.download(CSVData, fileName, 'application/csv')
	},
	base64ToBlob: function (base64Data) {
		var arr   = base64Data.split(',')
		var mime  = arr[0].match(/:(.*?);/)[1]
		var bstr  = atob(arr[1])
		var n     = bstr.length
		var u8arr = new Uint8ClampedArray(n)

		while (n--) {
			u8arr[n] = bstr.charCodeAt(n)
		}
		return new Blob([u8arr], { type: mime })
	},
	download: function (base64data, fileName, fileType) {
		var blob       = this.base64ToBlob(base64data)

		download(blob, fileName, fileType)
	}//end download
	}
}
</script>
