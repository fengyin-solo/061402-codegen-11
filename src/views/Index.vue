<template>
  <div class="island-container">
    <div class="island-header">
      <h1>🏝️ 海岛生存</h1>
      <p>在荒岛上建立你的生存基地</p>
    </div>
    
    <div class="island-main">
      <div class="stats-panel">
        <div class="stat-card">
          <div class="stat-icon">🍖</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.food }}</div>
            <div class="stat-label">食物</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">💧</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.water }}</div>
            <div class="stat-label">淡水</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">🪵</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.wood }}</div>
            <div class="stat-label">木材</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">⛏️</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.stone }}</div>
            <div class="stat-label">石头</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-icon">🗺️</div>
          <div class="stat-content">
            <div class="stat-number">{{ exploredCount }}/{{ mapGrid.length }}</div>
            <div class="stat-label">探索进度</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-icon">✨</div>
          <div class="stat-content">
            <div class="stat-number">+{{ totalBonus }}%</div>
            <div class="stat-label">采集加成</div>
          </div>
        </div>
      </div>

      <div class="area-info-panel" v-if="selectedCell !== null">
        <h3>📍 区域信息</h3>
        <div class="area-info-content">
          <div class="area-info-header">
            <span class="area-info-icon">{{ getCellDisplay(mapGrid[selectedCell]) }}</span>
            <span class="area-info-name">{{ getCellDisplayName(mapGrid[selectedCell]) }}</span>
            <span class="area-info-status" :class="mapGrid[selectedCell].explored ? 'explored' : 'unexplored'">
              {{ mapGrid[selectedCell].explored ? '已解锁' : '未探索' }}
            </span>
          </div>
          <div class="area-info-details" v-if="mapGrid[selectedCell].explored">
            <div class="info-row">
              <span class="info-label">地形描述：</span>
              <span class="info-value">{{ mapGrid[selectedCell].description }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">风险等级：</span>
              <span class="info-value risk-{{ mapGrid[selectedCell].riskLevel }}">
                {{ getRiskLabel(mapGrid[selectedCell].riskLevel) }}
              </span>
            </div>
            <div class="info-row">
              <span class="info-label">主要资源：</span>
              <span class="info-value">{{ mapGrid[selectedCell].primaryResources.join('、') }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">采集加成：</span>
              <span class="info-value bonus-positive">
                +{{ mapGrid[selectedCell].bonus }}% ({{ getBonusDescription(mapGrid[selectedCell]) }})
              </span>
            </div>
          </div>
          <div class="area-info-details area-fog" v-else>
            <div class="fog-warning">
              <span class="fog-icon">❓</span>
              <p>这片区域被迷雾笼罩，风险和奖励未知...</p>
              <p class="fog-hint">点击地图地块进行探索以解锁区域信息</p>
            </div>
          </div>
        </div>
      </div>
      
      <div class="actions-panel">
        <h3>📋 可执行操作</h3>
        
        <div class="action-grid">
          <div class="action-card" @click="gatherFood">
            <div class="action-icon">🍓</div>
            <div class="action-title">采集食物</div>
            <div class="action-desc">在岛上寻找可食用的果实和动物</div>
            <div class="action-time">耗时: 30秒</div>
            <div class="action-bonus" v-if="getActionBonus('food') > 0">
              地形加成: +{{ getActionBonus('food') }}%
            </div>
          </div>
          
          <div class="action-card" @click="collectWater">
            <div class="action-icon">💧</div>
            <div class="action-title">收集淡水</div>
            <div class="action-desc">收集雨水或净化海水</div>
            <div class="action-time">耗时: 1分钟</div>
            <div class="action-bonus" v-if="getActionBonus('water') > 0">
              地形加成: +{{ getActionBonus('water') }}%
            </div>
          </div>
          
          <div class="action-card" @click="chopWood">
            <div class="action-icon">🪓</div>
            <div class="action-title">砍伐木材</div>
            <div class="action-desc">砍伐树木获取木材资源</div>
            <div class="action-time">耗时: 2分钟</div>
            <div class="action-bonus" v-if="getActionBonus('wood') > 0">
              地形加成: +{{ getActionBonus('wood') }}%
            </div>
          </div>
          
          <div class="action-card" @click="mineStone">
            <div class="action-icon">⛏️</div>
            <div class="action-title">挖掘石头</div>
            <div class="action-desc">在岛上挖掘石头资源</div>
            <div class="action-time">耗时: 3分钟</div>
            <div class="action-bonus" v-if="getActionBonus('stone') > 0">
              地形加成: +{{ getActionBonus('stone') }}%
            </div>
          </div>
          
          <div class="action-card" @click="buildShelter">
            <div class="action-icon">🏠</div>
            <div class="action-title">建造庇护所</div>
            <div class="action-desc">建造一个安全的住所</div>
            <div class="action-cost">需要: 50木材, 30石头</div>
          </div>
          
          <div class="action-card" @click="craftTools">
            <div class="action-icon">🔨</div>
            <div class="action-title">制作工具</div>
            <div class="action-desc">制作更高效的生存工具</div>
            <div class="action-cost">需要: 20木材, 10石头</div>
          </div>
        </div>
      </div>
      
      <div class="map-panel">
        <h3>🗺️ 海岛地图 <span class="explore-progress">(已探索 {{ exploredCount }}/{{ mapGrid.length }})</span></h3>
        <div class="map-container">
          <div class="map-grid">
            <div v-for="(cell, index) in mapGrid" :key="index" 
                 :class="getCellClasses(cell, index)"
                 @click="onCellClick(index)">
              <div class="cell-inner">
                <span class="cell-icon">{{ getCellDisplay(cell) }}</span>
                <div class="cell-overlay" v-if="!cell.explored">
                  <span class="fog-text">迷雾</span>
                </div>
              </div>
            </div>
          </div>
          <div class="map-legend">
            <div class="legend-item">
              <span class="legend-icon">🌳</span>
              <span>森林</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🏔️</span>
              <span>山地</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🌊</span>
              <span>海洋</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🏠</span>
              <span>营地</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🌴</span>
              <span>海滩</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon fog-legend">❓</span>
              <span>未探索</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="message-log">
      <h3>📜 生存日志</h3>
      <div class="log-list">
        <div v-for="(msg, index) in messageLog" :key="index" 
             class="log-item"
             :class="'log-' + msg.type">
          <span class="log-time">{{ msg.time }}</span>
          <span class="log-tag" v-if="msg.tag">[{{ msg.tag }}]</span>
          <span class="log-content">{{ msg.content }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';

const TERRAIN_CONFIG = {
  forest: {
    name: '森林',
    icon: '🌳',
    description: '茂密的丛林，栖息着各种野生动物和野果',
    riskLevel: 2,
    primaryResources: ['食物', '木材'],
    bonus: { food: 20, wood: 25 },
    bonusDescription: '食物+20%, 木材+25%'
  },
  mountain: {
    name: '山地',
    icon: '🏔️',
    description: '崎岖的山地，蕴藏着丰富的矿石资源',
    riskLevel: 3,
    primaryResources: ['石头', '木材'],
    bonus: { stone: 35, wood: 10 },
    bonusDescription: '石头+35%, 木材+10%'
  },
  ocean: {
    name: '海洋',
    icon: '🌊',
    description: '广阔的海洋，可以捕鱼和收集海水',
    riskLevel: 2,
    primaryResources: ['食物', '淡水'],
    bonus: { food: 15, water: 30 },
    bonusDescription: '食物+15%, 淡水+30%'
  },
  beach: {
    name: '海滩',
    icon: '🌴',
    description: '美丽的海滩，可以找到漂流物和贝类',
    riskLevel: 1,
    primaryResources: ['食物', '淡水'],
    bonus: { food: 10, water: 15 },
    bonusDescription: '食物+10%, 淡水+15%'
  },
  camp: {
    name: '营地',
    icon: '🏠',
    description: '你的安全营地，可以在这里休息和整理物资',
    riskLevel: 0,
    primaryResources: ['无'],
    bonus: {},
    bonusDescription: '无采集加成'
  }
};

const RESOURCE_NAMES = {
  food: '食物',
  water: '淡水',
  wood: '木材',
  stone: '石头'
};

const resources = ref({
  food: 100,
  water: 100,
  wood: 100,
  stone: 100
});

const selectedCell = ref(4);

const messageLog = ref([
  { time: '00:00', content: '你来到了一个荒岛，开始你的生存之旅吧！', type: 'system', tag: '系统' }
]);

const generateMapCell = (type, explored = false) => {
  const config = TERRAIN_CONFIG[type];
  return {
    type,
    icon: config.icon,
    name: config.name,
    description: config.description,
    riskLevel: config.riskLevel,
    primaryResources: config.primaryResources,
    bonus: config.bonus,
    bonusDescription: config.bonusDescription,
    explored,
    exploredAt: explored ? new Date() : null
  };
};

const mapGrid = ref([
  generateMapCell('forest', true),
  generateMapCell('beach', true),
  generateMapCell('mountain', false),
  generateMapCell('ocean', false),
  generateMapCell('camp', true),
  generateMapCell('forest', false),
  generateMapCell('ocean', false),
  generateMapCell('mountain', false),
  generateMapCell('forest', false)
]);

const exploredCount = computed(() => {
  return mapGrid.value.filter(c => c.explored).length;
});

const totalBonus = computed(() => {
  let sum = 0;
  mapGrid.value.forEach(cell => {
    if (cell.explored && cell.bonus) {
      Object.values(cell.bonus).forEach(v => sum += v);
    }
  });
  return sum;
});

const addMessage = (content, type = 'normal', tag = null) => {
  const now = new Date();
  const time = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
  messageLog.value.push({ time, content, type, tag });
  if (messageLog.value.length > 30) {
    messageLog.value.shift();
  }
};

const getCellDisplay = (cell) => {
  if (!cell.explored) return '❓';
  return cell.icon;
};

const getCellDisplayName = (cell) => {
  if (!cell.explored) return '未知区域';
  return cell.name;
};

const getCellClasses = (cell, index) => {
  const classes = ['map-cell'];
  if (cell.explored) {
    classes.push('explored');
    classes.push('type-' + cell.type);
  } else {
    classes.push('unexplored');
    classes.push('fog');
  }
  if (selectedCell.value === index) {
    classes.push('selected');
  }
  return classes.join(' ');
};

const getRiskLabel = (level) => {
  const labels = ['安全', '低风险', '中风险', '高风险', '极危险'];
  return labels[level] || '未知';
};

const getBonusDescription = (cell) => {
  return cell.bonusDescription || '无';
};

const getActionBonus = (resourceType) => {
  let bonus = 0;
  mapGrid.value.forEach(cell => {
    if (cell.explored && cell.bonus && cell.bonus[resourceType]) {
      bonus += cell.bonus[resourceType];
    }
  });
  return bonus;
};

const onCellClick = (index) => {
  selectedCell.value = index;
  const cell = mapGrid.value[index];
  if (!cell.explored) {
    exploreCell(index);
  }
};

const calculateGainWithBonus = (baseGain) => {
  const finalGain = {};
  for (const [resource, amount] of Object.entries(baseGain)) {
    const bonus = getActionBonus(resource);
    const bonusAmount = Math.floor(amount * (bonus / 100));
    finalGain[resource] = amount + bonusAmount;
  }
  return finalGain;
};

const performAction = (name, cost, gain, time) => {
  for (const [resource, amount] of Object.entries(cost)) {
    if (resources.value[resource] < amount) {
      ElMessage.error(`资源不足，无法${name}`);
      return false;
    }
  }
  
  for (const [resource, amount] of Object.entries(cost)) {
    resources.value[resource] -= amount;
  }
  
  addMessage(`开始${name}...`, 'action', '行动');
  
  setTimeout(() => {
    const finalGain = calculateGainWithBonus(gain);
    const gainDetails = [];
    
    for (const [resource, amount] of Object.entries(finalGain)) {
      resources.value[resource] += amount;
      const baseAmount = gain[resource] || 0;
      const bonusAmount = amount - baseAmount;
      if (bonusAmount > 0) {
        gainDetails.push(`${amount}${RESOURCE_NAMES[resource]}(含地形加成+${bonusAmount})`);
      } else {
        gainDetails.push(`${amount}${RESOURCE_NAMES[resource]}`);
      }
    }
    
    addMessage(`${name}完成！获得了${gainDetails.join('、')}`, 'success', '收获');
    ElMessage.success(`${name}完成！`);
  }, time);
  
  return true;
};

const gatherFood = () => {
  performAction('采集食物', {}, { food: 20 }, 30000);
};

const collectWater = () => {
  performAction('收集淡水', {}, { water: 30 }, 60000);
};

const chopWood = () => {
  performAction('砍伐木材', {}, { wood: 15 }, 120000);
};

const mineStone = () => {
  performAction('挖掘石头', {}, { stone: 10 }, 180000);
};

const buildShelter = () => {
  if (performAction('建造庇护所', { wood: 50, stone: 30 }, {}, 300000)) {
    addMessage('庇护所建造完成！你现在有了一个安全的住所。', 'success', '建造');
  }
};

const craftTools = () => {
  if (performAction('制作工具', { wood: 20, stone: 10 }, {}, 120000)) {
    addMessage('工具制作完成！你的工作效率提高了。', 'success', '制作');
  }
};

const exploreCell = (index) => {
  const cell = mapGrid.value[index];
  if (cell.explored) {
    return;
  }
  
  ElMessageBox.confirm(
    `前方是未知的迷雾区域，确定要冒险探索吗？\n可能会遇到危险，也可能发现丰富资源！`,
    '探索迷雾区域',
    {
      confirmButtonText: '开始探索',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    addMessage(`踏入迷雾区域，开始探索未知地带...`, 'explore', '探索');
    
    setTimeout(() => {
      cell.explored = true;
      cell.exploredAt = new Date();
      
      const config = TERRAIN_CONFIG[cell.type];
      addMessage(
        `迷雾散去！发现了【${config.name}】${config.icon} - ${config.description}`,
        'discover',
        '发现'
      );
      addMessage(
        `区域信息：风险等级【${getRiskLabel(config.riskLevel)}】，主要资源【${config.primaryResources.join('、')}】`,
        'info',
        '情报'
      );
      addMessage(
        `解锁采集加成：${config.bonusDescription}`,
        'bonus',
        '加成'
      );
      ElMessage.success(`发现了新区域：${config.name}！`);
      
      const random = Math.random();
      if (random < 0.25) {
        const foodGain = Math.floor(Math.random() * 25) + 15;
        resources.value.food += foodGain;
        addMessage(`探索途中发现了野果丛！获得${foodGain}食物`, 'reward', '奖励');
        ElMessage.success(`探索奖励：获得${foodGain}食物！`);
      } else if (random < 0.5) {
        const woodGain = Math.floor(Math.random() * 20) + 10;
        resources.value.wood += woodGain;
        addMessage(`发现了倒下的古树！获得${woodGain}木材`, 'reward', '奖励');
        ElMessage.success(`探索奖励：获得${woodGain}木材！`);
      } else if (random < 0.7) {
        const stoneGain = Math.floor(Math.random() * 15) + 8;
        resources.value.stone += stoneGain;
        addMessage(`发现了裸露的矿脉！获得${stoneGain}石头`, 'reward', '奖励');
        ElMessage.success(`探索奖励：获得${stoneGain}石头！`);
      } else if (random < 0.85) {
        const waterGain = Math.floor(Math.random() * 20) + 10;
        resources.value.water += waterGain;
        addMessage(`发现了清澈的水源！获得${waterGain}淡水`, 'reward', '奖励');
        ElMessage.success(`探索奖励：获得${waterGain}淡水！`);
      } else {
        const foodLoss = Math.floor(Math.random() * 10) + 5;
        const waterLoss = Math.floor(Math.random() * 10) + 5;
        resources.value.food = Math.max(0, resources.value.food - foodLoss);
        resources.value.water = Math.max(0, resources.value.water - waterLoss);
        addMessage(
          `探索时遭遇${config.riskLevel >= 3 ? '凶猛野兽袭击' : '意外事故'}！损失了${foodLoss}食物和${waterLoss}水`,
          'danger',
          '危险'
        );
        ElMessage.warning(`遭遇危险！损失了${foodLoss}食物和${waterLoss}水`);
      }
    }, 5000);
  }).catch(() => {
    addMessage('决定暂不冒险，等待时机成熟再探索这片迷雾区域。', 'normal', '探索');
  });
};

onMounted(() => {
  addMessage('欢迎来到海岛生存游戏！探索迷雾区域解锁更多资源和加成。', 'system', '系统');
  setInterval(() => {
    resources.value.food -= 5;
    resources.value.water -= 5;
    
    if (resources.value.food <= 0 || resources.value.water <= 0) {
      ElMessageBox.alert(
        '你的食物或水耗尽了，游戏结束！',
        '游戏结束',
        {
          confirmButtonText: '重新开始',
          type: 'error'
        }
      ).then(() => {
        resources.value.food = 100;
        resources.value.water = 100;
        resources.value.wood = 100;
        resources.value.stone = 100;
        mapGrid.value = [
          generateMapCell('forest', true),
          generateMapCell('beach', true),
          generateMapCell('mountain', false),
          generateMapCell('ocean', false),
          generateMapCell('camp', true),
          generateMapCell('forest', false),
          generateMapCell('ocean', false),
          generateMapCell('mountain', false),
          generateMapCell('forest', false)
        ];
        messageLog.value = [
          { time: '00:00', content: '你来到了一个荒岛，开始你的生存之旅吧！', type: 'system', tag: '系统' }
        ];
        selectedCell.value = 4;
        addMessage('重新开始游戏！', 'system', '系统');
      });
    }
  }, 60000);
});
</script>

<style scoped>
.island-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.island-header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
}

.island-header h1 {
  font-size: 48px;
  margin: 0 0 10px 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.island-header p {
  font-size: 18px;
  margin: 0;
  opacity: 0.9;
}

.island-main {
  max-width: 1200px;
  margin: 0 auto;
}

.stats-panel {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 15px;
  margin-bottom: 25px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 15px;
  display: flex;
  align-items: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-icon {
  font-size: 40px;
  margin-right: 15px;
}

.stat-content {
  flex: 1;
}

.stat-number {
  font-size: 26px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 13px;
  color: #666;
}

.area-info-panel {
  background: white;
  border-radius: 12px;
  padding: 25px;
  margin-bottom: 25px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.area-info-panel h3 {
  margin: 0 0 15px 0;
  font-size: 20px;
  color: #333;
}

.area-info-header {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.area-info-icon {
  font-size: 42px;
}

.area-info-name {
  font-size: 22px;
  font-weight: bold;
  color: #333;
  flex: 1;
}

.area-info-status {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 500;
}

.area-info-status.explored {
  background: #e8f5e9;
  color: #4caf50;
}

.area-info-status.unexplored {
  background: #fff3e0;
  color: #ff9800;
}

.area-info-details {
  padding: 10px 0;
}

.info-row {
  display: flex;
  margin-bottom: 10px;
  font-size: 14px;
  line-height: 1.6;
}

.info-label {
  font-weight: 600;
  color: #555;
  min-width: 80px;
}

.info-value {
  color: #333;
  flex: 1;
}

.risk-0 {
  color: #4caf50;
  font-weight: 600;
}

.risk-1 {
  color: #8bc34a;
  font-weight: 600;
}

.risk-2 {
  color: #ff9800;
  font-weight: 600;
}

.risk-3 {
  color: #f44336;
  font-weight: 600;
}

.risk-4 {
  color: #9c27b0;
  font-weight: 600;
}

.bonus-positive {
  color: #4caf50;
  font-weight: 600;
}

.area-fog {
  text-align: center;
  padding: 20px;
  background: linear-gradient(135deg, #e0e0e0 0%, #bdbdbd 100%);
  border-radius: 8px;
}

.fog-warning {
  color: #555;
}

.fog-icon {
  font-size: 60px;
  display: block;
  margin-bottom: 10px;
  animation: fogPulse 2s ease-in-out infinite;
}

@keyframes fogPulse {
  0%, 100% {
    opacity: 0.6;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.1);
  }
}

.fog-warning p {
  margin: 8px 0;
  font-size: 14px;
}

.fog-hint {
  color: #757575;
  font-size: 12px !important;
  margin-top: 15px !important;
}

.actions-panel {
  background: white;
  border-radius: 12px;
  padding: 25px;
  margin-bottom: 25px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.actions-panel h3 {
  margin: 0 0 20px 0;
  font-size: 22px;
  color: #333;
}

.action-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 15px;
}

.action-card {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.action-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.12);
  border-color: #667eea;
}

.action-icon {
  font-size: 42px;
  margin-bottom: 10px;
}

.action-title {
  font-size: 17px;
  font-weight: bold;
  color: #333;
  margin-bottom: 6px;
}

.action-desc {
  font-size: 13px;
  color: #666;
  margin-bottom: 6px;
}

.action-time,
.action-cost {
  font-size: 12px;
  color: #999;
}

.action-bonus {
  margin-top: 8px;
  font-size: 12px;
  color: #4caf50;
  font-weight: 600;
  background: #e8f5e9;
  padding: 4px 8px;
  border-radius: 4px;
  display: inline-block;
}

.map-panel {
  background: white;
  border-radius: 12px;
  padding: 25px;
  margin-bottom: 25px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.map-panel h3 {
  margin: 0 0 20px 0;
  font-size: 22px;
  color: #333;
  display: flex;
  align-items: center;
  gap: 10px;
}

.explore-progress {
  font-size: 14px;
  color: #999;
  font-weight: normal;
}

.map-container {
  text-align: center;
}

.map-grid {
  display: grid;
  grid-template-columns: repeat(3, 110px);
  gap: 12px;
  justify-content: center;
  margin-bottom: 25px;
}

.map-cell {
  width: 110px;
  height: 110px;
  background: #f0f0f0;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 42px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 3px solid #ddd;
  position: relative;
  overflow: hidden;
}

.map-cell:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.map-cell.selected {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.3);
}

.cell-inner {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cell-icon {
  z-index: 1;
}

.map-cell.fog {
  background: linear-gradient(135deg, #9e9e9e 0%, #757575 50%, #616161 100%);
  border-color: #424242;
}

.map-cell.fog .cell-icon {
  filter: blur(1px);
  opacity: 0.8;
}

.cell-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(33, 33, 33, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(2px);
  z-index: 2;
}

.fog-text {
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  font-weight: 600;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  letter-spacing: 2px;
}

.map-cell.fog:hover {
  animation: fogGlow 1.5s ease-in-out infinite;
}

@keyframes fogGlow {
  0%, 100% {
    box-shadow: 0 0 8px rgba(158, 158, 158, 0.5);
  }
  50% {
    box-shadow: 0 0 16px rgba(158, 158, 158, 0.8);
  }
}

.map-cell.explored {
  background: #e8f4fa;
  border-color: #409eff;
}

.map-cell.explored.type-forest {
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  border-color: #66bb6a;
}

.map-cell.explored.type-mountain {
  background: linear-gradient(135deg, #eceff1 0%, #cfd8dc 100%);
  border-color: #78909c;
}

.map-cell.explored.type-ocean {
  background: linear-gradient(135deg, #e1f5fe 0%, #b3e5fc 100%);
  border-color: #29b6f6;
}

.map-cell.explored.type-beach {
  background: linear-gradient(135deg, #fff8e1 0%, #ffecb3 100%);
  border-color: #ffb74d;
}

.map-cell.explored.type-camp {
  background: linear-gradient(135deg, #fce4ec 0%, #f8bbd9 100%);
  border-color: #ec407a;
}

.map-legend {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: #555;
}

.legend-icon {
  font-size: 22px;
}

.legend-icon.fog-legend {
  opacity: 0.7;
}

.message-log {
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.message-log h3 {
  margin: 0 0 20px 0;
  font-size: 22px;
  color: #333;
}

.log-list {
  max-height: 320px;
  overflow-y: auto;
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 8px;
  background: #fafafa;
}

.log-item {
  display: flex;
  margin-bottom: 6px;
  padding: 10px 12px;
  background: white;
  border-radius: 6px;
  border-left: 3px solid #e0e0e0;
  align-items: flex-start;
  font-size: 13px;
  line-height: 1.5;
}

.log-item.log-system {
  border-left-color: #9c27b0;
  background: #f3e5f5;
}

.log-item.log-action {
  border-left-color: #2196f3;
  background: #e3f2fd;
}

.log-item.log-success {
  border-left-color: #4caf50;
  background: #e8f5e9;
}

.log-item.log-explore {
  border-left-color: #ff9800;
  background: #fff3e0;
}

.log-item.log-discover {
  border-left-color: #00bcd4;
  background: #e0f7fa;
}

.log-item.log-info {
  border-left-color: #607d8b;
  background: #eceff1;
}

.log-item.log-bonus {
  border-left-color: #e91e63;
  background: #fce4ec;
}

.log-item.log-reward {
  border-left-color: #8bc34a;
  background: #f1f8e9;
}

.log-item.log-danger {
  border-left-color: #f44336;
  background: #ffebee;
}

.log-time {
  font-weight: bold;
  color: #666;
  margin-right: 10px;
  min-width: 55px;
  font-size: 12px;
}

.log-tag {
  color: #888;
  margin-right: 10px;
  font-weight: 600;
  font-size: 12px;
  white-space: nowrap;
}

.log-content {
  flex: 1;
  color: #444;
}

@media (max-width: 768px) {
  .island-header h1 {
    font-size: 32px;
  }
  
  .stats-panel {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .action-grid {
    grid-template-columns: 1fr;
  }

  .map-grid {
    grid-template-columns: repeat(3, 80px);
    gap: 8px;
  }

  .map-cell {
    width: 80px;
    height: 80px;
    font-size: 32px;
  }

  .fog-text {
    font-size: 11px;
  }

  .stat-number {
    font-size: 20px;
  }

  .stat-icon {
    font-size: 32px;
    margin-right: 10px;
  }
}
</style>