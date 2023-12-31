<template>
	<div class="zfile-admin-down-link-body">
		<z-form :model="data"
		        v-loading="saveLoading"
		        v-if="data"
		        element-loading-text="保存中...">

      <template #form-title>
        直/短链设置
      </template>
      <template #form-sub-title>
        这里可以配置直/短链相关设置
      </template>

      <z-form-item label="记录下载日志">
        <el-switch v-model="data.recordDownloadLog"/>
        <template #tips>
          是否记录直/短链下载日志
        </template>
      </z-form-item>

      <z-form-item label="是否允许生成/使用直链（路径）">
        <el-switch v-model="data.showPathLink"/>
        <template #tips>
          控制是否生成直链时显示直链路径及是否允许使用直链进行下载.
        </template>
      </z-form-item>

			<z-form-item label="直链地址前缀">
				<el-input v-model="data.directLinkPrefix"></el-input>
				<template #tips>
					直链地址前缀, 如 http(s)://ip:port/<span class="text-red-400 font-bold">{{data.directLinkPrefix}}</span>/path/filename
				</template>
			</z-form-item>

			<z-form-item label="是否允许使用短链">
				<el-switch v-model="data.showShortLink"/>
				<template #tips>
					控制是否生成直链时显示短链路径及是否允许使用短链进行下载
				</template>
			</z-form-item>

      <z-form-item label="短链有效期">
        <TimePicker v-model="data.linkExpireTimes" />
        <template #tips>
          控制生成短链的有效期
        </template>
      </z-form-item>

			<z-form-item label="Referer 防盗链">
				<el-radio v-model="data.refererType" label="off">不启用 Referer 防盗链</el-radio>
				<el-radio v-model="data.refererType" label="white_list">启用白名单</el-radio>
				<el-radio v-model="data.refererType" label="black_list">启用黑名单</el-radio>
				<template #tips>
					防盗链支持访问文件直链或短链时校验，如用户直接访问直链跳转后的存储源原始链接，无法进行校验和拦截。
				</template>
			</z-form-item>

			<z-form-item v-show="data.refererType !== 'no'" label="Referer 配置">
				<el-radio v-model="data.refererAllowEmpty" :label="true">允许 Referer 为空</el-radio>
				<el-radio v-model="data.refererAllowEmpty" :label="false">禁止 Referer 为空</el-radio>
			</z-form-item>

			<z-form-item v-show="data.refererType !== 'no'" label="白名单">
				<el-input type="textarea"
									@input="testAntPathMatcher"
				          :rows="6"
				          v-model="data.refererValue">
				</el-input>
				<template #tips >
          每行输入一个域名，如：
          <ul>
            <li>限制泛域名 <span class="text-red-400">http 协议</span> 全部子目录可访问：<div class="inline select-all font-bold"><span class="text-red-400">http://</span>*example.com/**</div></li>
            <li>限制泛域名 <span class="text-red-400">https 协议</span> 全部子目录可访问：<div class="inline select-all font-bold"><span class="text-red-400">https://</span>*example.com/**</div></li>
            <li>限制泛域名 <span class="text-red-400">全部协议</span> 全部子目录可访问：<div class="inline select-all font-bold"><span class="text-red-400">*://</span>*example.com/**</div></li>
          </ul>
          <div v-if="data.refererType === 'white_list'">
            每行输入一个域名，需要写协议头支持 * 通配符，如白名单 http://*zfile.vip 将只允许 http://zfile.vip、http://www.zfile.vip、http://demo.zfile.vip 等网站访问。
          </div>
          <div v-if="data.refererType === 'black_list'">
            每行输入一个域名，需要写协议头，支持 * 通配符，如黑名单 http://*zfile.vip 将禁止所有如 http://zfile.vip、http://www.zfile.vip、http://demo.zfile.vip 等网站访问。
          </div>

          <div class="mt-4">
						<div>可在下方输入你要测试的地址（应包含协议，路径，从浏览器地址栏复制就是了）</div>
						<el-input @input="testAntPathMatcher" v-model="testAntVal">
							<template #suffix>
								<CheckCircleIcon class="w-4 text-green-500" v-if="testAntVal && testAntResult === true"></CheckCircleIcon>
								<XCircleIcon class="w-4 text-red-500" v-else-if="testAntVal && testAntResult === false"></XCircleIcon>
							</template>
						</el-input>
          </div>
				</template>
			</z-form-item>

      <!--设置直链防止恶意下载，单 IP N 秒内只允许访问 M 次直链-->
      <z-form-item label="直链下载限制">
        <span>单 IP</span>
        <el-input-number v-model="data.linkLimitSecond" :min="0" :max="86400" :step="1" size="small" class="mx-2"/>
        <span>秒内允许下载</span>
        <el-input-number v-model="data.linkDownloadLimit" :min="0" :max="9999999" :step="1" size="small" class="mx-2" />
        <span>次</span>
        <template #tips>
          <span>设置直链防止恶意下载，单 IP N 秒内只允许访问 M 次直链，如其中一个为 0 则不做任何限制.</span>
          <br><br>
          <span>注意：此功能对直链、短链都有效，且共享限制次数。但直链/短链跳转后的实际下载链接无法限制，因为那些链接不经过 ZFile.</span>
          <br><br>
          <span>注意：如果你使用了反向代理，而不是直接访问的 ZFile 端口, 那你需要在反向代理处设置以下请求头为用户真实 IP："X-Forwarded-For", "X-Real-IP", "Proxy-Client-IP", "WL-Proxy-Client-IP", "HTTP_CLIENT_IP", "HTTP_X_FORWARDED_FOR"，不然不论谁访问 ZFile 都只能获取到反代服务器的 IP，同服务器一般是获取到 127.0.0.1 或 localhost, 这样此功能就无法正常使用!!!  (辅助信息: 系统获取到您当前 IP 为 <span class="text-blue-400">{{clientIp}}</span>)</span>
        </template>
      </z-form-item>

			<template #footer>
				<span class="dialog-footer">
					<el-button type="primary" @click="saveData">保存</el-button>
				</span>
			</template>
		</z-form>
	</div>
</template>


<script setup>
import { CheckCircleIcon, XCircleIcon } from "@heroicons/vue/24/solid";


import useLinkSetting from "~/composables/admin/link/useLinkSetting";
const { data, saveData, saveLoading } = useLinkSetting();

const clientIp = ref('');

import { getClientIpReq, testAntPathMatcherReq } from "~/api/admin-setting";
getClientIpReq().then(res => {
  clientIp.value = res.data.data;
})


const testAntVal = ref('');
const testAntResult = ref(null);


const testAntPathMatcher = useDebounceFn(() => {
	let param = {
		antPath: data?.value?.refererValue,
		testPath: testAntVal.value
	}
	testAntPathMatcherReq(param).then(res => {
		testAntResult.value = res.data.data;
	});
}, 250);




</script>
