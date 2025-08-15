<script setup>
// Day 3 练习 - 互动游戏集合
import { ref, reactive, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'

// ==================== 游戏管理 ====================

// 当前游戏
const currentGame = ref('menu')

// 游戏列表
const games = [
  {
    id: 'memory',
    name: '记忆翻牌',
    icon: '🧠',
    description: '翻开相同的卡片，训练记忆力',
    difficulty: '中等',
    category: '记忆'
  },
  {
    id: 'guess',
    name: '猜数字',
    icon: '🎯',
    description: '在指定范围内猜出随机数',
    difficulty: '简单',
    category: '逻辑'
  },
  {
    id: 'reaction',
    name: '反应测试',
    icon: '⚡',
    description: '测试你的反应速度',
    difficulty: '简单',
    category: '反应'
  },
  {
    id: 'typing',
    name: '打字游戏',
    icon: '⌨️',
    description: '快速准确地输入文字',
    difficulty: '中等',
    category: '技能'
  },
  {
    id: 'snake',
    name: '贪吃蛇',
    icon: '🐍',
    description: '经典贪吃蛇游戏',
    difficulty: '困难',
    category: '动作'
  },
  {
    id: 'rps',
    name: '石头剪刀布',
    icon: '✂️',
    description: '与电脑对战石头剪刀布',
    difficulty: '简单',
    category: '策略'
  },
  {
    id: 'quiz',
    name: '知识问答',
    icon: '🧭',
    description: '回答各种知识问题',
    difficulty: '中等',
    category: '知识'
  },
  {
    id: 'color',
    name: '颜色挑战',
    icon: '🎨',
    description: '识别颜色名称与颜色',
    difficulty: '中等',
    category: '视觉'
  }
]

// 用户信息
const player = reactive({
  name: '玩家',
  totalScore: 0,
  gamesPlayed: 0,
  achievements: [],
  stats: {}
})

// ==================== 记忆翻牌游戏 ====================

const memoryGame = reactive({
  cards: [],
  flippedCards: [],
  matchedPairs: 0,
  moves: 0,
  timer: 0,
  isPlaying: false,
  difficulty: 'easy', // easy: 4x3, medium: 4x4, hard: 6x4
  gameOver: false,
  score: 0
})

const memoryGameConfig = {
  easy: { rows: 3, cols: 4, pairs: 6, timeBonus: 300 },
  medium: { rows: 4, cols: 4, pairs: 8, timeBonus: 600 },
  hard: { rows: 4, cols: 6, pairs: 12, timeBonus: 900 }
}

let memoryTimer = null

// 初始化记忆游戏
function initMemoryGame() {
  const config = memoryGameConfig[memoryGame.difficulty]
  const symbols = ['🐶', '🐱', '🐭', '🐹', '🐰', '🦊', '🐻', '🐼', '🐨', '🐯', '🦁', '🐮', '🐷', '🐸', '🐵', '🐔', '🐧', '🐦', '🐤', '🦄']
  
  // 创建卡片对
  const cardSymbols = symbols.slice(0, config.pairs)
  const allCards = [...cardSymbols, ...cardSymbols]
  
  // 打乱卡片
  for (let i = allCards.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [allCards[i], allCards[j]] = [allCards[j], allCards[i]]
  }
  
  // 创建卡片对象
  memoryGame.cards = allCards.map((symbol, index) => ({
    id: index,
    symbol,
    flipped: false,
    matched: false
  }))
  
  memoryGame.flippedCards = []
  memoryGame.matchedPairs = 0
  memoryGame.moves = 0
  memoryGame.timer = 0
  memoryGame.gameOver = false
  memoryGame.score = 0
}

function startMemoryGame() {
  initMemoryGame()
  memoryGame.isPlaying = true
  
  memoryTimer = setInterval(() => {
    memoryGame.timer++
  }, 1000)
}

function flipCard(card) {
  if (!memoryGame.isPlaying || card.flipped || card.matched || memoryGame.flippedCards.length >= 2) {
    return
  }
  
  card.flipped = true
  memoryGame.flippedCards.push(card)
  
  if (memoryGame.flippedCards.length === 2) {
    memoryGame.moves++
    checkMemoryMatch()
  }
}

function checkMemoryMatch() {
  const [card1, card2] = memoryGame.flippedCards
  
  setTimeout(() => {
    if (card1.symbol === card2.symbol) {
      // 匹配成功
      card1.matched = true
      card2.matched = true
      memoryGame.matchedPairs++
      memoryGame.score += 100
      
      // 检查游戏是否结束
      const config = memoryGameConfig[memoryGame.difficulty]
      if (memoryGame.matchedPairs === config.pairs) {
        endMemoryGame()
      }
    } else {
      // 匹配失败
      card1.flipped = false
      card2.flipped = false
      memoryGame.score = Math.max(0, memoryGame.score - 10)
    }
    
    memoryGame.flippedCards = []
  }, 1000)
}

function endMemoryGame() {
  memoryGame.isPlaying = false
  memoryGame.gameOver = true
  
  if (memoryTimer) {
    clearInterval(memoryTimer)
    memoryTimer = null
  }
  
  // 计算最终分数
  const config = memoryGameConfig[memoryGame.difficulty]
  const timeBonus = Math.max(0, config.timeBonus - memoryGame.timer)
  const moveBonus = Math.max(0, (config.pairs * 2 - memoryGame.moves) * 10)
  memoryGame.score += timeBonus + moveBonus
  
  updatePlayerStats('memory', memoryGame.score)
}

// ==================== 猜数字游戏 ====================

const guessGame = reactive({
  targetNumber: 0,
  userGuess: '',
  attempts: 0,
  maxAttempts: 7,
  range: { min: 1, max: 100 },
  hints: [],
  isPlaying: false,
  gameOver: false,
  won: false,
  score: 0,
  difficulty: 'medium' // easy: 1-50, medium: 1-100, hard: 1-500
})

const guessGameConfig = {
  easy: { min: 1, max: 50, maxAttempts: 6, baseScore: 100 },
  medium: { min: 1, max: 100, maxAttempts: 7, baseScore: 200 },
  hard: { min: 1, max: 500, maxAttempts: 9, baseScore: 500 }
}

function startGuessGame() {
  const config = guessGameConfig[guessGame.difficulty]
  guessGame.targetNumber = Math.floor(Math.random() * (config.max - config.min + 1)) + config.min
  guessGame.range = { min: config.min, max: config.max }
  guessGame.maxAttempts = config.maxAttempts
  guessGame.userGuess = ''
  guessGame.attempts = 0
  guessGame.hints = []
  guessGame.isPlaying = true
  guessGame.gameOver = false
  guessGame.won = false
  guessGame.score = 0
}

function makeGuess() {
  const guess = parseInt(guessGame.userGuess)
  
  if (isNaN(guess) || guess < guessGame.range.min || guess > guessGame.range.max) {
    alert(`请输入 ${guessGame.range.min} 到 ${guessGame.range.max} 之间的数字`)
    return
  }
  
  guessGame.attempts++
  
  if (guess === guessGame.targetNumber) {
    guessGame.won = true
    guessGame.gameOver = true
    guessGame.isPlaying = false
    
    const config = guessGameConfig[guessGame.difficulty]
    guessGame.score = config.baseScore - (guessGame.attempts - 1) * 20
    guessGame.score = Math.max(guessGame.score, 20)
    
    guessGame.hints.push(`🎉 恭喜！你猜中了！答案就是 ${guessGame.targetNumber}`)
    updatePlayerStats('guess', guessGame.score)
  } else if (guessGame.attempts >= guessGame.maxAttempts) {
    guessGame.gameOver = true
    guessGame.isPlaying = false
    guessGame.hints.push(`😔 游戏结束！答案是 ${guessGame.targetNumber}`)
    updatePlayerStats('guess', 0)
  } else {
    const hint = guess > guessGame.targetNumber ? '太大了' : '太小了'
    const remaining = guessGame.maxAttempts - guessGame.attempts
    guessGame.hints.push(`第${guessGame.attempts}次：${guess} - ${hint}，还有${remaining}次机会`)
  }
  
  guessGame.userGuess = ''
}

// ==================== 反应测试游戏 ====================

const reactionGame = reactive({
  phase: 'waiting', // waiting, ready, go, result
  startTime: 0,
  reactionTime: 0,
  bestTime: localStorage.getItem('bestReactionTime') || null,
  attempts: 0,
  averageTime: 0,
  times: [],
  score: 0,
  message: '点击开始按钮准备测试'
})

let reactionTimeout = null

function startReactionTest() {
  reactionGame.phase = 'ready'
  reactionGame.message = '等待绿色信号...'
  
  // 随机延迟 2-5 秒
  const delay = Math.random() * 3000 + 2000
  
  reactionTimeout = setTimeout(() => {
    if (reactionGame.phase === 'ready') {
      reactionGame.phase = 'go'
      reactionGame.message = '现在点击！'
      reactionGame.startTime = Date.now()
    }
  }, delay)
}

function handleReactionClick() {
  if (reactionGame.phase === 'ready') {
    // 太早点击了
    reactionGame.phase = 'waiting'
    reactionGame.message = '太早了！等待绿色信号再点击'
    clearTimeout(reactionTimeout)
  } else if (reactionGame.phase === 'go') {
    // 正确的反应
    const endTime = Date.now()
    reactionGame.reactionTime = endTime - reactionGame.startTime
    reactionGame.phase = 'result'
    
    reactionGame.attempts++
    reactionGame.times.push(reactionGame.reactionTime)
    
    // 计算平均时间
    reactionGame.averageTime = Math.round(
      reactionGame.times.reduce((sum, time) => sum + time, 0) / reactionGame.times.length
    )
    
    // 更新最佳时间
    if (!reactionGame.bestTime || reactionGame.reactionTime < reactionGame.bestTime) {
      reactionGame.bestTime = reactionGame.reactionTime
      localStorage.setItem('bestReactionTime', reactionGame.bestTime)
    }
    
    // 计算分数
    let score = 0
    if (reactionGame.reactionTime < 200) score = 100
    else if (reactionGame.reactionTime < 300) score = 80
    else if (reactionGame.reactionTime < 400) score = 60
    else if (reactionGame.reactionTime < 500) score = 40
    else score = 20
    
    reactionGame.score = score
    reactionGame.message = getReactionMessage(reactionGame.reactionTime)
    
    updatePlayerStats('reaction', score)
  }
}

function getReactionMessage(time) {
  if (time < 200) return '🚀 闪电般的反应！'
  if (time < 300) return '⚡ 反应很快！'
  if (time < 400) return '👍 反应不错！'
  if (time < 500) return '😐 还需要练习'
  return '🐌 反应有点慢哦'
}

function resetReactionGame() {
  reactionGame.phase = 'waiting'
  reactionGame.message = '点击开始按钮准备测试'
  clearTimeout(reactionTimeout)
}

// ==================== 打字游戏 ====================

const typingGame = reactive({
  texts: [
    'The quick brown fox jumps over the lazy dog',
    'Pack my box with five dozen liquor jugs',
    'How vexingly quick daft zebras jump',
    'Vue.js is a progressive JavaScript framework',
    'TypeScript adds static typing to JavaScript',
    'React is a library for building user interfaces',
    'Node.js is a JavaScript runtime built on Chrome V8',
    'HTML, CSS, and JavaScript are the building blocks of web'
  ],
  currentText: '',
  userInput: '',
  startTime: 0,
  endTime: 0,
  isPlaying: false,
  gameOver: false,
  wpm: 0,
  accuracy: 0,
  score: 0,
  mistakes: 0,
  currentIndex: 0
})

function startTypingGame() {
  const randomIndex = Math.floor(Math.random() * typingGame.texts.length)
  typingGame.currentText = typingGame.texts[randomIndex]
  typingGame.userInput = ''
  typingGame.startTime = Date.now()
  typingGame.endTime = 0
  typingGame.isPlaying = true
  typingGame.gameOver = false
  typingGame.wpm = 0
  typingGame.accuracy = 0
  typingGame.score = 0
  typingGame.mistakes = 0
  typingGame.currentIndex = 0
}

function handleTypingInput() {
  if (!typingGame.isPlaying) return
  
  const input = typingGame.userInput
  const target = typingGame.currentText
  
  typingGame.currentIndex = input.length
  
  // 计算错误数
  typingGame.mistakes = 0
  for (let i = 0; i < input.length && i < target.length; i++) {
    if (input[i] !== target[i]) {
      typingGame.mistakes++
    }
  }
  
  // 检查是否完成
  if (input === target) {
    endTypingGame()
  }
  
  // 实时计算WPM和准确率
  if (input.length > 0) {
    const timeElapsed = (Date.now() - typingGame.startTime) / 60000 // 分钟
    const wordsTyped = input.split(' ').length
    typingGame.wpm = Math.round(wordsTyped / timeElapsed)
    typingGame.accuracy = Math.round(((input.length - typingGame.mistakes) / input.length) * 100)
  }
}

function endTypingGame() {
  typingGame.isPlaying = false
  typingGame.gameOver = true
  typingGame.endTime = Date.now()
  
  const timeElapsed = (typingGame.endTime - typingGame.startTime) / 60000
  const wordsCount = typingGame.currentText.split(' ').length
  typingGame.wpm = Math.round(wordsCount / timeElapsed)
  
  const accuracy = ((typingGame.currentText.length - typingGame.mistakes) / typingGame.currentText.length) * 100
  typingGame.accuracy = Math.round(accuracy)
  
  // 计算分数
  typingGame.score = Math.round(typingGame.wpm * (typingGame.accuracy / 100) * 10)
  
  updatePlayerStats('typing', typingGame.score)
}

// ==================== 贪吃蛇游戏 ====================

const snakeGame = reactive({
  board: { width: 20, height: 15 },
  snake: [{ x: 10, y: 7 }],
  food: { x: 5, y: 5 },
  direction: { x: 1, y: 0 },
  nextDirection: { x: 1, y: 0 },
  score: 0,
  isPlaying: false,
  gameOver: false,
  speed: 200
})

let snakeGameInterval = null

function startSnakeGame() {
  snakeGame.snake = [{ x: 10, y: 7 }]
  snakeGame.food = generateFood()
  snakeGame.direction = { x: 1, y: 0 }
  snakeGame.nextDirection = { x: 1, y: 0 }
  snakeGame.score = 0
  snakeGame.isPlaying = true
  snakeGame.gameOver = false
  snakeGame.speed = 200
  
  snakeGameInterval = setInterval(updateSnake, snakeGame.speed)
}

function generateFood() {
  let food
  do {
    food = {
      x: Math.floor(Math.random() * snakeGame.board.width),
      y: Math.floor(Math.random() * snakeGame.board.height)
    }
  } while (snakeGame.snake.some(segment => segment.x === food.x && segment.y === food.y))
  
  return food
}

function updateSnake() {
  if (!snakeGame.isPlaying) return
  
  snakeGame.direction = { ...snakeGame.nextDirection }
  
  const head = { ...snakeGame.snake[0] }
  head.x += snakeGame.direction.x
  head.y += snakeGame.direction.y
  
  // 检查碰撞
  if (head.x < 0 || head.x >= snakeGame.board.width || 
      head.y < 0 || head.y >= snakeGame.board.height ||
      snakeGame.snake.some(segment => segment.x === head.x && segment.y === head.y)) {
    endSnakeGame()
    return
  }
  
  snakeGame.snake.unshift(head)
  
  // 检查是否吃到食物
  if (head.x === snakeGame.food.x && head.y === snakeGame.food.y) {
    snakeGame.score += 10
    snakeGame.food = generateFood()
    
    // 增加速度
    if (snakeGame.score % 50 === 0 && snakeGame.speed > 100) {
      snakeGame.speed -= 20
      clearInterval(snakeGameInterval)
      snakeGameInterval = setInterval(updateSnake, snakeGame.speed)
    }
  } else {
    snakeGame.snake.pop()
  }
}

function changeSnakeDirection(newDirection) {
  if (!snakeGame.isPlaying) return
  
  // 防止反向移动
  if ((newDirection.x === -snakeGame.direction.x && newDirection.y === snakeGame.direction.y) ||
      (newDirection.y === -snakeGame.direction.y && newDirection.x === snakeGame.direction.x)) {
    return
  }
  
  snakeGame.nextDirection = newDirection
}

function endSnakeGame() {
  snakeGame.isPlaying = false
  snakeGame.gameOver = true
  
  if (snakeGameInterval) {
    clearInterval(snakeGameInterval)
    snakeGameInterval = null
  }
  
  updatePlayerStats('snake', snakeGame.score)
}

// ==================== 石头剪刀布游戏 ====================

const rpsGame = reactive({
  playerChoice: '',
  computerChoice: '',
  result: '',
  playerScore: 0,
  computerScore: 0,
  round: 0,
  maxRounds: 5,
  gameOver: false,
  choices: [
    { name: 'rock', emoji: '🪨', label: '石头' },
    { name: 'paper', emoji: '📄', label: '布' },
    { name: 'scissors', emoji: '✂️', label: '剪刀' }
  ],
  history: []
})

function playRPS(playerChoice) {
  if (rpsGame.gameOver) return
  
  const choices = ['rock', 'paper', 'scissors']
  const computerChoice = choices[Math.floor(Math.random() * 3)]
  
  rpsGame.playerChoice = playerChoice
  rpsGame.computerChoice = computerChoice
  rpsGame.round++
  
  let result = ''
  let winner = ''
  
  if (playerChoice === computerChoice) {
    result = '平局'
    winner = 'tie'
  } else if (
    (playerChoice === 'rock' && computerChoice === 'scissors') ||
    (playerChoice === 'paper' && computerChoice === 'rock') ||
    (playerChoice === 'scissors' && computerChoice === 'paper')
  ) {
    result = '你赢了'
    winner = 'player'
    rpsGame.playerScore++
  } else {
    result = '电脑赢了'
    winner = 'computer'
    rpsGame.computerScore++
  }
  
  rpsGame.result = result
  rpsGame.history.unshift({
    round: rpsGame.round,
    player: playerChoice,
    computer: computerChoice,
    result: winner
  })
  
  if (rpsGame.round >= rpsGame.maxRounds) {
    rpsGame.gameOver = true
    
    let finalScore = 0
    if (rpsGame.playerScore > rpsGame.computerScore) {
      finalScore = rpsGame.playerScore * 20
    } else if (rpsGame.playerScore === rpsGame.computerScore) {
      finalScore = rpsGame.playerScore * 10
    }
    
    updatePlayerStats('rps', finalScore)
  }
}

function resetRPSGame() {
  rpsGame.playerChoice = ''
  rpsGame.computerChoice = ''
  rpsGame.result = ''
  rpsGame.playerScore = 0
  rpsGame.computerScore = 0
  rpsGame.round = 0
  rpsGame.gameOver = false
  rpsGame.history = []
}

// ==================== 知识问答游戏 ====================

const quizGame = reactive({
  questions: [
    {
      question: 'Vue.js 是由哪位开发者创建的？',
      options: ['尤雨溪', '阮一峰', '廖雪峰', '玉伯'],
      correct: 0
    },
    {
      question: 'CSS 中哪个属性用于设置文本颜色？',
      options: ['background-color', 'color', 'text-color', 'font-color'],
      correct: 1
    },
    {
      question: 'JavaScript 中哪个方法用于将字符串转换为数字？',
      options: ['toString()', 'parseInt()', 'valueOf()', 'convert()'],
      correct: 1
    },
    {
      question: 'HTML 中哪个标签用于创建超链接？',
      options: ['<link>', '<href>', '<a>', '<url>'],
      correct: 2
    },
    {
      question: 'HTTP 状态码 404 表示什么？',
      options: ['服务器错误', '请求成功', '页面未找到', '权限不足'],
      correct: 2
    },
    {
      question: '在响应式设计中，px、em、rem 中哪个是相对单位？',
      options: ['px', 'em 和 rem', '只有 em', '只有 rem'],
      correct: 1
    },
    {
      question: 'Git 中哪个命令用于查看提交历史？',
      options: ['git status', 'git log', 'git show', 'git history'],
      correct: 1
    },
    {
      question: 'JSON 的全称是什么？',
      options: ['JavaScript Object Notation', 'Java Source Object Name', 'JavaScript Original Network', 'Java Standard Object Notation'],
      correct: 0
    }
  ],
  currentQuestion: 0,
  selectedAnswer: null,
  correctAnswers: 0,
  wrongAnswers: 0,
  isPlaying: false,
  gameOver: false,
  score: 0,
  timeLeft: 30,
  totalTime: 30
})

let quizTimer = null

function startQuizGame() {
  // 打乱问题顺序
  quizGame.questions = [...quizGame.questions].sort(() => Math.random() - 0.5)
  
  quizGame.currentQuestion = 0
  quizGame.selectedAnswer = null
  quizGame.correctAnswers = 0
  quizGame.wrongAnswers = 0
  quizGame.isPlaying = true
  quizGame.gameOver = false
  quizGame.score = 0
  quizGame.timeLeft = quizGame.totalTime
  
  startQuizTimer()
}

function startQuizTimer() {
  quizTimer = setInterval(() => {
    quizGame.timeLeft--
    
    if (quizGame.timeLeft <= 0) {
      // 时间到，自动选择错误答案
      answerQuestion(-1)
    }
  }, 1000)
}

function answerQuestion(answerIndex) {
  if (!quizGame.isPlaying) return
  
  clearInterval(quizTimer)
  
  const question = quizGame.questions[quizGame.currentQuestion]
  const isCorrect = answerIndex === question.correct
  
  quizGame.selectedAnswer = answerIndex
  
  if (isCorrect) {
    quizGame.correctAnswers++
    quizGame.score += Math.max(10, quizGame.timeLeft * 2) // 时间奖励
  } else {
    quizGame.wrongAnswers++
  }
  
  setTimeout(() => {
    quizGame.currentQuestion++
    
    if (quizGame.currentQuestion >= quizGame.questions.length) {
      endQuizGame()
    } else {
      quizGame.selectedAnswer = null
      quizGame.timeLeft = quizGame.totalTime
      startQuizTimer()
    }
  }, 1500)
}

function endQuizGame() {
  quizGame.isPlaying = false
  quizGame.gameOver = true
  
  if (quizTimer) {
    clearInterval(quizTimer)
    quizTimer = null
  }
  
  updatePlayerStats('quiz', quizGame.score)
}

// ==================== 颜色挑战游戏 ====================

const colorGame = reactive({
  colors: [
    { name: '红色', color: '#FF0000', chinese: '红色' },
    { name: '蓝色', color: '#0000FF', chinese: '蓝色' },
    { name: '绿色', color: '#00FF00', chinese: '绿色' },
    { name: '黄色', color: '#FFFF00', chinese: '黄色' },
    { name: '紫色', color: '#800080', chinese: '紫色' },
    { name: '橙色', color: '#FFA500', chinese: '橙色' },
    { name: '粉色', color: '#FFC0CB', chinese: '粉色' },
    { name: '青色', color: '#00FFFF', chinese: '青色' }
  ],
  currentWord: '',
  currentColor: '',
  correctAnswer: '',
  options: [],
  selectedAnswer: '',
  score: 0,
  round: 0,
  maxRounds: 10,
  isPlaying: false,
  gameOver: false,
  timeLeft: 10,
  streak: 0,
  maxStreak: 0
})

let colorTimer = null

function startColorGame() {
  colorGame.score = 0
  colorGame.round = 0
  colorGame.isPlaying = true
  colorGame.gameOver = false
  colorGame.streak = 0
  colorGame.maxStreak = 0
  
  nextColorRound()
}

function nextColorRound() {
  if (colorGame.round >= colorGame.maxRounds) {
    endColorGame()
    return
  }
  
  const wordColor = colorGame.colors[Math.floor(Math.random() * colorGame.colors.length)]
  const displayColor = colorGame.colors[Math.floor(Math.random() * colorGame.colors.length)]
  
  colorGame.currentWord = wordColor.chinese
  colorGame.currentColor = displayColor.color
  colorGame.correctAnswer = displayColor.chinese
  colorGame.selectedAnswer = ''
  colorGame.timeLeft = 10
  colorGame.round++
  
  // 生成选项（正确答案 + 3个错误答案）
  const wrongOptions = colorGame.colors
    .filter(c => c.chinese !== colorGame.correctAnswer)
    .sort(() => Math.random() - 0.5)
    .slice(0, 3)
    .map(c => c.chinese)
  
  colorGame.options = [colorGame.correctAnswer, ...wrongOptions]
    .sort(() => Math.random() - 0.5)
  
  startColorTimer()
}

function startColorTimer() {
  colorTimer = setInterval(() => {
    colorGame.timeLeft--
    
    if (colorGame.timeLeft <= 0) {
      // 时间到，自动选择错误答案
      answerColorQuestion('')
    }
  }, 1000)
}

function answerColorQuestion(answer) {
  if (!colorGame.isPlaying) return
  
  clearInterval(colorTimer)
  
  colorGame.selectedAnswer = answer
  const isCorrect = answer === colorGame.correctAnswer
  
  if (isCorrect) {
    colorGame.score += Math.max(10, colorGame.timeLeft * 5)
    colorGame.streak++
    colorGame.maxStreak = Math.max(colorGame.maxStreak, colorGame.streak)
  } else {
    colorGame.streak = 0
  }
  
  setTimeout(() => {
    nextColorRound()
  }, 1000)
}

function endColorGame() {
  colorGame.isPlaying = false
  colorGame.gameOver = true
  
  if (colorTimer) {
    clearInterval(colorTimer)
    colorTimer = null
  }
  
  // 连击奖励
  colorGame.score += colorGame.maxStreak * 50
  
  updatePlayerStats('color', colorGame.score)
}

// ==================== 通用功能 ====================

// 更新玩家统计
function updatePlayerStats(gameType, score) {
  player.totalScore += score
  player.gamesPlayed++
  
  if (!player.stats[gameType]) {
    player.stats[gameType] = {
      played: 0,
      bestScore: 0,
      totalScore: 0
    }
  }
  
  const gameStats = player.stats[gameType]
  gameStats.played++
  gameStats.totalScore += score
  gameStats.bestScore = Math.max(gameStats.bestScore, score)
  
  // 检查成就
  checkAchievements(gameType, score)
  
  // 保存到本地存储
  localStorage.setItem('gamePlayerStats', JSON.stringify(player))
}

// 检查成就
function checkAchievements(gameType, score) {
  const achievements = [
    { id: 'first_game', name: '初次体验', description: '完成第一个游戏', condition: () => player.gamesPlayed === 1 },
    { id: 'score_1000', name: '千分达成', description: '总分达到1000分', condition: () => player.totalScore >= 1000 },
    { id: 'game_master', name: '游戏大师', description: '完成所有类型的游戏', condition: () => Object.keys(player.stats).length >= 6 },
    { id: 'memory_expert', name: '记忆专家', description: '记忆游戏得分超过800', condition: () => player.stats.memory?.bestScore >= 800 },
    { id: 'quick_reaction', name: '反应神速', description: '反应时间少于200ms', condition: () => reactionGame.bestTime && reactionGame.bestTime < 200 },
    { id: 'typing_master', name: '打字高手', description: '打字速度超过60WPM', condition: () => player.stats.typing?.bestScore >= 600 },
    { id: 'snake_legend', name: '贪吃蛇传说', description: '贪吃蛇得分超过200', condition: () => player.stats.snake?.bestScore >= 200 }
  ]
  
  achievements.forEach(achievement => {
    if (!player.achievements.includes(achievement.id) && achievement.condition()) {
      player.achievements.push(achievement.id)
      showAchievement(achievement)
    }
  })
}

// 显示成就通知
function showAchievement(achievement) {
  // 简单的成就通知
  setTimeout(() => {
    alert(`🏆 获得成就：${achievement.name}\n${achievement.description}`)
  }, 500)
}

// 初始化玩家数据
function initPlayerData() {
  const savedStats = localStorage.getItem('gamePlayerStats')
  if (savedStats) {
    Object.assign(player, JSON.parse(savedStats))
  }
}

// 键盘事件处理
function handleKeyPress(event) {
  if (currentGame.value === 'snake' && snakeGame.isPlaying) {
    switch (event.key) {
      case 'ArrowUp':
      case 'w':
      case 'W':
        event.preventDefault()
        changeSnakeDirection({ x: 0, y: -1 })
        break
      case 'ArrowDown':
      case 's':
      case 'S':
        event.preventDefault()
        changeSnakeDirection({ x: 0, y: 1 })
        break
      case 'ArrowLeft':
      case 'a':
      case 'A':
        event.preventDefault()
        changeSnakeDirection({ x: -1, y: 0 })
        break
      case 'ArrowRight':
      case 'd':
      case 'D':
        event.preventDefault()
        changeSnakeDirection({ x: 1, y: 0 })
        break
    }
  }
  
  if (currentGame.value === 'reaction' && reactionGame.phase === 'go') {
    handleReactionClick()
  }
  
  if (currentGame.value === 'guess' && event.key === 'Enter') {
    makeGuess()
  }
}

// 生成页码
function getPageNumbers() {
  // 这里是一个简化版本，实际项目中可能需要更复杂的逻辑
  return [1, 2, 3, '...', 10]
}

// 处理过滤选项变化
function handleFilterOptionChange(columnKey, option, checked) {
  if (!tableConfig.filters.columns[columnKey]) {
    tableConfig.filters.columns[columnKey] = { value: [] }
  }
  
  if (!Array.isArray(tableConfig.filters.columns[columnKey].value)) {
    tableConfig.filters.columns[columnKey].value = []
  }
  
  const values = tableConfig.filters.columns[columnKey].value
  const index = values.indexOf(option)
  
  if (checked && index === -1) {
    values.push(option)
  } else if (!checked && index > -1) {
    values.splice(index, 1)
  }
}

// 计算属性：当前游戏信息
const currentGameInfo = computed(() => {
  return games.find(game => game.id === currentGame.value) || null
})

// 生命周期
onMounted(() => {
  initPlayerData()
  document.addEventListener('keydown', handleKeyPress)
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyPress)
  
  // 清理定时器
  if (memoryTimer) clearInterval(memoryTimer)
  if (snakeGameInterval) clearInterval(snakeGameInterval)
  if (reactionTimeout) clearTimeout(reactionTimeout)
  if (quizTimer) clearInterval(quizTimer)
  if (colorTimer) clearInterval(colorTimer)
})
</script>

<template>
  <div class="interactive-game">
    <h2>🎮 互动游戏集合</h2>
    
    <!-- 游戏菜单 -->
    <div v-if="currentGame === 'menu'" class="game-menu">
      <div class="player-info">
        <div class="player-card">
          <h3>👤 玩家信息</h3>
          <div class="player-stats">
            <div class="stat-item">
              <span class="stat-label">总分:</span>
              <span class="stat-value">{{ player.totalScore }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">游戏数:</span>
              <span class="stat-value">{{ player.gamesPlayed }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">成就:</span>
              <span class="stat-value">{{ player.achievements.length }}</span>
            </div>
          </div>
          
          <div v-if="player.achievements.length > 0" class="achievements">
            <h4>🏆 获得成就</h4>
            <div class="achievement-list">
              <span v-for="achievement in player.achievements" :key="achievement" class="achievement-badge">
                {{ achievement }}
              </span>
            </div>
          </div>
        </div>
      </div>
      
      <div class="games-grid">
        <div v-for="game in games" :key="game.id" 
             @click="currentGame = game.id"
             class="game-card">
          <div class="game-icon">{{ game.icon }}</div>
          <h3 class="game-name">{{ game.name }}</h3>
          <p class="game-description">{{ game.description }}</p>
          <div class="game-meta">
            <span class="game-difficulty">{{ game.difficulty }}</span>
            <span class="game-category">{{ game.category }}</span>
          </div>
          
          <div v-if="player.stats[game.id]" class="game-stats">
            <div class="best-score">
              最高: {{ player.stats[game.id].bestScore }}
            </div>
            <div class="play-count">
              游戏: {{ player.stats[game.id].played }}次
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 记忆翻牌游戏 -->
    <div v-else-if="currentGame === 'memory'" class="memory-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>🧠 记忆翻牌</h3>
        <div class="game-controls">
          <select v-model="memoryGame.difficulty" :disabled="memoryGame.isPlaying">
            <option value="easy">简单 (3×4)</option>
            <option value="medium">中等 (4×4)</option>
            <option value="hard">困难 (4×6)</option>
          </select>
          <button v-if="!memoryGame.isPlaying" @click="startMemoryGame">开始游戏</button>
        </div>
      </div>
      
      <div class="memory-info">
        <div class="info-item">
          <span>移动数:</span>
          <span>{{ memoryGame.moves }}</span>
        </div>
        <div class="info-item">
          <span>时间:</span>
          <span>{{ Math.floor(memoryGame.timer / 60) }}:{{ (memoryGame.timer % 60).toString().padStart(2, '0') }}</span>
        </div>
        <div class="info-item">
          <span>分数:</span>
          <span>{{ memoryGame.score }}</span>
        </div>
        <div class="info-item">
          <span>配对:</span>
          <span>{{ memoryGame.matchedPairs }}/{{ memoryGameConfig[memoryGame.difficulty].pairs }}</span>
        </div>
      </div>
      
      <div class="memory-board" :class="`difficulty-${memoryGame.difficulty}`">
        <div v-for="card in memoryGame.cards" :key="card.id"
             @click="flipCard(card)"
             :class="['memory-card', { 
               flipped: card.flipped, 
               matched: card.matched,
               disabled: !memoryGame.isPlaying 
             }]">
          <div class="card-front">?</div>
          <div class="card-back">{{ card.symbol }}</div>
        </div>
      </div>
      
      <div v-if="memoryGame.gameOver" class="game-result">
        <h3>🎉 游戏完成！</h3>
        <div class="result-stats">
          <p>用时: {{ Math.floor(memoryGame.timer / 60) }}分{{ memoryGame.timer % 60 }}秒</p>
          <p>移动数: {{ memoryGame.moves }}</p>
          <p>得分: {{ memoryGame.score }}</p>
        </div>
        <button @click="startMemoryGame">再玩一次</button>
      </div>
    </div>
    
    <!-- 猜数字游戏 -->
    <div v-else-if="currentGame === 'guess'" class="guess-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>🎯 猜数字</h3>
        <div class="game-controls">
          <select v-model="guessGame.difficulty" :disabled="guessGame.isPlaying">
            <option value="easy">简单 (1-50)</option>
            <option value="medium">中等 (1-100)</option>
            <option value="hard">困难 (1-500)</option>
          </select>
          <button v-if="!guessGame.isPlaying" @click="startGuessGame">开始游戏</button>
        </div>
      </div>
      
      <div class="guess-info">
        <div class="info-item">
          <span>范围:</span>
          <span>{{ guessGame.range.min }} - {{ guessGame.range.max }}</span>
        </div>
        <div class="info-item">
          <span>尝试:</span>
          <span>{{ guessGame.attempts }}/{{ guessGame.maxAttempts }}</span>
        </div>
        <div class="info-item">
          <span>分数:</span>
          <span>{{ guessGame.score }}</span>
        </div>
      </div>
      
      <div v-if="guessGame.isPlaying" class="guess-input">
        <input v-model="guessGame.userGuess" 
               @keyup.enter="makeGuess"
               type="number" 
               :min="guessGame.range.min" 
               :max="guessGame.range.max"
               placeholder="输入你的猜测">
        <button @click="makeGuess" :disabled="!guessGame.userGuess">提交</button>
      </div>
      
      <div class="guess-hints">
        <div v-for="(hint, index) in guessGame.hints" :key="index" class="hint-item">
          {{ hint }}
        </div>
      </div>
      
      <div v-if="guessGame.gameOver" class="game-result">
        <h3>{{ guessGame.won ? '🎉 恭喜获胜！' : '😔 游戏结束' }}</h3>
        <div class="result-stats">
          <p>答案: {{ guessGame.targetNumber }}</p>
          <p>尝试次数: {{ guessGame.attempts }}</p>
          <p>得分: {{ guessGame.score }}</p>
        </div>
        <button @click="startGuessGame">再玩一次</button>
      </div>
    </div>
    
    <!-- 反应测试游戏 -->
    <div v-else-if="currentGame === 'reaction'" class="reaction-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>⚡ 反应测试</h3>
      </div>
      
      <div class="reaction-stats">
        <div class="stat-card">
          <span class="stat-label">当前用时</span>
          <span class="stat-value">{{ reactionGame.reactionTime }}ms</span>
        </div>
        <div class="stat-card">
          <span class="stat-label">最佳成绩</span>
          <span class="stat-value">{{ reactionGame.bestTime || '-' }}ms</span>
        </div>
        <div class="stat-card">
          <span class="stat-label">平均用时</span>
          <span class="stat-value">{{ reactionGame.averageTime || '-' }}ms</span>
        </div>
        <div class="stat-card">
          <span class="stat-label">测试次数</span>
          <span class="stat-value">{{ reactionGame.attempts }}</span>
        </div>
      </div>
      
      <div class="reaction-area" 
           :class="[`phase-${reactionGame.phase}`]"
           @click="handleReactionClick">
        <div class="reaction-message">
          {{ reactionGame.message }}
        </div>
        
        <div v-if="reactionGame.phase === 'result'" class="reaction-result">
          <div class="reaction-time">{{ reactionGame.reactionTime }}ms</div>
          <div class="reaction-score">得分: {{ reactionGame.score }}</div>
        </div>
      </div>
      
      <div class="reaction-controls">
        <button v-if="reactionGame.phase === 'waiting'" @click="startReactionTest">
          开始测试
        </button>
        <button v-if="reactionGame.phase === 'result'" @click="startReactionTest">
          再次测试
        </button>
        <button v-if="reactionGame.phase === 'ready'" @click="resetReactionGame">
          取消测试
        </button>
      </div>
    </div>
    
    <!-- 打字游戏 -->
    <div v-else-if="currentGame === 'typing'" class="typing-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>⌨️ 打字游戏</h3>
        <button v-if="!typingGame.isPlaying" @click="startTypingGame">开始游戏</button>
      </div>
      
      <div class="typing-stats">
        <div class="stat-item">
          <span>WPM:</span>
          <span>{{ typingGame.wpm }}</span>
        </div>
        <div class="stat-item">
          <span>准确率:</span>
          <span>{{ typingGame.accuracy }}%</span>
        </div>
        <div class="stat-item">
          <span>错误:</span>
          <span>{{ typingGame.mistakes }}</span>
        </div>
        <div class="stat-item">
          <span>分数:</span>
          <span>{{ typingGame.score }}</span>
        </div>
      </div>
      
      <div class="typing-text">
        <div class="target-text">
          <span v-for="(char, index) in typingGame.currentText" :key="index"
                :class="[
                  'text-char',
                  {
                    'typed': index < typingGame.userInput.length,
                    'correct': index < typingGame.userInput.length && typingGame.userInput[index] === char,
                    'incorrect': index < typingGame.userInput.length && typingGame.userInput[index] !== char,
                    'current': index === typingGame.userInput.length
                  }
                ]">
            {{ char }}
          </span>
        </div>
      </div>
      
      <div v-if="typingGame.isPlaying" class="typing-input">
        <textarea v-model="typingGame.userInput" 
                  @input="handleTypingInput"
                  placeholder="开始输入上面的文字..."
                  :disabled="!typingGame.isPlaying">
        </textarea>
      </div>
      
      <div v-if="typingGame.gameOver" class="game-result">
        <h3>🎉 输入完成！</h3>
        <div class="result-stats">
          <p>WPM: {{ typingGame.wpm }}</p>
          <p>准确率: {{ typingGame.accuracy }}%</p>
          <p>错误数: {{ typingGame.mistakes }}</p>
          <p>得分: {{ typingGame.score }}</p>
        </div>
        <button @click="startTypingGame">再玩一次</button>
      </div>
    </div>
    
    <!-- 贪吃蛇游戏 -->
    <div v-else-if="currentGame === 'snake'" class="snake-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>🐍 贪吃蛇</h3>
        <div class="game-controls">
          <div class="score">分数: {{ snakeGame.score }}</div>
          <button v-if="!snakeGame.isPlaying" @click="startSnakeGame">开始游戏</button>
        </div>
      </div>
      
      <div class="snake-instructions">
        使用方向键或WASD控制蛇的移动
      </div>
      
      <div class="snake-board">
        <div v-for="y in snakeGame.board.height" :key="y" class="snake-row">
          <div v-for="x in snakeGame.board.width" :key="x" 
               :class="[
                 'snake-cell',
                 {
                   'snake-head': snakeGame.snake[0]?.x === x-1 && snakeGame.snake[0]?.y === y-1,
                   'snake-body': snakeGame.snake.slice(1).some(segment => segment.x === x-1 && segment.y === y-1),
                   'food': snakeGame.food.x === x-1 && snakeGame.food.y === y-1
                 }
               ]">
          </div>
        </div>
      </div>
      
      <div v-if="snakeGame.gameOver" class="game-result">
        <h3>🐍 游戏结束</h3>
        <div class="result-stats">
          <p>最终分数: {{ snakeGame.score }}</p>
          <p>蛇的长度: {{ snakeGame.snake.length }}</p>
        </div>
        <button @click="startSnakeGame">再玩一次</button>
      </div>
    </div>
    
    <!-- 石头剪刀布游戏 -->
    <div v-else-if="currentGame === 'rps'" class="rps-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>✂️ 石头剪刀布</h3>
        <button v-if="rpsGame.gameOver" @click="resetRPSGame">新游戏</button>
      </div>
      
      <div class="rps-info">
        <div class="round-info">
          第 {{ rpsGame.round }}/{{ rpsGame.maxRounds }} 轮
        </div>
        <div class="score-info">
          <span class="player-score">你: {{ rpsGame.playerScore }}</span>
          <span class="computer-score">电脑: {{ rpsGame.computerScore }}</span>
        </div>
      </div>
      
      <div v-if="rpsGame.round > 0" class="rps-result">
        <div class="choice-display">
          <div class="player-choice">
            <h4>你的选择</h4>
            <div class="choice-icon">
              {{ rpsGame.choices.find(c => c.name === rpsGame.playerChoice)?.emoji }}
            </div>
          </div>
          
          <div class="vs">VS</div>
          
          <div class="computer-choice">
            <h4>电脑选择</h4>
            <div class="choice-icon">
              {{ rpsGame.choices.find(c => c.name === rpsGame.computerChoice)?.emoji }}
            </div>
          </div>
        </div>
        
        <div class="round-result">{{ rpsGame.result }}</div>
      </div>
      
      <div v-if="!rpsGame.gameOver" class="rps-choices">
        <button v-for="choice in rpsGame.choices" :key="choice.name"
                @click="playRPS(choice.name)"
                class="choice-btn">
          <div class="choice-emoji">{{ choice.emoji }}</div>
          <div class="choice-label">{{ choice.label }}</div>
        </button>
      </div>
      
      <div v-if="rpsGame.gameOver" class="game-result">
        <h3>
          {{ rpsGame.playerScore > rpsGame.computerScore ? '🎉 你赢了！' : 
             rpsGame.playerScore < rpsGame.computerScore ? '😔 电脑赢了' : 
             '🤝 平局' }}
        </h3>
        <div class="final-score">
          {{ rpsGame.playerScore }} : {{ rpsGame.computerScore }}
        </div>
      </div>
      
      <div v-if="rpsGame.history.length > 0" class="rps-history">
        <h4>游戏记录</h4>
        <div class="history-list">
          <div v-for="record in rpsGame.history.slice(0, 5)" :key="record.round" 
               :class="['history-item', record.result]">
            <span class="round">第{{ record.round }}轮</span>
            <span class="choices">
              {{ rpsGame.choices.find(c => c.name === record.player)?.emoji }} vs 
              {{ rpsGame.choices.find(c => c.name === record.computer)?.emoji }}
            </span>
            <span class="result">
              {{ record.result === 'player' ? '胜' : record.result === 'computer' ? '负' : '平' }}
            </span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 知识问答游戏 -->
    <div v-else-if="currentGame === 'quiz'" class="quiz-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>🧭 知识问答</h3>
        <button v-if="!quizGame.isPlaying && quizGame.currentQuestion === 0" @click="startQuizGame">
          开始答题
        </button>
      </div>
      
      <div v-if="quizGame.isPlaying" class="quiz-progress">
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: (quizGame.currentQuestion / quizGame.questions.length) * 100 + '%' }"></div>
        </div>
        <div class="progress-text">
          {{ quizGame.currentQuestion + 1 }} / {{ quizGame.questions.length }}
        </div>
        <div class="timer">
          ⏰ {{ quizGame.timeLeft }}s
        </div>
      </div>
      
      <div v-if="quizGame.isPlaying" class="quiz-content">
        <div class="question">
          {{ quizGame.questions[quizGame.currentQuestion].question }}
        </div>
        
        <div class="options">
          <button v-for="(option, index) in quizGame.questions[quizGame.currentQuestion].options" 
                  :key="index"
                  @click="answerQuestion(index)"
                  :class="[
                    'option-btn',
                    {
                      'selected': quizGame.selectedAnswer === index,
                      'correct': quizGame.selectedAnswer !== null && index === quizGame.questions[quizGame.currentQuestion].correct,
                      'incorrect': quizGame.selectedAnswer === index && index !== quizGame.questions[quizGame.currentQuestion].correct
                    }
                  ]"
                  :disabled="quizGame.selectedAnswer !== null">
            {{ option }}
          </button>
        </div>
      </div>
      
      <div class="quiz-stats">
        <div class="stat-item">
          <span>正确:</span>
          <span>{{ quizGame.correctAnswers }}</span>
        </div>
        <div class="stat-item">
          <span>错误:</span>
          <span>{{ quizGame.wrongAnswers }}</span>
        </div>
        <div class="stat-item">
          <span>分数:</span>
          <span>{{ quizGame.score }}</span>
        </div>
      </div>
      
      <div v-if="quizGame.gameOver" class="game-result">
        <h3>📊 答题完成！</h3>
        <div class="result-stats">
          <p>正确率: {{ Math.round((quizGame.correctAnswers / quizGame.questions.length) * 100) }}%</p>
          <p>正确: {{ quizGame.correctAnswers }}</p>
          <p>错误: {{ quizGame.wrongAnswers }}</p>
          <p>得分: {{ quizGame.score }}</p>
        </div>
        <button @click="startQuizGame">再来一次</button>
      </div>
    </div>
    
    <!-- 颜色挑战游戏 -->
    <div v-else-if="currentGame === 'color'" class="color-game">
      <div class="game-header">
        <button @click="currentGame = 'menu'" class="back-btn">← 返回菜单</button>
        <h3>🎨 颜色挑战</h3>
        <button v-if="!colorGame.isPlaying" @click="startColorGame">开始挑战</button>
      </div>
      
      <div class="color-instructions">
        选择文字显示的颜色（不是文字内容）
      </div>
      
      <div v-if="colorGame.isPlaying" class="color-progress">
        <div class="progress-info">
          第 {{ colorGame.round }} / {{ colorGame.maxRounds }} 题
        </div>
        <div class="timer">⏰ {{ colorGame.timeLeft }}s</div>
        <div class="streak">连击: {{ colorGame.streak }}</div>
      </div>
      
      <div v-if="colorGame.isPlaying" class="color-challenge">
        <div class="color-word" :style="{ color: colorGame.currentColor }">
          {{ colorGame.currentWord }}
        </div>
        
        <div class="color-options">
          <button v-for="option in colorGame.options" :key="option"
                  @click="answerColorQuestion(option)"
                  :class="[
                    'color-option',
                    {
                      'selected': colorGame.selectedAnswer === option,
                      'correct': colorGame.selectedAnswer && option === colorGame.correctAnswer,
                      'incorrect': colorGame.selectedAnswer === option && option !== colorGame.correctAnswer
                    }
                  ]"
                  :disabled="colorGame.selectedAnswer !== ''">
            {{ option }}
          </button>
        </div>
      </div>
      
      <div class="color-stats">
        <div class="stat-item">
          <span>分数:</span>
          <span>{{ colorGame.score }}</span>
        </div>
        <div class="stat-item">
          <span>最高连击:</span>
          <span>{{ colorGame.maxStreak }}</span>
        </div>
      </div>
      
      <div v-if="colorGame.gameOver" class="game-result">
        <h3>🎨 挑战完成！</h3>
        <div class="result-stats">
          <p>最终分数: {{ colorGame.score }}</p>
          <p>最高连击: {{ colorGame.maxStreak }}</p>
          <p>准确率: {{ Math.round(((colorGame.maxRounds - colorGame.wrongAnswers) / colorGame.maxRounds) * 100) }}%</p>
        </div>
        <button @click="startColorGame">再次挑战</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.interactive-game {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

/* 通用样式 */
button {
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

button:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.back-btn {
  background: #757575;
  color: white;
}

/* 游戏菜单样式 */
.game-menu {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.player-info {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

.player-card {
  padding: 20px;
}

.player-card h3 {
  margin: 0 0 15px 0;
  color: #333;
}

.player-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  background: #f5f5f5;
  border-radius: 8px;
}

.stat-label {
  font-size: 14px;
  color: #666;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 20px;
  font-weight: bold;
  color: #42b883;
}

.achievements h4 {
  margin: 0 0 10px 0;
  color: #333;
}

.achievement-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.achievement-badge {
  background: linear-gradient(45deg, #ffd700, #ffed4e);
  color: #b7950b;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.games-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
}

.game-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  transition: all 0.3s;
  cursor: pointer;
  border: 2px solid transparent;
}

.game-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  border-color: #42b883;
}

.game-icon {
  font-size: 48px;
  text-align: center;
  margin-bottom: 15px;
}

.game-name {
  margin: 0 0 10px 0;
  color: #333;
  text-align: center;
}

.game-description {
  margin: 0 0 15px 0;
  color: #666;
  text-align: center;
  font-size: 14px;
}

.game-meta {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.game-difficulty,
.game-category {
  background: #e0e0e0;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  color: #666;
}

.game-stats {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #42b883;
  font-weight: bold;
}

/* 游戏头部样式 */
.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-wrap: wrap;
  gap: 15px;
}

.game-header h3 {
  margin: 0;
  color: #333;
}

.game-controls {
  display: flex;
  gap: 10px;
  align-items: center;
}

.score {
  font-weight: bold;
  color: #42b883;
}

/* 记忆游戏样式 */
.memory-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.info-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.info-item span:first-child {
  font-size: 14px;
  color: #666;
}

.info-item span:last-child {
  font-size: 18px;
  font-weight: bold;
  color: #42b883;
}

.memory-board {
  display: grid;
  gap: 10px;
  margin: 20px auto;
  max-width: 600px;
}

.memory-board.difficulty-easy {
  grid-template-columns: repeat(4, 1fr);
}

.memory-board.difficulty-medium {
  grid-template-columns: repeat(4, 1fr);
}

.memory-board.difficulty-hard {
  grid-template-columns: repeat(6, 1fr);
}

.memory-card {
  aspect-ratio: 1;
  background: #f0f0f0;
  border-radius: 8px;
  cursor: pointer;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
  border: 2px solid #ddd;
}

.memory-card:hover {
  border-color: #42b883;
}

.memory-card.flipped {
  transform: rotateY(180deg);
}

.memory-card.matched {
  background: #e8f5e9;
  border-color: #4caf50;
}

.memory-card.disabled {
  cursor: not-allowed;
  opacity: 0.7;
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  font-weight: bold;
  border-radius: 6px;
  backface-visibility: hidden;
}

.card-front {
  background: #42b883;
  color: white;
}

.card-back {
  background: white;
  transform: rotateY(180deg);
}

/* 猜数字游戏样式 */
.guess-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.guess-input {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.guess-input input {
  flex: 1;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.guess-input input:focus {
  outline: none;
  border-color: #42b883;
}

.guess-input button {
  background: #42b883;
  color: white;
  padding: 10px 20px;
}

.guess-hints {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  min-height: 200px;
  max-height: 300px;
  overflow-y: auto;
}

.hint-item {
  padding: 8px 0;
  border-bottom: 1px solid #eee;
  color: #333;
}

.hint-item:last-child {
  border-bottom: none;
}

/* 反应测试游戏样式 */
.reaction-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.stat-card .stat-label {
  display: block;
  font-size: 14px;
  color: #666;
  margin-bottom: 5px;
}

.stat-card .stat-value {
  display: block;
  font-size: 20px;
  font-weight: bold;
  color: #42b883;
}

.reaction-area {
  height: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  margin-bottom: 20px;
  user-select: none;
}

.reaction-area.phase-waiting {
  background: #f5f5f5;
  color: #666;
}

.reaction-area.phase-ready {
  background: #ffebee;
  color: #d32f2f;
}

.reaction-area.phase-go {
  background: #e8f5e9;
  color: #2e7d32;
}

.reaction-area.phase-result {
  background: #e3f2fd;
  color: #1976d2;
}

.reaction-message {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
}

.reaction-result {
  text-align: center;
}

.reaction-time {
  font-size: 36px;
  font-weight: bold;
  margin-bottom: 10px;
}

.reaction-score {
  font-size: 18px;
}

.reaction-controls {
  text-align: center;
}

.reaction-controls button {
  background: #42b883;
  color: white;
  padding: 12px 24px;
  font-size: 16px;
}

/* 打字游戏样式 */
.typing-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.typing-text {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.target-text {
  font-size: 18px;
  line-height: 1.6;
  font-family: 'Courier New', monospace;
}

.text-char {
  position: relative;
}

.text-char.typed.correct {
  background: #e8f5e9;
  color: #2e7d32;
}

.text-char.typed.incorrect {
  background: #ffebee;
  color: #d32f2f;
}

.text-char.current {
  background: #42b883;
  color: white;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0.3; }
}

.typing-input {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.typing-input textarea {
  width: 100%;
  height: 100px;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  font-family: 'Courier New', monospace;
  resize: none;
}

.typing-input textarea:focus {
  outline: none;
  border-color: #42b883;
}

/* 贪吃蛇游戏样式 */
.snake-instructions {
  text-align: center;
  color: #666;
  margin-bottom: 15px;
  padding: 10px;
  background: #f9f9f9;
  border-radius: 4px;
}

.snake-board {
  display: flex;
  flex-direction: column;
  border: 2px solid #333;
  background: #000;
  margin: 0 auto;
  width: fit-content;
}

.snake-row {
  display: flex;
}

.snake-cell {
  width: 20px;
  height: 20px;
  border: 1px solid #333;
}

.snake-cell.snake-head {
  background: #42b883;
}

.snake-cell.snake-body {
  background: #66d9a2;
}

.snake-cell.food {
  background: #f44336;
  border-radius: 50%;
}

/* 石头剪刀布游戏样式 */
.rps-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.round-info {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.score-info {
  display: flex;
  gap: 20px;
}

.player-score,
.computer-score {
  font-size: 16px;
  font-weight: bold;
  color: #42b883;
}

.rps-result {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  text-align: center;
}

.choice-display {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 20px;
}

.player-choice,
.computer-choice {
  text-align: center;
}

.player-choice h4,
.computer-choice h4 {
  margin: 0 0 10px 0;
  color: #333;
}

.choice-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.vs {
  font-size: 24px;
  font-weight: bold;
  color: #666;
}

.round-result {
  font-size: 24px;
  font-weight: bold;
  color: #42b883;
}

.rps-choices {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.choice-btn {
  background: white;
  border: 2px solid #ddd;
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  transition: all 0.3s;
  cursor: pointer;
}

.choice-btn:hover {
  border-color: #42b883;
  transform: translateY(-2px);
}

.choice-emoji {
  font-size: 48px;
  margin-bottom: 10px;
}

.choice-label {
  font-size: 16px;
  font-weight: bold;
  color: #333;
}

.final-score {
  font-size: 36px;
  font-weight: bold;
  margin: 10px 0;
  color: #42b883;
}

.rps-history {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.rps-history h4 {
  margin: 0 0 15px 0;
  color: #333;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.history-item {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 15px;
  align-items: center;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
}

.history-item.player {
  background: #e8f5e9;
  color: #2e7d32;
}

.history-item.computer {
  background: #ffebee;
  color: #d32f2f;
}

.history-item.tie {
  background: #f5f5f5;
  color: #666;
}

/* 知识问答游戏样式 */
.quiz-progress {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.progress-bar {
  flex: 1;
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  margin-right: 15px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #42b883;
  transition: width 0.3s;
}

.progress-text {
  font-weight: bold;
  color: #333;
  margin-right: 15px;
}

.timer {
  font-weight: bold;
  color: #f44336;
  font-size: 16px;
}

.quiz-content {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.question {
  font-size: 20px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
  text-align: center;
}

.options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

.option-btn {
  padding: 15px;
  background: #f5f5f5;
  border: 2px solid #ddd;
  border-radius: 8px;
  text-align: left;
  transition: all 0.3s;
  cursor: pointer;
}

.option-btn:hover:not(:disabled) {
  border-color: #42b883;
  background: #f0f8f5;
}

.option-btn:disabled {
  cursor: not-allowed;
}

.option-btn.selected {
  border-color: #42b883;
  background: #e8f5e9;
}

.option-btn.correct {
  border-color: #4caf50;
  background: #e8f5e9;
  color: #2e7d32;
}

.option-btn.incorrect {
  border-color: #f44336;
  background: #ffebee;
  color: #d32f2f;
}

.quiz-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 颜色挑战游戏样式 */
.color-instructions {
  text-align: center;
  color: #666;
  margin-bottom: 15px;
  padding: 10px;
  background: #fff3e0;
  border-radius: 4px;
  border: 1px solid #ff9800;
}

.color-progress {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.progress-info {
  font-weight: bold;
  color: #333;
}

.streak {
  font-weight: bold;
  color: #ff9800;
}

.color-challenge {
  background: white;
  border-radius: 8px;
  padding: 30px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  text-align: center;
}

.color-word {
  font-size: 48px;
  font-weight: bold;
  margin-bottom: 30px;
  padding: 20px;
  border-radius: 8px;
  background: #f9f9f9;
}

.color-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
}

.color-option {
  padding: 15px;
  background: #f5f5f5;
  border: 2px solid #ddd;
  border-radius: 8px;
  transition: all 0.3s;
  cursor: pointer;
  font-size: 16px;
}

.color-option:hover:not(:disabled) {
  border-color: #42b883;
  background: #f0f8f5;
}

.color-option:disabled {
  cursor: not-allowed;
}

.color-option.selected {
  border-color: #42b883;
  background: #e8f5e9;
}

.color-option.correct {
  border-color: #4caf50;
  background: #e8f5e9;
  color: #2e7d32;
}

.color-option.incorrect {
  border-color: #f44336;
  background: #ffebee;
  color: #d32f2f;
}

.color-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 游戏结果样式 */
.game-result {
  background: white;
  border-radius: 8px;
  padding: 30px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  text-align: center;
  margin-top: 20px;
}

.game-result h3 {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 24px;
}

.result-stats {
  margin: 20px 0;
  color: #666;
}

.result-stats p {
  margin: 8px 0;
  font-size: 16px;
}

.game-result button {
  background: #42b883;
  color: white;
  padding: 12px 24px;
  font-size: 16px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .games-grid {
    grid-template-columns: 1fr;
  }
  
  .game-header {
    flex-direction: column;
    align-items: stretch;
  }
  
  .memory-board {
    max-width: 100%;
  }
  
  .memory-board.difficulty-hard {
    grid-template-columns: repeat(4, 1fr);
  }
  
  .reaction-stats {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .rps-choices {
    grid-template-columns: 1fr;
  }
  
  .options {
    grid-template-columns: 1fr;
  }
  
  .color-options {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .choice-display {
    flex-direction: column;
    gap: 20px;
  }
  
  .vs {
    transform: rotate(90deg);
  }
  
  .player-stats {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 480px) {
  .memory-board.difficulty-medium,
  .memory-board.difficulty-hard {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .reaction-stats {
    grid-template-columns: 1fr;
  }
  
  .color-options {
    grid-template-columns: 1fr;
  }
}
</style>