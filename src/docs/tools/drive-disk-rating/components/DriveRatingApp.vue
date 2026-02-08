<template>
  <div class="py-5 min-h-screen">
    <header class="text-center mb-4">
      <p class="text-2xl font-bold text-(--vp-c-text-1)">驱动盘评分</p>
      <span class="text-sm text-(--vp-c-text-2)">测试版本，欢迎反馈</span>
    </header>

    <!-- 模式切换 -->
    <template v-if="characters.length === 0">
      <div class="flex justify-center gap-5">
        <button
          v-for="mode in modes"
          :key="mode.id"
          @click="switchMode(mode.id)"
          :class="[
            'px-5 py-2.5 cursor-pointer text-base font-medium ',
            {
              'text-black! dark:text-white! font-semibold border-b-3! border-(--main-color-1)!':
                currentMode === mode.id,
              'text-gray-400! hover:text-(--main-color-1)!':
                currentMode !== mode.id,
            },
          ]"
        >
          {{ mode.name }}
        </button>
      </div>
      <div class="relative">
        <transition name="fade" mode="out-in">
          <AutoExtractTab
            class="md:px-96"
            v-if="currentMode === 'auto-new'"
            @switch-mode="switchMode"
            @data-received="handleDataReceived"
          />
          <UploadFileTab
            class="md:px-96"
            v-else-if="currentMode === 'auto-upload'"
            @switch-mode="switchMode"
            @data-received="handleDataReceived"
          />
          <ManualEntryTab
            class="md:px-16"
            v-else-if="currentMode === 'manual'"
            @switch-mode="switchMode"
          />
        </transition>
      </div>
    </template>

    <!-- 评分结果展示 -->
    <transition name="fade" mode="out-in">
      <ResultStep
        v-if="characters.length > 0"
        :characters="characters"
        :selected-character="selectedCharacter"
        :on-character-change="handleCharacterChange"
        @reset="handleReset"
      />
    </transition>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { calculateCharacterTotalScore } from "zzz-drive-disk-rating";
import AutoExtractTab from "./Upload_Page_Tab/AutoExtractTab.vue";
import UploadFileTab from "./Upload_Page_Tab/UploadFileTab.vue";
import ManualEntryTab from "./Upload_Page_Tab/ManualEntryTab.vue";
import ResultStep from "./ResultStep.vue";

const currentMode = ref("auto-new");
const characters = ref([]);
const showCalculation = ref(false);
const selectedCharacter = ref("星见雅");
const allCharacterData = ref([]);

const modes = [
  { id: "auto-new", name: "自动提取" },
  { id: "auto-upload", name: "上传文件" },
  { id: "manual", name: "手动填写" },
];

const switchMode = (mode) => {
  currentMode.value = mode;
  characters.value = [];
  showCalculation.value = false;
};

const handleDataReceived = (data) => {
  // 数据打包逻辑：按照 types.ts 定义进行转换
  const packedData = data.map((item) => {
    // 转换为 CharacterData 类型
    const characterData = {
      characterName: item.characterName || "",
      characterFullName: item.characterFullName || "",
      level: Number(item.level) || 0,
      profession: Number(item.profession) || 0,
      driveDiscs: (item.driveDiscs || []).map((disc) => {
        // 转换为 DriveDisc 类型
        return {
          position: Number(disc.position) || 0,
          name: disc.name || "",
          level: Number(disc.level) || 0,
          rarity: disc.rarity || "S",
          invalidProperty:
            Number(disc.invalidProperty || disc.invalid_property_cnt) || 0,
          mainProperty: {
            // 转换为 MainProperty 类型
            name:
              disc.mainProperty?.name || disc.mainProperty?.property_name || "",
            value: String(
              disc.mainProperty?.value || disc.mainProperty?.val || "",
            ),
          },
          subProperties: (disc.subProperties || []).map((sub) => {
            // 转换为 SubProperty 类型
            return {
              name: sub.name || sub.property_name || "",
              value: String(sub.value || sub.val || ""),
              level: Number(sub.level) || 0,
              valid: Boolean(sub.valid) || false,
              add: Number(sub.add) || 0,
            };
          }),
        };
      }),
    };
    return characterData;
  });

  allCharacterData.value = packedData;

  // 如果上传的数据不为空，选择第一个角色
  if (packedData.length > 0) {
    const firstCharacterName = packedData[0].characterName;
    selectedCharacter.value = firstCharacterName;
    switchCharacter(firstCharacterName);
  }
};

const switchCharacter = (characterName) => {
  const targetCharacter = allCharacterData.value.find(
    (c) => c.characterName === characterName,
  );

  if (!targetCharacter) {
    characters.value = [];
    return;
  }

  try {
    if (
      !targetCharacter.driveDiscs ||
      targetCharacter.driveDiscs.length === 0
    ) {
      characters.value = [
        {
          ...targetCharacter,
          totalScore: 0,
          averageScore: 0,
          discScores: {},
          discDetails: [],
        },
      ];
      return;
    }

    // 使用评分算法计算评分
    const result = calculateCharacterTotalScore(targetCharacter);

    // 计算平均分
    const totalDiscs = targetCharacter.driveDiscs.length;
    const averageScore = totalDiscs > 0 ? result.totalScore / totalDiscs : 0;

    // 确保所有6个位置都有数据
    const discDetails = [];
    const existingPositions = new Set(
      targetCharacter.driveDiscs.map((disc) => disc.position),
    );

    // 添加已装备的驱动盘
    result.discDetails.forEach((disc) => {
      discDetails.push({
        ...disc,
        details: {
          ...disc.details,
          validProperties: disc.details.validProperties || [],
        },
      });
    });

    // 添加未装备的驱动盘位置
    for (let i = 1; i <= 6; i++) {
      if (!existingPositions.has(i)) {
        discDetails.push({
          position: i,
          name: "未装备",
          level: 0,
          rarity: "-",
          mainProperty: { name: "无", value: 0 },
          score: 0,
          details: {
            score: 0,
            subPropertiesWeight: 0,
            mainPropertyWeight: 0,
            maxWeightSum: 0,
            maxWeightFormula: "",
            validProperties: [],
            qualityWeight: 0,
            levelWeight: 0,
          },
        });
      }
    }

    // 按位置排序
    discDetails.sort((a, b) => a.position - b.position);

    characters.value = [
      {
        ...targetCharacter,
        totalScore: result.totalScore,
        averageScore: Math.round(averageScore * 10) / 10, // 保留一位小数
        discScores: result.discScores,
        discDetails,
      },
    ];
  } catch (err) {
    console.error(`处理角色 "${characterName}" 时出错:`, err);
    characters.value = [];
  }
};

const handleCharacterChange = (event) => {
  const characterName = event.target.value;
  selectedCharacter.value = characterName;
  switchCharacter(characterName);
};

const handleReset = () => {
  // 重置状态
  characters.value = [];
  allCharacterData.value = [];
  selectedCharacter.value = "星见雅";
  // 切换回自动提取模式
  currentMode.value = "auto-new";
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
