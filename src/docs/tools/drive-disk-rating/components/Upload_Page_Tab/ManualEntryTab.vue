<template>
  <div class="p-5 mb-5">
    <!-- 角色选择 -->
    <CharacterSelector
      v-model="currentCharacterName"
      @change="handleCharacterChange"
    />

    <!-- 驱动盘配置区 -->
    <div class="rounded-lg">
      <div class="text-lg font-semibold my-2 text-black dark:text-gray-200">
        驱动盘配置 (理论满分权重{{ maxPossibleWeight.toFixed(1) }})
      </div>

      <!-- 驱动盘网格 -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <DriveCard
          v-for="slotId in slots"
          :key="slotId"
          :slot-id="slotId"
          :data="driveData[slotId]"
          :character="currentCharacter"
          @update:basic="handleUpdateBasic"
          @update:mainStat="handleUpdateMainStat"
          @update:subStat="handleUpdateSubStat"
        />
      </div>

      <!-- 操作按钮 -->
      <div class="flex justify-center gap-5 mt-5">
        <button
          @click="calculateScore"
          class="px-8! py-2.5! border-none rounded text-sm font-semibold cursor-pointer bg-(--main-color-1)! hover:bg-(--main-color-2)! text-white! dark:text-black! transition-colors"
        >
          计算评分
        </button>
        <button
          @click="resetAll"
          class="px-8! py-2.5! border-none rounded text-sm font-semibold cursor-pointer hover:bg-gray-200! dark:bg-[#2d2d30] dark:hover:bg-gray-800! dark:text-gray-300 transition-colors"
        >
          重置所有
        </button>
      </div>
    </div>

    <!-- 结果面板 -->
    <div
      class="text-lg font-semibold my-2 text-black dark:text-gray-200"
      v-if="showResult"
    >
      评分结果
    </div>
    <ResultPanel
      :visible="showResult"
      :score-result="scoreResult"
      :max-weight="maxPossibleWeight"
      ref="results"
    />

    <!-- 切换提示 -->
    <div
      class="text-center border-t-(--vp-c-divider) mt-[30px] pt-5 border-t border-solid"
    >
      <button
        @click="$emit('switch-mode', 'auto-new')"
        class="text-(--vp-c-text-1) cursor-pointer text-sm px-4 py-2 hover:text-(--main-color-1)! transition-colors"
      >
        填累了？试试自动提取
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick } from "vue";
import CharacterSelector from "../CharacterSelector.vue";
import DriveCard from "../DriveCard.vue";
import ResultPanel from "../ResultPanel.vue";
import {
  getCharacterHighlightSubStats,
  getMainStatOptions,
  buildCharacterConfigs,
} from "../ManualEntryTab_Method_Library.ts";
import { QUALITY_WEIGHTS } from "zzz-drive-disk-rating";

// 定义类型
interface CharacterConfig {
  element: string;
  mainStats: {
    [slot: string]: { [stat: string]: number };
  };
  subStats: { [stat: string]: number };
  highlightSubStats: string[];
}

// 构建角色配置对象
const CHARACTER_CONFIGS = buildCharacterConfigs();

// 工具函数
const getMaxUpgradesForLevel = (level: number) => {
  if (level >= 12) return 5;
  if (level >= 9) return 4;
  if (level >= 6) return 3;
  if (level >= 3) return 2;
  return 1;
};

const getInitialSubSlots = (level: number, remainingUpgrades: number) => {
  const maxUpgrades = getMaxUpgradesForLevel(level);
  if (maxUpgrades >= 5) {
    if (remainingUpgrades <= 0) {
      return 4;
    } else if (remainingUpgrades >= 1) {
      return 3;
    }
  }
  return 3;
};

const initDriveData = () => {
  const slots = ["I", "II", "III", "IV", "V", "VI"];
  const data: any = {};

  slots.forEach((slot) => {
    data[slot] = {
      quality: "S",
      level: 15,
      mainStat: "",
      element: "",
      subStats: Array(4)
        .fill(null)
        .map(() => ({
          name: "",
          upgrades: 0,
        })),
    };
  });

  data["I"].mainStat = "生命值";
  data["II"].mainStat = "攻击力";
  data["III"].mainStat = "防御力";

  return data;
};

// 角色选择
const currentCharacterName = ref("星见雅");
const currentCharacter = computed(() => {
  return CHARACTER_CONFIGS[currentCharacterName.value];
});

// 驱动盘数据
const driveData = ref(initDriveData());

// 部位列表
const slots = ["I", "II", "III", "IV", "V", "VI"];

// 结果显示
const showResult = ref(false);
const scoreResult = ref({
  totalScore: 0,
  grade: "-",
  gradeClass: "grade-f",
  gradeDesc: "",
  validMainCount: 0,
  validSubTotalHits: 0,
  invalidSubUpgrades: 0,
  breakdown: {},
});

// 结果面板
const results = ref<HTMLElement | null>(null);

// 角色切换处理
const handleCharacterChange = (character: CharacterConfig) => {
  showResult.value = false;
};

// 更新基础信息（品质、等级）
const handleUpdateBasic = (slotId: string, field: string, value: string) => {
  driveData.value[slotId][field] = value;
};

// 更新主词条
const handleUpdateMainStat = (slotId: string, value: string) => {
  driveData.value[slotId].mainStat = value;
  driveData.value[slotId].element = "";
};

// 更新副词条
const handleUpdateSubStat = (
  slotId: string,
  index: number,
  field: string,
  value: number | string,
) => {
  const stat = driveData.value[slotId].subStats[index];
  const currentTotalUpgrades = driveData.value[slotId].subStats.reduce(
    (sum: number, s: any) => sum + s.upgrades,
    0,
  );
  const maxAllowedUpgrades = getMaxUpgradesForLevel(
    driveData.value[slotId].level,
  );

  if (field === "name") {
    stat.name = value as string;
    stat.upgrades = 0;
  } else if (field === "upgrades") {
    const numValue = value as number;
    if (numValue > 0) {
      if (stat.upgrades < 5 && currentTotalUpgrades < maxAllowedUpgrades) {
        stat.upgrades += numValue;
      }
    } else {
      if (stat.upgrades > 0) {
        stat.upgrades += numValue;
      }
    }
  }
};

// 动态计算最大权重（总分）
const maxPossibleWeight = computed(() => {
  const char = currentCharacter.value;
  let totalMaxWeight = 0;

  slots.forEach((slot) => {
    const fixedSlots = ["I", "II", "III"];
    const isFixedSlot = fixedSlots.includes(slot);

    if (isFixedSlot) {
      const subStatsAvailable: [string, number][] = (
        Object.entries(char.subStats) as [string, number][]
      )
        .filter(([, weight]) => weight > 0)
        .sort((a, b) => b[1] - a[1]);

      const top4SubStats = subStatsAvailable.slice(0, 4);

      const initialWeight = top4SubStats.reduce(
        (sum: number, [, weight]: [string, number]) => sum + weight,
        0,
      );
      const maxEnhancement =
        top4SubStats.length > 0 ? top4SubStats[0][1] * 5 : 0;

      totalMaxWeight += initialWeight + maxEnhancement;
    } else {
      let slotMaxWeight = 0;
      const availableMainStats = getMainStatOptions(slot);

      availableMainStats.forEach((mainStatName: string) => {
        let mainStatWeight = 0;

        if (mainStatName === "属性伤害%") {
          mainStatWeight = char.mainStats.V
            ? char.mainStats.V["属性伤害%"]
            : 1.0;
        } else {
          if (char.mainStats[slot]) {
            mainStatWeight = char.mainStats[slot][mainStatName] || 0;
          }
        }

        if (mainStatWeight > 0) {
          const mainWeight = mainStatWeight * 3.0;

          const availableSubStats: [string, number][] = (
            Object.entries(char.subStats) as [string, number][]
          )
            .filter(([name]) => name !== mainStatName)
            .filter(([, weight]) => weight > 0)
            .sort((a, b) => b[1] - a[1]);

          const top3SubStats = availableSubStats.slice(0, 3);

          const initialSubWeight = top3SubStats.reduce(
            (sum: number, [, weight]: [string, number]) => sum + weight,
            0,
          );
          const maxSubEnhancement =
            top3SubStats.length > 0 ? top3SubStats[0][1] * 5 : 0;

          const currentSlotWeight =
            mainWeight + initialSubWeight + maxSubEnhancement;
          if (currentSlotWeight > slotMaxWeight) {
            slotMaxWeight = currentSlotWeight;
          }
        }
      });

      totalMaxWeight += slotMaxWeight;
    }
  });

  return totalMaxWeight;
});

// 计算评分
const calculateScore = () => {
  let totalRawWeight = 0;
  let validMainCount = 0;
  let validSubTotalHits = 0;
  let invalidSubUpgrades = 0;
  let breakdown: any = {};

  const char = currentCharacter.value;

  slots.forEach((slot) => {
    const data = driveData.value[slot];
    const qualityMult =
      QUALITY_WEIGHTS[data.quality as keyof typeof QUALITY_WEIGHTS];
    const currentTotalUpgrades = data.subStats.reduce(
      (sum: number, s: any) => sum + s.upgrades,
      0,
    );
    const maxAllowedUpgrades = getMaxUpgradesForLevel(data.level);
    const remainingUpgrades = maxAllowedUpgrades - currentTotalUpgrades;
    const initialSubCount = getInitialSubSlots(data.level, remainingUpgrades);

    let actualWeight = 0;

    if (["IV", "V", "VI"].includes(slot) && data.mainStat) {
      let w = 0;
      if (data.mainStat === "属性伤害%") {
        if (data.element === char.element) {
          w = char.mainStats.V ? char.mainStats.V["属性伤害%"] : 1.0;
        }
      } else {
        if (char.mainStats[slot]) {
          w = char.mainStats[slot][data.mainStat] || 0;
        }
      }
      if (w > 0) validMainCount++;
      actualWeight += w * 3.0;
    }

    data.subStats.forEach((sub: any) => {
      if (sub.name) {
        const isHighlight = char.highlightSubStats.includes(sub.name);
        if (isHighlight) {
          const w = (char.subStats as any)[sub.name] || 0;
          const totalHits = 1 + sub.upgrades;
          validSubTotalHits += totalHits;
          actualWeight += w * totalHits;

          const key = sub.name;
          if (!breakdown[key]) breakdown[key] = 0;
          breakdown[key] += w * totalHits * qualityMult;
        } else {
          const totalHits = 1 + sub.upgrades;
          invalidSubUpgrades += totalHits;
        }
      }
    });

    totalRawWeight += actualWeight * qualityMult;
  });

  const currentMaxWeight = maxPossibleWeight.value;
  const finalScore = (totalRawWeight / currentMaxWeight) * 100;

  let grade = "F";
  let gradeClass = "grade-f";
  let gradeDesc = "可以掰了(是不是没升级？)";

  if (finalScore >= 97) {
    grade = "SSS+";
    gradeClass = "grade-sssp";
    gradeDesc = "极限毕业 (神话盘)";
  } else if (finalScore >= 93) {
    grade = "SSS";
    gradeClass = "grade-sss";
    gradeDesc = "完美毕业 (神盘)";
  } else if (finalScore >= 90) {
    grade = "SS";
    gradeClass = "grade-ss";
    gradeDesc = "大毕业 (极品)";
  } else if (finalScore >= 80) {
    grade = "S";
    gradeClass = "grade-s";
    gradeDesc = "毕业 (好用)";
  } else if (finalScore >= 70) {
    grade = "A";
    gradeClass = "grade-a";
    gradeDesc = "毕业 (标准)";
  } else if (finalScore >= 60) {
    grade = "B";
    gradeClass = "grade-b";
    gradeDesc = "可用 (过渡)";
  } else if (finalScore >= 50) {
    grade = "C";
    gradeClass = "grade-c";
    gradeDesc = "胚子 (需强化)";
  } else if (finalScore >= 40) {
    grade = "D";
    gradeClass = "grade-d";
    gradeDesc = "较差";
  } else if (finalScore >= 30) {
    grade = "E";
    gradeClass = "grade-e";
    gradeDesc = "废弃";
  } else if (finalScore == 0) {
    grade = "?";
    gradeClass = "grade-f";
    gradeDesc = "一个词条都没填评什么？快去填属性！";
  }

  scoreResult.value = {
    totalScore: finalScore,
    grade,
    gradeClass,
    gradeDesc,
    validMainCount,
    validSubTotalHits,
    invalidSubUpgrades,
    breakdown,
  };

  showResult.value = true;

  nextTick(() => {
    if (results.value) {
      results.value.scrollIntoView({
        behavior: "smooth",
        block: "center",
      });
    }
  });
};

// 重置所有
const resetAll = () => {
  driveData.value = initDriveData();
  showResult.value = false;
};
</script>

<style scoped>
.grade-sssp {
  color: #00eeff;
  text-shadow: 0 0 10px rgba(107, 220, 255, 0.5);
}

.grade-sss {
  color: #ff4444;
  text-shadow: 0 0 10px rgba(255, 68, 68, 0.5);
}

.grade-ss {
  color: #ff8c00;
}

.grade-s {
  color: #ffd700;
}

.grade-a {
  color: #c0c0c0;
}

.grade-b {
  color: #cd7f32;
}

.grade-c {
  color: #a0a0a0;
}

.grade-d {
  color: #808080;
}

.grade-e {
  color: #606060;
}

.grade-f {
  color: #404040;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
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
