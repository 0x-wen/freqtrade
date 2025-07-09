# FreqTrade 使用指南和配置说明

## 目录
- [1. 快速开始](#1-快速开始)
- [2. 安装配置](#2-安装配置)
- [3. 基础配置](#3-基础配置)
- [4. 交易策略开发](#4-交易策略开发)
- [5. 回测与优化](#5-回测与优化)
- [6. 实盘交易](#6-实盘交易)
- [7. 监控与管理](#7-监控与管理)
- [8. 高级功能](#8-高级功能)
- [9. 故障排除](#9-故障排除)
- [10. 最佳实践](#10-最佳实践)

## 1. 快速开始

### 1.1 系统要求

- **Python**: 3.10 或更高版本
- **内存**: 最少 2GB RAM
- **存储**: 至少 1GB 可用空间
- **网络**: 稳定的网络连接

### 1.2 安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/freqtrade/freqtrade.git
cd freqtrade

# 2. 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 3. 安装依赖
pip install -e .

# 4. 安装 TA-Lib (技术分析库)
# Linux/Mac
pip install TA-Lib

# Windows
# 下载预编译包: https://www.lfd.uci.edu/~gohlke/pythonlibs/#ta-lib
pip install TA_Lib-0.4.28-cp310-cp310-win_amd64.whl
```

### 1.3 初始化配置

```bash
# 创建用户数据目录
freqtrade create-userdir --userdir user_data

# 创建配置文件
freqtrade new-config --config user_data/config.json

# 创建示例策略
freqtrade new-strategy --strategy MyStrategy --userdir user_data
```

## 2. 安装配置

### 2.1 Docker 安装 (推荐)

```bash
# 拉取镜像
docker pull freqtradeorg/freqtrade:stable

# 创建配置目录
mkdir ft_userdata
cd ft_userdata

# 创建配置文件
docker run --rm -it -v $(pwd):/freqtrade/user_data freqtradeorg/freqtrade:stable new-config --config user_data/config.json

# 运行 FreqTrade
docker run -d \
  --name freqtrade \
  -v $(pwd):/freqtrade/user_data \
  freqtradeorg/freqtrade:stable trade --config user_data/config.json --strategy SampleStrategy
```

### 2.2 开发环境配置

```bash
# 安装开发依赖
pip install -e .[dev]

# 安装 pre-commit 钩子
pre-commit install

# 运行测试
pytest tests/

# 代码格式化
ruff format .
ruff check .
```

### 2.3 生产环境配置

```bash
# 创建系统服务
sudo cp freqtrade.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable freqtrade
sudo systemctl start freqtrade

# 查看状态
sudo systemctl status freqtrade
```

## 3. 基础配置

### 3.1 核心配置文件结构

```json
{
  "max_open_trades": 5,
  "stake_currency": "USDT",
  "stake_amount": 100,
  "tradable_balance_ratio": 0.99,
  "fiat_display_currency": "USD",
  "timeframe": "5m",
  "dry_run": true,
  "dry_run_wallet": 1000,
  
  "exchange": {
    "name": "binance",
    "key": "your-api-key",
    "secret": "your-api-secret",
    "ccxt_config": {},
    "ccxt_async_config": {},
    "pair_whitelist": [
      "BTC/USDT",
      "ETH/USDT",
      "ADA/USDT",
      "DOT/USDT",
      "SOL/USDT"
    ],
    "pair_blacklist": []
  },
  
  "entry_pricing": {
    "price_side": "same",
    "use_order_book": true,
    "order_book_top": 1,
    "price_last_balance": 0.0,
    "check_depth_of_market": {
      "enabled": false,
      "bids_to_ask_delta": 1
    }
  },
  
  "exit_pricing": {
    "price_side": "same",
    "use_order_book": true,
    "order_book_top": 1
  },
  
  "pairlists": [
    {
      "method": "StaticPairList"
    }
  ],
  
  "protections": [],
  
  "telegram": {
    "enabled": false,
    "token": "your-telegram-token",
    "chat_id": "your-chat-id"
  },
  
  "api_server": {
    "enabled": false,
    "listen_ip_address": "127.0.0.1",
    "listen_port": 8080,
    "verbosity": "error",
    "enable_openapi": false,
    "jwt_secret_key": "your-secret-key",
    "CORS_origins": [],
    "username": "freqtrade",
    "password": "your-password"
  },
  
  "bot_name": "freqtrade",
  "initial_state": "running",
  "force_entry_enable": false,
  "internals": {
    "process_throttle_secs": 5
  }
}
```

### 3.2 交易所配置

#### Binance 配置示例

```json
{
  "exchange": {
    "name": "binance",
    "key": "your-api-key",
    "secret": "your-api-secret",
    "ccxt_config": {
      "enableRateLimit": true,
      "rateLimit": 200,
      "options": {
        "defaultType": "spot"
      }
    },
    "ccxt_async_config": {
      "enableRateLimit": true,
      "rateLimit": 200
    }
  }
}
```

#### Binance Futures 配置

```json
{
  "exchange": {
    "name": "binance",
    "key": "your-api-key",
    "secret": "your-api-secret",
    "ccxt_config": {
      "options": {
        "defaultType": "future"
      }
    }
  },
  "trading_mode": "futures",
  "margin_mode": "isolated",
  "liquidation_buffer": 0.05
}
```

### 3.3 风险管理配置

```json
{
  "max_open_trades": 5,
  "stake_amount": "unlimited",
  "tradable_balance_ratio": 0.99,
  "amount_reserve_percent": 0.05,
  
  "unfilledtimeout": {
    "entry": 10,
    "exit": 10,
    "exit_timeout_count": 0,
    "unit": "minutes"
  },
  
  "order_types": {
    "entry": "limit",
    "exit": "limit",
    "emergency_exit": "market",
    "force_exit": "market",
    "force_entry": "market",
    "stoploss": "market",
    "stoploss_on_exchange": false,
    "stoploss_on_exchange_interval": 60,
    "stoploss_on_exchange_market_ratio": 0.99
  },
  
  "order_time_in_force": {
    "entry": "GTC",
    "exit": "GTC"
  }
}
```

## 4. 交易策略开发

### 4.1 策略基础结构

```python
# user_data/strategies/MyStrategy.py
from freqtrade.strategy import IStrategy
from pandas import DataFrame
import talib.abstract as ta
import pandas_ta as pta

class MyStrategy(IStrategy):
    """
    示例交易策略
    """
    
    # 策略元数据
    INTERFACE_VERSION = 3
    
    # 最小ROI表 - 定义不同时间的最小收益率
    minimal_roi = {
        "60": 0.01,    # 60分钟后至少1%收益
        "30": 0.02,    # 30分钟后至少2%收益
        "0": 0.04      # 立即至少4%收益
    }
    
    # 止损设置
    stoploss = -0.05  # 5%止损
    
    # 跟踪止损 (可选)
    trailing_stop = True
    trailing_stop_positive = 0.01
    trailing_stop_positive_offset = 0.02
    trailing_only_offset_is_reached = False
    
    # 时间框架
    timeframe = '5m'
    
    # 启动时需要的历史数据量
    startup_candle_count: int = 30
    
    # 订单类型
    order_types = {
        'entry': 'limit',
        'exit': 'limit',
        'stoploss': 'market',
        'stoploss_on_exchange': False
    }
    
    # 订单时间有效性
    order_time_in_force = {
        'entry': 'GTC',
        'exit': 'GTC'
    }
    
    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        添加技术指标
        """
        # 移动平均线
        dataframe['sma_20'] = ta.SMA(dataframe, timeperiod=20)
        dataframe['ema_12'] = ta.EMA(dataframe, timeperiod=12)
        dataframe['ema_26'] = ta.EMA(dataframe, timeperiod=26)
        
        # MACD
        macd = ta.MACD(dataframe)
        dataframe['macd'] = macd['macd']
        dataframe['macdsignal'] = macd['macdsignal']
        dataframe['macdhist'] = macd['macdhist']
        
        # RSI
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
        
        # 布林带
        bollinger = ta.BBANDS(dataframe, timeperiod=20)
        dataframe['bb_lower'] = bollinger['lowerband']
        dataframe['bb_middle'] = bollinger['middleband']
        dataframe['bb_upper'] = bollinger['upperband']
        
        # 成交量移动平均
        dataframe['volume_sma'] = ta.SMA(dataframe['volume'], timeperiod=20)
        
        # ATR (平均真实范围)
        dataframe['atr'] = ta.ATR(dataframe, timeperiod=14)
        
        return dataframe
    
    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        入场信号逻辑
        """
        dataframe.loc[
            (
                # 趋势过滤
                (dataframe['close'] > dataframe['sma_20']) &
                (dataframe['ema_12'] > dataframe['ema_26']) &
                
                # 动量确认
                (dataframe['rsi'] > 30) &
                (dataframe['rsi'] < 70) &
                (dataframe['macd'] > dataframe['macdsignal']) &
                
                # 价格位置
                (dataframe['close'] > dataframe['bb_lower']) &
                (dataframe['close'] < dataframe['bb_upper']) &
                
                # 成交量确认
                (dataframe['volume'] > dataframe['volume_sma']) &
                
                # 波动率过滤
                (dataframe['atr'] > dataframe['atr'].rolling(20).mean() * 0.8)
            ),
            'enter_long'] = 1
        
        return dataframe
    
    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        出场信号逻辑
        """
        dataframe.loc[
            (
                # 趋势反转
                (dataframe['ema_12'] < dataframe['ema_26']) |
                
                # 超买状态
                (dataframe['rsi'] > 70) |
                
                # MACD死叉
                (dataframe['macd'] < dataframe['macdsignal']) |
                
                # 价格触碰上轨
                (dataframe['close'] > dataframe['bb_upper'])
            ),
            'exit_long'] = 1
        
        return dataframe
    
    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                       current_rate: float, current_profit: float, **kwargs) -> float:
        """
        自定义止损逻辑
        """
        # 根据ATR调整止损
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        last_candle = dataframe.iloc[-1]
        
        # 基于ATR的动态止损
        atr_stop = last_candle['atr'] * 2
        stop_price = current_rate - atr_stop
        
        # 计算止损百分比
        stoploss_pct = (stop_price / current_rate) - 1
        
        # 确保不高于最大止损
        return max(stoploss_pct, self.stoploss)
    
    def custom_exit(self, pair: str, trade: 'Trade', current_time: datetime,
                   current_rate: float, current_profit: float, **kwargs) -> Union[str, bool]:
        """
        自定义退出逻辑
        """
        # 获取当前数据
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        last_candle = dataframe.iloc[-1]
        
        # 时间退出 - 持仓超过4小时
        if current_time - trade.open_date_utc > timedelta(hours=4):
            return 'time_exit'
        
        # 利润退出 - 达到10%收益
        if current_profit > 0.10:
            return 'profit_exit'
        
        # RSI极端超买
        if last_candle['rsi'] > 85:
            return 'rsi_overbought'
        
        return False
```

### 4.2 多时间框架策略

```python
from freqtrade.strategy import IStrategy, merge_informative_pair
from pandas import DataFrame
import talib.abstract as ta

class MultiTimeframeStrategy(IStrategy):
    """
    多时间框架策略示例
    """
    
    # 主时间框架
    timeframe = '5m'
    
    # 信息时间框架
    informative_timeframe = '1h'
    
    def informative_pairs(self):
        """
        定义信息时间框架的交易对
        """
        pairs = self.dp.current_whitelist()
        informative_pairs = [(pair, self.informative_timeframe) for pair in pairs]
        return informative_pairs
    
    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        添加主时间框架指标
        """
        # 主时间框架指标
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
        dataframe['macd'] = ta.MACD(dataframe)['macd']
        dataframe['ema_20'] = ta.EMA(dataframe, timeperiod=20)
        
        # 获取信息时间框架数据
        informative = self.dp.get_pair_dataframe(
            pair=metadata['pair'], 
            timeframe=self.informative_timeframe
        )
        
        # 添加信息时间框架指标
        informative['rsi_1h'] = ta.RSI(informative, timeperiod=14)
        informative['ema_50_1h'] = ta.EMA(informative, timeperiod=50)
        informative['trend_1h'] = (informative['close'] > informative['ema_50_1h']).astype(int)
        
        # 合并信息时间框架数据
        dataframe = merge_informative_pair(
            dataframe, 
            informative, 
            self.timeframe, 
            self.informative_timeframe, 
            ffill=True
        )
        
        return dataframe
    
    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        多时间框架入场逻辑
        """
        dataframe.loc[
            (
                # 1小时时间框架趋势向上
                (dataframe['trend_1h'] == 1) &
                
                # 1小时RSI不超买
                (dataframe['rsi_1h'] < 70) &
                
                # 5分钟时间框架入场信号
                (dataframe['close'] > dataframe['ema_20']) &
                (dataframe['rsi'] > 30) &
                (dataframe['rsi'] < 70) &
                (dataframe['macd'] > 0)
            ),
            'enter_long'] = 1
        
        return dataframe
    
    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        多时间框架出场逻辑
        """
        dataframe.loc[
            (
                # 1小时趋势反转
                (dataframe['trend_1h'] == 0) |
                
                # 5分钟超买
                (dataframe['rsi'] > 80) |
                
                # MACD转负
                (dataframe['macd'] < 0)
            ),
            'exit_long'] = 1
        
        return dataframe
```

## 5. 回测与优化

### 5.1 基础回测

```bash
# 下载数据
freqtrade download-data \
    --exchange binance \
    --pairs BTC/USDT ETH/USDT ADA/USDT \
    --timeframes 5m 1h \
    --days 30

# 运行回测
freqtrade backtesting \
    --config user_data/config.json \
    --strategy MyStrategy \
    --timeframe 5m \
    --timerange 20231201-20231231

# 显示回测结果
freqtrade backtesting-show \
    --config user_data/config.json \
    --strategy MyStrategy
```

### 5.2 回测配置

```json
{
  "backtesting": {
    "max_open_trades": 5,
    "stake_amount": 100,
    "fee": 0.001,
    "timeframe": "5m",
    "timerange": "20231201-20231231",
    "enable_position_stacking": false,
    "position_stacking": false,
    "use_max_market_positions": true,
    "cache": "day"
  }
}
```

### 5.3 超参数优化

```bash
# 创建超参数优化配置
freqtrade hyperopt \
    --config user_data/config.json \
    --hyperopt-loss SharpeHyperOptLoss \
    --strategy MyStrategy \
    --epochs 100 \
    --spaces entry exit \
    --jobs 4

# 查看优化结果
freqtrade hyperopt-list \
    --config user_data/config.json \
    --profitable \
    --min-trades 10 \
    --max-trades 100 \
    --hyperopt-filename hyperopt_results.pickle

# 显示最佳结果
freqtrade hyperopt-show \
    --config user_data/config.json \
    --hyperopt-filename hyperopt_results.pickle \
    --best
```

### 5.4 策略优化示例

```python
from freqtrade.strategy import IStrategy, IntParameter, DecimalParameter, CategoricalParameter

class OptimizedStrategy(IStrategy):
    """
    可优化的策略示例
    """
    
    # 可优化参数
    rsi_buy = IntParameter(20, 40, default=30, space="entry")
    rsi_sell = IntParameter(60, 80, default=70, space="exit")
    
    ema_short = IntParameter(5, 20, default=12, space="entry")
    ema_long = IntParameter(20, 50, default=26, space="entry")
    
    stoploss_value = DecimalParameter(-0.15, -0.05, default=-0.10, space="protection")
    
    roi_t1 = IntParameter(10, 120, default=60, space="roi")
    roi_t2 = IntParameter(10, 60, default=30, space="roi")
    roi_t3 = IntParameter(10, 40, default=20, space="roi")
    
    roi_p1 = DecimalParameter(0.01, 0.04, default=0.01, space="roi")
    roi_p2 = DecimalParameter(0.01, 0.07, default=0.02, space="roi")
    roi_p3 = DecimalParameter(0.01, 0.20, default=0.03, space="roi")
    
    @property
    def minimal_roi(self):
        """
        动态ROI配置
        """
        return {
            "0": self.roi_p1.value,
            str(self.roi_t3.value): self.roi_p2.value,
            str(self.roi_t2.value): self.roi_p3.value,
            str(self.roi_t1.value): 0
        }
    
    @property
    def stoploss(self):
        """
        动态止损
        """
        return self.stoploss_value.value
    
    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        使用优化参数计算指标
        """
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
        dataframe['ema_short'] = ta.EMA(dataframe, timeperiod=self.ema_short.value)
        dataframe['ema_long'] = ta.EMA(dataframe, timeperiod=self.ema_long.value)
        
        return dataframe
    
    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        使用优化参数的入场逻辑
        """
        dataframe.loc[
            (
                (dataframe['rsi'] < self.rsi_buy.value) &
                (dataframe['ema_short'] > dataframe['ema_long'])
            ),
            'enter_long'] = 1
        
        return dataframe
    
    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        使用优化参数的出场逻辑
        """
        dataframe.loc[
            (
                (dataframe['rsi'] > self.rsi_sell.value) |
                (dataframe['ema_short'] < dataframe['ema_long'])
            ),
            'exit_long'] = 1
        
        return dataframe
```

## 6. 实盘交易

### 6.1 实盘交易准备

```bash
# 1. 验证配置
freqtrade show-config --config user_data/config.json

# 2. 测试交易所连接
freqtrade list-exchanges

# 3. 检查余额
freqtrade balance --config user_data/config.json

# 4. 验证策略
freqtrade test-pairlist --config user_data/config.json

# 5. 运行模拟交易
freqtrade trade --config user_data/config.json --dry-run
```

### 6.2 实盘配置

```json
{
  "dry_run": false,
  "dry_run_wallet": 0,
  
  "exchange": {
    "name": "binance",
    "key": "your-real-api-key",
    "secret": "your-real-api-secret",
    "password": "",
    "sandbox": false
  },
  
  "max_open_trades": 3,
  "stake_amount": 50,
  "tradable_balance_ratio": 0.95,
  "amount_reserve_percent": 0.05,
  
  "order_types": {
    "entry": "limit",
    "exit": "limit",
    "stoploss": "limit",
    "stoploss_on_exchange": true,
    "stoploss_on_exchange_interval": 60
  },
  
  "unfilledtimeout": {
    "entry": 5,
    "exit": 5,
    "unit": "minutes"
  },
  
  "telegram": {
    "enabled": true,
    "token": "your-telegram-token",
    "chat_id": "your-chat-id",
    "notification_settings": {
      "status": "on",
      "warning": "on",
      "startup": "on",
      "entry": "on",
      "entry_cancel": "on",
      "entry_fill": "on",
      "exit": "on",
      "exit_cancel": "on",
      "exit_fill": "on",
      "protection_trigger": "on",
      "protection_trigger_global": "on"
    }
  }
}
```

### 6.3 启动实盘交易

```bash
# 启动实盘交易
freqtrade trade --config user_data/config.json --strategy MyStrategy

# 后台运行
nohup freqtrade trade --config user_data/config.json --strategy MyStrategy > trade.log 2>&1 &

# 使用 systemd 服务
sudo systemctl start freqtrade
sudo systemctl enable freqtrade
```

## 7. 监控与管理

### 7.1 Web UI 配置

```json
{
  "api_server": {
    "enabled": true,
    "listen_ip_address": "0.0.0.0",
    "listen_port": 8080,
    "verbosity": "info",
    "enable_openapi": true,
    "jwt_secret_key": "your-secret-key",
    "username": "freqtrade",
    "password": "your-password",
    "ws_token": "your-ws-token"
  }
}
```

```bash
# 安装 FreqUI
freqtrade install-ui

# 启动 Web 服务器
freqtrade webserver --config user_data/config.json
```

### 7.2 Telegram 机器人配置

```json
{
  "telegram": {
    "enabled": true,
    "token": "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11",
    "chat_id": "123456789",
    "balance_dust_level": 0.01,
    "notification_settings": {
      "status": "silent",
      "warning": "on",
      "startup": "on",
      "entry": "silent",
      "entry_cancel": "on",
      "entry_fill": "silent",
      "exit": "silent",
      "exit_cancel": "on",
      "exit_fill": "silent",
      "protection_trigger": "on",
      "protection_trigger_global": "on"
    },
    "reload": true,
    "balance_dust_level": 0.01
  }
}
```

**常用 Telegram 命令**:
```
/start - 启动机器人
/stop - 停止机器人
/status - 查看交易状态
/profit - 查看收益
/balance - 查看余额
/daily - 每日统计
/forceexit - 强制退出交易
/reload_config - 重新加载配置
/help - 帮助信息
```

### 7.3 监控脚本

```bash
#!/bin/bash
# monitor_freqtrade.sh

LOG_FILE="/var/log/freqtrade.log"
PID_FILE="/var/run/freqtrade.pid"
CONFIG_FILE="/path/to/config.json"

check_process() {
    if [ -f "$PID_FILE" ]; then
        PID=$(cat "$PID_FILE")
        if ps -p $PID > /dev/null 2>&1; then
            echo "FreqTrade is running (PID: $PID)"
            return 0
        else
            echo "FreqTrade is not running"
            return 1
        fi
    else
        echo "PID file not found"
        return 1
    fi
}

restart_freqtrade() {
    echo "Restarting FreqTrade..."
    pkill -f freqtrade
    sleep 5
    
    nohup freqtrade trade --config "$CONFIG_FILE" > "$LOG_FILE" 2>&1 &
    echo $! > "$PID_FILE"
    echo "FreqTrade restarted"
}

# 检查进程
if ! check_process; then
    restart_freqtrade
fi

# 检查日志中的错误
if grep -q "CRITICAL\|ERROR" "$LOG_FILE"; then
    echo "Found errors in log file"
    # 发送告警通知
    curl -X POST "https://api.telegram.org/bot$TELEGRAM_TOKEN/sendMessage" \
         -d "chat_id=$CHAT_ID" \
         -d "text=FreqTrade Error Detected!"
fi
```

## 8. 高级功能

### 8.1 FreqAI 配置

```json
{
  "freqai": {
    "enabled": true,
    "startup_candles": 10000,
    "purge_old_models": true,
    "train_period_days": 30,
    "backtest_period_days": 7,
    "live_retrain_hours": 0,
    "expiration_hours": 1,
    "identifier": "example",
    "feature_parameters": {
      "include_timeframes": ["5m", "15m", "4h"],
      "include_corr_pairlist": ["ETH/USD", "LINK/USD", "BNB/USD"],
      "label_period_candles": 24,
      "include_shifted_candles": 2,
      "DI_threshold": 0.9,
      "weight_factor": 0.9,
      "principal_component_analysis": false,
      "use_SVM_to_remove_outliers": true,
      "plot_feature_importances": 0
    },
    "data_split_parameters": {
      "test_size": 0.33,
      "shuffle": false
    },
    "model_training_parameters": {
      "n_estimators": 800
    }
  }
}
```

### 8.2 多货币对配置

```json
{
  "pairlists": [
    {
      "method": "VolumePairList",
      "number_assets": 20,
      "sort_key": "quoteVolume",
      "min_value": 0,
      "refresh_period": 1800
    },
    {
      "method": "AgeFilter",
      "min_days_listed": 10
    },
    {
      "method": "PrecisionFilter"
    },
    {
      "method": "PriceFilter",
      "low_price_ratio": 0.01
    },
    {
      "method": "SpreadFilter",
      "max_spread_ratio": 0.005
    },
    {
      "method": "RangeStabilityFilter",
      "lookback_days": 10,
      "min_rate_of_change": 0.01,
      "refresh_period": 1440
    }
  ]
}
```

### 8.3 保护机制配置

```json
{
  "protections": [
    {
      "method": "StoplossGuard",
      "lookback_period_candles": 60,
      "trade_limit": 4,
      "stop_duration_candles": 60,
      "only_per_pair": false
    },
    {
      "method": "MaxDrawdown",
      "lookback_period_candles": 200,
      "trade_limit": 20,
      "stop_duration_candles": 60,
      "max_allowed_drawdown": 0.2
    },
    {
      "method": "LowProfitPairs",
      "lookback_period_candles": 6000,
      "trade_limit": 2,
      "stop_duration": 60,
      "required_profit": 0.02
    },
    {
      "method": "CooldownPeriod",
      "stop_duration_candles": 20
    }
  ]
}
```

### 8.4 杠杆交易配置

```json
{
  "trading_mode": "futures",
  "margin_mode": "isolated",
  "liquidation_buffer": 0.05,
  
  "leverage": {
    "BTC/USDT": 3,
    "ETH/USDT": 2,
    "default": 1
  },
  
  "collateral_type": "base",
  
  "position_adjustment_enable": true,
  "max_entry_position_adjustment": 3,
  
  "order_types": {
    "entry": "limit",
    "exit": "limit",
    "stoploss": "limit",
    "stoploss_on_exchange": true,
    "stoploss_on_exchange_interval": 60,
    "stoploss_on_exchange_limit_ratio": 0.99
  }
}
```

## 9. 故障排除

### 9.1 常见问题

#### 问题1: 交易所连接失败
```bash
# 检查 API 密钥
freqtrade list-exchanges

# 测试连接
freqtrade balance --config user_data/config.json

# 检查防火墙设置
curl -I https://api.binance.com/api/v3/ping
```

#### 问题2: 策略无法加载
```bash
# 检查策略语法
python -m py_compile user_data/strategies/MyStrategy.py

# 列出可用策略
freqtrade list-strategies --userdir user_data

# 验证策略
freqtrade test-pairlist --config user_data/config.json --strategy MyStrategy
```

#### 问题3: 数据下载失败
```bash
# 检查网络连接
curl -I https://api.binance.com/api/v3/ping

# 清理缓存
rm -rf user_data/data/binance/*.json

# 重新下载数据
freqtrade download-data --exchange binance --pairs BTC/USDT --days 30
```

### 9.2 日志分析

```bash
# 查看实时日志
tail -f user_data/logs/freqtrade.log

# 搜索错误信息
grep -i "error\|exception" user_data/logs/freqtrade.log

# 分析交易记录
grep -i "entering\|exiting\|order" user_data/logs/freqtrade.log
```

### 9.3 性能优化

```json
{
  "internals": {
    "process_throttle_secs": 5,
    "heartbeat_interval": 60,
    "sd_notify": false
  },
  
  "dataformat_ohlcv": "feather",
  "dataformat_trades": "feather",
  
  "cache": {
    "enabled": true,
    "filename": "user_data/cache/freqtrade.cache",
    "max_size": 1000000
  }
}
```

## 10. 最佳实践

### 10.1 风险管理

1. **资金管理**:
   - 单笔交易风险不超过总资金的1-2%
   - 同时开仓不超过总资金的10%
   - 设置合理的止损和止盈

2. **策略验证**:
   - 充分的历史数据回测
   - 多市场环境测试
   - 前向测试验证

3. **监控告警**:
   - 设置关键指标告警
   - 定期检查系统状态
   - 建立应急预案

### 10.2 开发建议

1. **代码质量**:
   - 遵循PEP8规范
   - 添加详细注释
   - 单元测试覆盖

2. **策略设计**:
   - 简单清晰的逻辑
   - 避免过度拟合
   - 考虑交易成本

3. **性能优化**:
   - 合理使用缓存
   - 优化数据结构
   - 避免重复计算

### 10.3 部署建议

1. **环境配置**:
   - 使用虚拟环境
   - 版本控制配置
   - 定期备份数据

2. **监控运维**:
   - 系统资源监控
   - 日志轮转设置
   - 自动重启机制

3. **安全措施**:
   - API密钥加密存储
   - 网络访问控制
   - 定期安全审计

这份完整的使用指南涵盖了 FreqTrade 的所有主要功能和最佳实践，帮助您从入门到精通，成功构建和运行自己的量化交易系统。
