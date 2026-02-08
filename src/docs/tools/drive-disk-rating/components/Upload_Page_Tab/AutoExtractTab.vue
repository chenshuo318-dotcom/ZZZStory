<template>
  <div class="p-5 mb-5">
    <div
      class="flex gap-4 mb-6 p-4 bg-(--vp-c-bg) border border-(--vp-c-divider) rounded-lg"
    >
      <div
        class="shrink-0 w-8 h-8 bg-(--main-color-1) text-white dark:text-black rounded-full flex items-center justify-center font-semibold text-sm"
      >
        1
      </div>
      <div class="flex-1">
        <strong class="block text-base text-(--vp-c-text-1) mb-2"
          >访问官方页面</strong
        >
        <p class="text-sm text-(--vp-c-text-2) mb-3">
          点击下方按钮打开绝区零角色练度页面，并确保您已登录账号。
        </p>
        <a
          href="https://act.mihoyo.com/zzz/gt/character-builder-h/index.html#/"
          target="_blank"
          class="inline-block px-5 py-2.5 bg-(--main-color-1) text-white! dark:text-black! rounded-md font-medium transition-all duration-200 hover:opacity-90"
        >
          打开角色练度页面
        </a>
      </div>
    </div>

    <div
      class="flex gap-4 mb-6 p-4 bg-(--vp-c-bg) border border-(--vp-c-divider) rounded-lg"
    >
      <div
        class="shrink-0 w-8 h-8 bg-(--main-color-1) text-white dark:text-black rounded-full flex items-center justify-center font-semibold text-sm"
      >
        2
      </div>
      <div class="flex-1">
        <strong class="block text-base text-(--vp-c-text-1) mb-2"
          >添加书签脚本</strong
        >
        <p class="text-sm text-(--vp-c-text-2) mb-3">
          将下方的按钮拖拽到浏览器的书签栏中。
        </p>
        <a
          href="javascript:(async function(){const API_BASE='https://act-api-takumi.mihoyo.com/event/nap_cultivate_tool';const API_LOGIN='https://api-takumi.mihoyo.com/common/badge/v1/login/info';const cleanText=t=>t?.replace(/<[^>]*>/g,'').replace(/\\n/g,'')||'';const fetchJSON=(t,e)=>fetch(t,{credentials:'include',...e}).then(t=>t.json());const getGameUID=async()=>(await fetchJSON(`${API_LOGIN}?game_biz=nap_cn&lang=zh-cn`)).data?.game_uid;const getDeviceFP=()=>document.cookie.match(/DEVICEFP=(\\w+)/)?.[1];const getBasicList=(t,e)=>fetchJSON(`${API_BASE}/user/avatar_basic_list?uid=${t}&region=prod_gf_cn`,{headers:{'x-rpc-device_fp':e}});const getEquipBatch=(t,e,o)=>fetchJSON(`${API_BASE}/user/batch_avatar_detail_v2?uid=${t}&region=prod_gf_cn`,{method:'POST',headers:{'x-rpc-device_fp':o},body:JSON.stringify({avatar_list:e})});const processEquipData=({avatar:t,equip:e,weapon:o})=>({characterName:t.name_mi18n,characterFullName:t.full_name_mi18n,level:t.level,profession:t.avatar_profession,driveDiscs:e?.map(({level:t,name:e,icon:o,rarity:a,invalid_property_cnt:i,equipment_type:s,properties:r,main_properties:n,equip_suit:c})=>({position:s,name:e,level:t,rarity:a,invalidProperty:i,mainProperty:{name:n[0].property_name,val:n[0].base},subProperties:r.map(({property_name:t,base:e,level:o,valid:a,add:i})=>({name:t,val:e,level:o,valid:a,add:i})),suit:{name:c.name,desc1:c.desc1,desc2:cleanText(c.desc2)}}))||[]});const uid=await getGameUID();const device_fp=getDeviceFP();if(!uid||!device_fp){alert('❌ 无法读取 UID 或 DEVICEFP，可能未登录！');return;}const basicData=await getBasicList(uid,device_fp);const avatarList=basicData.data.list.filter(t=>t.unlocked).map(t=>({avatar_id:t.avatar.id}));const batches=[];for(let t=0;t<avatarList.length;t+=10)batches.push(avatarList.slice(t,t+10));const detailResponses=await Promise.all(batches.map(t=>getEquipBatch(uid,t,device_fp)));const allResults=detailResponses.flatMap(t=>t.data.list.map(processEquipData));const result=allResults.map(t=>({...t,discDetails:t.driveDiscs.map(d=>({...d,mainProperty:{...d.mainProperty,value:d.mainProperty?.val||d.mainProperty?.value}}))}));const newWindow=window.open('https://zzzstory.doupoa.site/tools/drive-disk-rating/','_blank');if(newWindow){newWindow.addEventListener('load',()=>{setTimeout(()=>{newWindow.postMessage({type:'ZZZ_CHARACTER_DATA',payload:result},'*');alert('✅ 数据已传输！');},1000);});}else{alert('❌ 无法打开新窗口，请检查弹窗拦截设置。');}})();"
          class="inline-block px-6 py-3 bg-(--main-color-1) text-white! dark:text-black! rounded-lg font-semibold transition-all duration-200 cursor-move hover:opacity-90"
          rel="noopener noreferrer"
        >
          一键提取并传输
        </a>
        <p class="text-sm text-(--vp-c-text-2) mt-3">
          提示：如果看不到书签栏，按
          <code class="bg-(--vp-c-bg-soft) px-1.5 py-0.5 rounded font-mono"
            >Ctrl+Shift+B</code
          >
          (Windows) 或
          <code class="bg-(--vp-c-bg-soft) px-1.5 py-0.5 rounded font-mono"
            >Cmd+Shift+B</code
          >
          (Mac) 显示
        </p>
      </div>
    </div>

    <div
      class="flex gap-4 mb-6 p-4 bg-(--vp-c-bg) border border-(--vp-c-divider) rounded-lg"
    >
      <div
        class="shrink-0 w-8 h-8 bg-(--main-color-1) text-white dark:text-black rounded-full flex items-center justify-center font-semibold text-sm"
      >
        3
      </div>
      <div class="flex-1">
        <strong class="block text-base text-(--vp-c-text-1) mb-2"
          >提取数据</strong
        >
        <p class="text-sm text-(--vp-c-text-2)">
          在官方页面点击书签栏中的按钮，脚本将自动提取数据并传输到本页面。
        </p>
      </div>
    </div>

    <div
      v-if="receivingStatus"
      :class="[
        'flex items-center gap-3 px-4 py-4 rounded-lg mt-5',
        {
          'bg-(--vp-c-info-soft) border border-(--vp-c-info-1) text-(--vp-c-info-1)':
            receivingStatus.type === 'info',
          'bg-(--vp-c-success-soft) border border-(--vp-c-success-1) text-(--vp-c-success-1)':
            receivingStatus.type === 'success',
          'bg-(--vp-c-danger-soft) border border-(--vp-c-danger-1) text-(--vp-c-danger-1)':
            receivingStatus.type === 'error',
        },
      ]"
    >
      <div class="text-2xl">{{ receivingStatus.icon }}</div>
      <div class="font-medium">{{ receivingStatus.text }}</div>
    </div>
    <div
      class="text-center border-t-(--vp-c-divider) mt-[30px] pt-5 border-t border-solid"
    >
      <button
        @click="emit('switch-mode', 'manual')"
        class="text-(--vp-c-text-1) cursor-pointer text-sm px-4 py-2 hover:text-(--main-color-1)! transition-colors"
      >
        无法获取数据？试试手动填写
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

defineEmits(["switch-mode", "data-received"]);

const receivingStatus = ref(null);

const handleMessage = (event) => {
  const allowedOrigins = [
    "https://act.mihoyo.com",
    "https://zzzstory.doupoa.site",
  ];
  if (!allowedOrigins.includes(event.origin)) {
    console.warn("拒绝非信任来源的消息:", event.origin);
    return;
  }

  if (event.data?.type === "ZZZ_CHARACTER_DATA" && event.data?.payload) {
    try {
      const data = event.data.payload;

      if (!Array.isArray(data)) {
        throw new Error("数据格式错误：应为数组");
      }

      receivingStatus.value = {
        type: "success",
        icon: "✓",
        text: `成功接收 ${data.length} 个角色的数据！`,
      };

      setTimeout(() => {
        receivingStatus.value = null;
      }, 3000);

      $emit("data-received", data);
    } catch (e) {
      receivingStatus.value = {
        type: "error",
        icon: "✕",
        text: "数据解析失败：" + e.message,
      };
      console.error("PostMessage 数据解析失败:", e);
    }
  }
};

onMounted(() => {
  window.addEventListener("message", handleMessage);
  receivingStatus.value = {
    type: "info",
    icon: "⏳",
    text: "等待接收数据...请在官方页面点击书签按钮",
  };
});

onUnmounted(() => {
  window.removeEventListener("message", handleMessage);
});
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
