<template>
	<div>
		<!--limit:最大上传数，on-exceed：超过最大上传数量时的操作  on-change:上传成功和失败都会执行-->
		<el-upload
		    class="upload-demo"
		    action=""
		    :on-change="handleChange"
		    :on-remove="handleRemove"
		    :on-exceed="handleExceed"
		    :limit="limitUpload"
		    accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"
		    :auto-upload="false">
		<el-button size="small" type="primary">点击上传</el-button>
		<div slot="tip" class="el-upload__tip">只 能 上 传 xlsx / xls 文 件</div>
		</el-upload>
		
		<!-- 数据展示 -->
		  <el-main>
		      <el-table :data="da">
		        <el-table-column prop="code" label="编号">
		        </el-table-column>
		        <el-table-column prop="name" label="姓名">
		        </el-table-column>
		        <el-table-column prop="pro" label="省份">
		        </el-table-column>
		        <el-table-column prop="cit" label="城市">
		        </el-table-column>
		        <el-table-column prop="dis" label="区县">
		        </el-table-column>
		      </el-table>
		    </el-main>
	</div>
</template>

<script>
	export default{
		data(){
			return{
				limitUpload:1,
				fileTemp:null,
				file:null,
				da:[],
				dalen:0
			};
		},
		methods:{
			// 添加文件和上传成功和失败都会执行该文件
			handleChange(file, fileList){ 
				console.log("handleChangec参数：",file);
			    this.fileTemp = file.raw; //获取上传文件的头部的信息
			    if(this.fileTemp){ //判断头部是不是为空
			        if((this.fileTemp.type == 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') 
			            || (this.fileTemp.type == 'application/vnd.ms-excel')){ //判断头部是否符合设置
			            this.importfxx(this.fileTemp); //调用文件上传方法
			        } else { //文件格式不对时，提示信息
			            this.$message({
			                type:'warning',
			                message:'附件格式错误，请删除后重新上传！'
			            })
			        }
			    } else { //文件头部是空
			        this.$message({
			            type:'warning',
			            message:'请上传附件！'
			        })
			    }
			},
			//超出文件最大的数量，有关参数limitUpload
			handleExceed(){
			    this.$message({
			        type:'warning',
			        message:'超出最大上传文件数量的限制！'
			    })
			    return;
			},
			//文件移除时的钩子
			handleRemove(file,fileList){
			    this.fileTemp = null
			},
			//文件上传方法
			importfxx(obj) {
			    let _this = this; //声明一个变量指向Vue实例this，保证作用域一致,_this只是一个变量名，this代表父函数，如果在子函数还用this，this的指向就变成子函数了，_this就是用来存储指向的。
			    let inputDOM = this.$refs.inputer; //通过ref绑定的组件，直接访问demo，减少消耗。
			    // 通过DOM取文件数据
				
			    this.file = event.currentTarget.files[0];
				
			    var rABS = false; //是否将文件读取为二进制字符串
			    var f = this.file; //将父类的file，赋值给f
			
			    var reader = new FileReader();
				
			    FileReader.prototype.readAsBinaryString = function(f) {
			        var binary = "";
			        var rABS = false; //是否将文件读取为二进制字符串
			        var pt = this;
			        var wb; //读取完成,转换后数据
			        var outdata;//用来存储原始数据
			        var reader = new FileReader();
					//读取文件readAsBinaryString，会触发一个load时间，使用reader.onload对该事件进行处理
			        reader.onload = function(e) {
						//reader.result:返回文件的内容,只有读取操作后，该属性才能生效
			            var bytes = new Uint8Array(reader.result); // Uint8Array: 8位无符号整数，长度1个字节
			            var length = bytes.byteLength;
			            for (var i = 0; i < length; i++) {
			                binary += String.fromCharCode(bytes[i]); //返回由指定的 UTF-16 代码单元序列创建的字符串
			            }
			            var XLSX = require("xlsx");
			            if (rABS) {
			                wb = XLSX.read(btoa(fixdata(binary)), {
			                //手动转化
			                type: "base64"
			                });
			            } else {
			                wb = XLSX.read(binary, {
			                type: "binary"
			                });
			            }
			            outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]); //outdata就是你想要的东西
			            console.log('未处理的原始数据如下：');
			            console.log(outdata);
			            //此处可对数据进行处理
			            let arr = [];
						// outdata.map将outdata中的数据循环并且放置在arr中进行输出
			            outdata.map(v => {
			                let obj = {}
			                obj.code = v['Code']
			                obj.name = v['Name']
			                obj.pro = v['province']
			                obj.cit = v['city']
			                obj.dis = v['district']
			                arr.push(obj)
			            });
			            _this.da=arr;
			            _this.dalen=arr.length;
			            return arr;
			        };
			        reader.readAsArrayBuffer(f);
			    };
			    if (rABS) {
			        reader.readAsArrayBuffer(f);
			    } else {
			        reader.readAsBinaryString(f);
			    }
			}
			
			
		}
		
	}
</script>


<style>
</style>
