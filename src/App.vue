<template>
  <div class="main">
    <el-container>
      <el-header>
        <span>subconverter 订阅生成</span>
      </el-header>

      <el-main>
        <el-form :model="form">
          <el-row style="padding-bottom: 10px">
            <template v-for="(subscribe, index) in form.subscribes" :key="index">
              <el-col :span="10">
                <el-form-item label="订阅地址" prop="subscribes" style="margin: auto; padding-bottom: 10px">
                  <el-input type="textarea" v-model="form.subscribes[index]" placeholder="请输入订阅链接"
                            autosize/>
                </el-form-item>
              </el-col>
              <el-col :span="2">
                <el-button-group size="small">
                  <el-button @click="() => { form.subscribes.splice(index + 1, 0, '')}"
                             :disabled="subscribe === '' || index !== (form.subscribes.length - 1)" :icon="Plus"/>
                  <el-button v-if="form.subscribes.length === 1" @click="() => { form.subscribes[index] = '' }"
                             :icon="CircleClose"/>
                  <el-button v-else @click="() => { form.subscribes.splice(index, 1)}"
                             :disabled="form.subscribes.length === 1"
                             :icon="Delete"/>
                </el-button-group>
              </el-col>
            </template>
          </el-row>

          <el-row :gutter="10">
            <el-col :span="6">
              <el-form-item label="配置地址" prop="config">
                <el-select v-model="form.targetConfig">
                  <el-option v-for="(config, index) in configOptions" :key="index" :label="config.label"
                             :value="config.value"/>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="文件名" prop="fileName">
                <el-input v-model="form.fileName" placeholder="请输入文件名" clearable/>
              </el-form-item>
            </el-col>

            <el-col :span="3">
              <el-form-item label="目标类型" prop="">
                <el-select v-model="form.targetType" placeholder="请选择目标类型">
                  <el-option v-for="(option, index) in targetOptions" :key="index" :label="option.label"
                             :value="option.value"/>
                </el-select>
              </el-form-item>
            </el-col>

            <el-col :span="1.5">
              <el-form-item label="上传" prop="uploadGist">
                <el-tooltip effect="light" content="上传订阅到 gist">
                  <el-switch v-model="form.isUpload"/>
                </el-tooltip>
              </el-form-item>
            </el-col>

            <el-col :span="4">
              <el-form-item label="gist 命名" prop="uploadPath">
                <el-input type="text" v-model="form.gistName" :disabled="!form.isUpload" placeholder="请输入 gist 命名"
                          clearable/>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>

        <el-row>
          <el-col :span="24">
            <span>目标网址</span>
            <el-input type="textarea" v-model="result" readonly autosize/>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <span>编码后目标网址</span>
            <el-input type="textarea" v-model="resultAfterEncode" readonly autosize/>
          </el-col>
        </el-row>
      </el-main>

      <el-footer style="margin-left: auto">
        <el-button type="primary" plain @click="generate">确定</el-button>
        <el-button plain @click="init">重置</el-button>
      </el-footer>
    </el-container>
  </div>
</template>

<script lang="ts">


import {CircleClose, Delete, Plus} from "@element-plus/icons-vue";
import {ElMessage} from "element-plus";

export default {
  computed: {
    CircleClose() {
      return CircleClose
    },
    Plus() {
      return Plus
    },
    Delete() {
      return Delete
    }
  },
  created() {
    this.init();
  },
  data() {
    return {
      form: {
        subscribes: [],
        isUpload: true,
        gistName: null,
        fileName: null,
        targetType: null,
        targetConfig: null,
      },
      configOptions: [
        {
          label: "ACL4SSR_Online_Full_MultiMode",
          value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_MultiMode.ini"
        },
      ],
      targetOptions: [
        {label: "Clash", value: "clash"},
        {label: "SSR", value: "ssr"},
        {label: "Trojan", value: "trojan"},
        {label: "V2Ray", value: "v2ray"},
        {label: "Mixed", value: "mixed"},
        {label: "Auto", value: "auto"}
      ],
      prefix: "http://127.0.0.1:25500/sub",
      result: null,
      resultAfterEncode: null,
    }
  },

  methods: {
    init() {
      this.form.subscribes = [""];
      this.form.isUpload = true;
      this.form.gistName = null;
      this.form.fileName = null;
      this.form.targetType = this.targetOptions[0].value;
      this.form.targetConfig = this.configOptions[0].value;
      this.result = null;
      this.resultAfterEncode = null;
    },

    validateForm() {
      let subscribes = this.form.subscribes.filter((val) => {
        return val !== '' && val !== undefined && val !== null
      });
      if (subscribes.length === 0) {
        return ElMessage.error("订阅地址不能为空")
      }
    },

    generate() {
      let error = this.validateForm();

      if (error === undefined) {
        let url = this.form.subscribes.join("|")
        let urlAfterEncode = encodeURIComponent(url);

        let config = this.form.targetConfig;
        let configAfterEncode = encodeURIComponent(config);

        let prefix = this.prefix;
        let targetType = this.form.targetType;

        let result = `${prefix}?target=${targetType}&url=${url}&config=${config}&interval=0`
        let resultAfterEncode = `${prefix}?target=${targetType}&url=${urlAfterEncode}&config=${configAfterEncode}&interval=0`

        let fileName = this.form.fileName;
        if (fileName !== '' && fileName !== null && fileName !== undefined) {
          result += `&filename=${fileName}`;
          resultAfterEncode += `&filename=${fileName}`;
        }


        let isUpload = this.form.isUpload;
        if (isUpload) {
          let gistName = this.form.gistName;
          let gistNameAfterEncode = encodeURIComponent(gistName);

          result += `&upload=${isUpload}`
          resultAfterEncode += `&upload=${isUpload}`

          if (gistName !== '' && gistName !== null && gistName !== undefined) {
            result += `&upload_path=${gistName}`
            resultAfterEncode += `&upload_path=${gistNameAfterEncode}`
          }
        }

        this.result = result;
        this.resultAfterEncode = resultAfterEncode;
      }
    },
  },
}
</script>

<style scoped>
.main {
  position: absolute;
  top: 10%;
  right: 10%;
  bottom: 10%;
  left: 10%;
  width: 80%;
}
</style>
